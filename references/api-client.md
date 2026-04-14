# API Client — Dio

## Contents

- Dio setup
- Interceptors
- Base API service
- Error mapping
- Retry logic
- Connectivity handling

## Dio Setup [LOW]

### Dio Client

```dart
// core/network/dio_client.dart
import 'package:dio/dio.dart';
import 'package:injectable/injectable.dart';

@lazySingleton
class DioClient {
  late final Dio dio;

  DioClient(
    AuthInterceptor authInterceptor,
    LoggingInterceptor loggingInterceptor,
    ErrorInterceptor errorInterceptor,
  ) {
    dio = Dio(
      BaseOptions(
        baseUrl: EnvConfig.instance.apiBaseUrl,
        connectTimeout: const Duration(seconds: 10),
        receiveTimeout: const Duration(seconds: 30),
        sendTimeout: const Duration(seconds: 15),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      ),
    );

    dio.interceptors.addAll([
      authInterceptor,
      loggingInterceptor,
      errorInterceptor,
    ]);
  }
}
```

### API Endpoints

```dart
// core/network/api_endpoints.dart
class ApiEndpoints {
  ApiEndpoints._();

  static const String _base = '/api/v1';

  // Auth
  static const String login = '$_base/auth/login';
  static const String register = '$_base/auth/register';
  static const String refreshToken = '$_base/auth/refresh';
  static const String logout = '$_base/auth/logout';
  static const String me = '$_base/auth/me';

  // Users
  static const String users = '$_base/users';
  static String user(String id) => '$_base/users/$id';

  // Health
  static const String health = '$_base/health';
}
```

## Interceptors [LOW]

### Auth Interceptor (Token Injection + Refresh)

```dart
// core/network/interceptors/auth_interceptor.dart
@injectable
class AuthInterceptor extends QueuedInterceptor {
  final AuthLocalDataSource _authLocalDataSource;
  final Dio _refreshDio; // Separate Dio instance for refresh (no interceptors)

  AuthInterceptor(this._authLocalDataSource)
    : _refreshDio = Dio(BaseOptions(
        baseUrl: EnvConfig.instance.apiBaseUrl,
        headers: {'Content-Type': 'application/json'},
      ));

  @override
  Future<void> onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    // Skip auth header for public endpoints
    if (_isPublicEndpoint(options.path)) {
      return handler.next(options);
    }

    final token = await _authLocalDataSource.getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  Future<void> onError(
    DioException err,
    ErrorInterceptorHandler handler,
  ) async {
    if (err.response?.statusCode != 401) {
      return handler.next(err);
    }

    // Attempt token refresh
    try {
      final refreshToken = await _authLocalDataSource.getRefreshToken();
      if (refreshToken == null) {
        return handler.next(err);
      }

      final response = await _refreshDio.post(
        ApiEndpoints.refreshToken,
        data: {'refresh_token': refreshToken},
      );

      final newAccessToken = response.data['data']['access_token'] as String;
      final newRefreshToken = response.data['data']['refresh_token'] as String;

      await _authLocalDataSource.saveTokens(
        accessToken: newAccessToken,
        refreshToken: newRefreshToken,
      );

      // Retry original request with new token
      final options = err.requestOptions;
      options.headers['Authorization'] = 'Bearer $newAccessToken';
      final retryResponse = await _refreshDio.fetch(options);
      return handler.resolve(retryResponse);
    } on DioException {
      // Refresh failed → clear tokens, let error propagate
      await _authLocalDataSource.clearTokens();
      return handler.next(err);
    }
  }

  bool _isPublicEndpoint(String path) {
    const publicPaths = [
      ApiEndpoints.login,
      ApiEndpoints.register,
      ApiEndpoints.refreshToken,
      ApiEndpoints.health,
    ];
    return publicPaths.any((p) => path.contains(p));
  }
}
```

### Logging Interceptor

```dart
// core/network/interceptors/logging_interceptor.dart
@injectable
class LoggingInterceptor extends Interceptor {
  final Logger _logger = Logger();

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (EnvConfig.instance.enableLogging) {
      _logger.i('→ ${options.method} ${options.path}');
      // DON'T log request body — may contain passwords
    }
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    if (EnvConfig.instance.enableLogging) {
      _logger.i('← ${response.statusCode} ${response.requestOptions.path}');
    }
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    if (EnvConfig.instance.enableLogging) {
      _logger.e('✕ ${err.response?.statusCode} ${err.requestOptions.path}');
    }
    handler.next(err);
  }
}
```

### Error Interceptor (DioException → App Exception)

```dart
// core/network/interceptors/error_interceptor.dart
@injectable
class ErrorInterceptor extends Interceptor {
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    final exception = switch (err.type) {
      DioExceptionType.connectionTimeout ||
      DioExceptionType.sendTimeout ||
      DioExceptionType.receiveTimeout =>
        const NetworkException('Connection timed out'),

      DioExceptionType.connectionError =>
        const NetworkException('No internet connection'),

      DioExceptionType.badResponse => _handleBadResponse(err.response!),

      DioExceptionType.cancel =>
        const NetworkException('Request cancelled'),

      _ => ServerException(
          err.message ?? 'Unknown error',
          statusCode: err.response?.statusCode,
        ),
    };

    handler.reject(
      DioException(
        requestOptions: err.requestOptions,
        error: exception,
        response: err.response,
        type: err.type,
      ),
    );
  }

  Exception _handleBadResponse(Response response) {
    final statusCode = response.statusCode;
    final body = response.data as Map<String, dynamic>?;
    final message = body?['message'] as String? ?? 'Server error';

    return switch (statusCode) {
      401 => UnauthorizedException(message),
      403 => ServerException(message, statusCode: 403),
      404 => ServerException(message, statusCode: 404),
      422 => ServerException(message, statusCode: 422, body: body),
      429 => ServerException('Too many requests', statusCode: 429),
      _ => ServerException(message, statusCode: statusCode),
    };
  }
}
```

## Base API Service [MEDIUM]

```dart
// core/network/base_api_service.dart
abstract class BaseApiService {
  final DioClient dioClient;

  const BaseApiService(this.dioClient);

  Dio get dio => dioClient.dio;

  /// Wraps API calls with standard error handling
  Future<T> safeApiCall<T>(Future<T> Function() apiCall) async {
    try {
      return await apiCall();
    } on DioException catch (e) {
      throw e.error ?? ServerException(e.message ?? 'API call failed');
    }
  }
}
```

### Usage in Data Source

```dart
class AuthRemoteDataSourceImpl extends BaseApiService
    implements AuthRemoteDataSource {

  AuthRemoteDataSourceImpl(super.dioClient);

  @override
  Future<UserModel> login(String email, String password) async {
    return safeApiCall(() async {
      final response = await dio.post(
        ApiEndpoints.login,
        data: {'email': email, 'password': password},
      );
      return UserModel.fromJson(response.data['data']);
    });
  }
}
```

## Connectivity Handling [MEDIUM]

### Network Info

```dart
// core/network/network_info.dart
abstract class NetworkInfo {
  Future<bool> get isConnected;
  Stream<bool> get onConnectivityChanged;
}

@LazySingleton(as: NetworkInfo)
class NetworkInfoImpl implements NetworkInfo {
  final Connectivity _connectivity;

  NetworkInfoImpl(this._connectivity);

  @override
  Future<bool> get isConnected async {
    final result = await _connectivity.checkConnectivity();
    return !result.contains(ConnectivityResult.none);
  }

  @override
  Stream<bool> get onConnectivityChanged {
    return _connectivity.onConnectivityChanged.map(
      (results) => !results.contains(ConnectivityResult.none),
    );
  }
}
```

### Usage in Repository

```dart
@override
Future<Either<Failure, List<Item>>> getItems() async {
  if (!await networkInfo.isConnected) {
    // Try local cache
    try {
      final cached = await localDataSource.getCachedItems();
      return Right(cached.map((m) => m.toEntity()).toList());
    } on CacheException {
      return const Left(NetworkFailure());
    }
  }

  try {
    final models = await remoteDataSource.getItems();
    await localDataSource.cacheItems(models); // Cache for offline
    return Right(models.map((m) => m.toEntity()).toList());
  } on ServerException catch (e) {
    return Left(ServerFailure(e.message, statusCode: e.statusCode));
  }
}
```

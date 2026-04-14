# Authentication — Flutter (Client-Side)

## Contents

- Auth flow overview
- Token storage
- Auth data sources
- Auth repository
- Auth BLoC / state
- Token refresh strategy
- Biometric auth (optional)
- OAuth2 / OIDC (optional)

## Auth Flow Overview [LOW]

Flutter is a **client** — it does NOT generate or verify JWT tokens. It:
1. Sends credentials to the backend API
2. Receives access + refresh tokens
3. Stores tokens securely (flutter_secure_storage)
4. Attaches access token to every API request (Dio interceptor)
5. Refreshes access token when it expires (401 → refresh → retry)
6. Clears tokens on logout or refresh failure

```
┌──────────┐     credentials      ┌──────────┐
│  Login   │ ──────────────────── │  Backend │
│  Screen  │                      │   API    │
│          │ ◄─── tokens ──────── │          │
└──────────┘                      └──────────┘
     │
     ▼
┌──────────────┐
│ Secure       │ save access_token + refresh_token
│ Storage      │
└──────────────┘
     │
     ▼
┌──────────────┐
│ Dio          │ attach Authorization: Bearer {access_token}
│ Interceptor  │ on 401 → refresh → retry
└──────────────┘
```

## Token Storage [LOW]

Use `flutter_secure_storage`. See [local-storage.md](local-storage.md) for the `SecureStorage` wrapper.

**Rules:**
- NEVER store tokens in SharedPreferences (unencrypted)
- NEVER store tokens in memory only (lost on app kill)
- NEVER log token values
- DO use `encryptedSharedPreferences: true` on Android
- DO set iOS accessibility to `first_unlock_this_device`

## Auth Data Sources [LOW]

### Remote Data Source

```dart
// features/auth/data/datasources/auth_remote_data_source.dart
abstract class AuthRemoteDataSource {
  Future<AuthResponseModel> login(String email, String password);
  Future<AuthResponseModel> register(RegisterParams params);
  Future<TokenModel> refreshToken(String refreshToken);
  Future<void> logout(String accessToken);
  Future<UserModel> getCurrentUser();
}

@LazySingleton(as: AuthRemoteDataSource)
class AuthRemoteDataSourceImpl extends BaseApiService
    implements AuthRemoteDataSource {

  AuthRemoteDataSourceImpl(super.dioClient);

  @override
  Future<AuthResponseModel> login(String email, String password) {
    return safeApiCall(() async {
      final response = await dio.post(
        ApiEndpoints.login,
        data: {'email': email, 'password': password},
      );
      return AuthResponseModel.fromJson(response.data['data']);
    });
  }

  @override
  Future<AuthResponseModel> register(RegisterParams params) {
    return safeApiCall(() async {
      final response = await dio.post(
        ApiEndpoints.register,
        data: params.toJson(),
      );
      return AuthResponseModel.fromJson(response.data['data']);
    });
  }

  @override
  Future<TokenModel> refreshToken(String refreshToken) {
    return safeApiCall(() async {
      final response = await dio.post(
        ApiEndpoints.refreshToken,
        data: {'refresh_token': refreshToken},
      );
      return TokenModel.fromJson(response.data['data']);
    });
  }

  @override
  Future<void> logout(String accessToken) {
    return safeApiCall(() async {
      await dio.post(ApiEndpoints.logout);
    });
  }

  @override
  Future<UserModel> getCurrentUser() {
    return safeApiCall(() async {
      final response = await dio.get(ApiEndpoints.me);
      return UserModel.fromJson(response.data['data']);
    });
  }
}
```

### Local Data Source

```dart
// features/auth/data/datasources/auth_local_data_source.dart
abstract class AuthLocalDataSource {
  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  });
  Future<String?> getAccessToken();
  Future<String?> getRefreshToken();
  Future<void> clearTokens();
  Future<void> saveUser(UserModel user);
  Future<UserModel?> getCachedUser();
}

@LazySingleton(as: AuthLocalDataSource)
class AuthLocalDataSourceImpl implements AuthLocalDataSource {
  final SecureStorage _secureStorage;
  final LocalStorage _localStorage;

  AuthLocalDataSourceImpl(this._secureStorage, this._localStorage);

  @override
  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) => _secureStorage.saveTokens(
    accessToken: accessToken,
    refreshToken: refreshToken,
  );

  @override
  Future<String?> getAccessToken() => _secureStorage.getAccessToken();

  @override
  Future<String?> getRefreshToken() => _secureStorage.getRefreshToken();

  @override
  Future<void> clearTokens() => _secureStorage.clearTokens();

  // Cache user as JSON in local storage for quick startup
  @override
  Future<void> saveUser(UserModel user) async {
    // Implementation depends on chosen local storage
  }

  @override
  Future<UserModel?> getCachedUser() async {
    // Implementation depends on chosen local storage
    return null;
  }
}
```

## Auth Repository [LOW]

```dart
// features/auth/data/repositories/auth_repository_impl.dart
@LazySingleton(as: AuthRepository)
class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource _remoteDataSource;
  final AuthLocalDataSource _localDataSource;

  AuthRepositoryImpl(this._remoteDataSource, this._localDataSource);

  @override
  Future<Either<Failure, User>> login(String email, String password) async {
    try {
      final response = await _remoteDataSource.login(email, password);
      await _localDataSource.saveTokens(
        accessToken: response.accessToken,
        refreshToken: response.refreshToken,
      );
      await _localDataSource.saveUser(response.user);
      return Right(response.user.toEntity());
    } on UnauthorizedException catch (e) {
      return Left(AuthFailure(e.message));
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message, statusCode: e.statusCode));
    } on NetworkException {
      return Left(const NetworkFailure());
    }
  }

  @override
  Future<Either<Failure, void>> logout() async {
    try {
      final token = await _localDataSource.getAccessToken();
      if (token != null) {
        await _remoteDataSource.logout(token);
      }
    } catch (_) {
      // Ignore server errors on logout — clear local state regardless
    }
    await _localDataSource.clearTokens();
    return const Right(null);
  }

  @override
  Future<Either<Failure, User>> getCurrentUser() async {
    try {
      final model = await _remoteDataSource.getCurrentUser();
      await _localDataSource.saveUser(model);
      return Right(model.toEntity());
    } on UnauthorizedException {
      return Left(const SessionExpiredFailure());
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message, statusCode: e.statusCode));
    } on NetworkException {
      // Try cached user when offline
      final cached = await _localDataSource.getCachedUser();
      if (cached != null) {
        return Right(cached.toEntity());
      }
      return const Left(NetworkFailure());
    }
  }

  @override
  Future<Either<Failure, TokenPair>> refreshToken() async {
    try {
      final refreshToken = await _localDataSource.getRefreshToken();
      if (refreshToken == null) {
        return Left(const SessionExpiredFailure());
      }
      final tokens = await _remoteDataSource.refreshToken(refreshToken);
      await _localDataSource.saveTokens(
        accessToken: tokens.accessToken,
        refreshToken: tokens.refreshToken,
      );
      return Right(tokens.toEntity());
    } on UnauthorizedException {
      await _localDataSource.clearTokens();
      return Left(const SessionExpiredFailure());
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message));
    }
  }
}
```

## Auth BLoC [LOW]

```dart
// features/auth/presentation/bloc/auth_bloc.dart

// Events
sealed class AuthEvent extends Equatable {
  const AuthEvent();
  @override
  List<Object?> get props => [];
}

class AuthCheckRequested extends AuthEvent {
  const AuthCheckRequested();
}

class AuthLoginRequested extends AuthEvent {
  final String email;
  final String password;
  const AuthLoginRequested(this.email, this.password);
  @override
  List<Object?> get props => [email, password];
}

class AuthLogoutRequested extends AuthEvent {
  const AuthLogoutRequested();
}

// States
sealed class AuthState extends Equatable {
  const AuthState();
  @override
  List<Object?> get props => [];
}

class AuthInitial extends AuthState {
  const AuthInitial();
}

class AuthLoading extends AuthState {
  const AuthLoading();
}

class AuthAuthenticated extends AuthState {
  final User user;
  const AuthAuthenticated(this.user);
  @override
  List<Object?> get props => [user];
}

class AuthUnauthenticated extends AuthState {
  const AuthUnauthenticated();
}

class AuthError extends AuthState {
  final String message;
  const AuthError(this.message);
  @override
  List<Object?> get props => [message];
}

// BLoC
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final Login _login;
  final Logout _logout;
  final GetCurrentUser _getCurrentUser;

  AuthBloc(this._login, this._logout, this._getCurrentUser)
    : super(const AuthInitial()) {
    on<AuthCheckRequested>(_onCheckRequested);
    on<AuthLoginRequested>(_onLoginRequested);
    on<AuthLogoutRequested>(_onLogoutRequested);
  }

  Future<void> _onCheckRequested(
    AuthCheckRequested event,
    Emitter<AuthState> emit,
  ) async {
    final result = await _getCurrentUser();
    result.fold(
      (_) => emit(const AuthUnauthenticated()),
      (user) => emit(AuthAuthenticated(user)),
    );
  }

  Future<void> _onLoginRequested(
    AuthLoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(const AuthLoading());
    final result = await _login(event.email, event.password);
    result.fold(
      (failure) => emit(AuthError(failure.message)),
      (user) => emit(AuthAuthenticated(user)),
    );
  }

  Future<void> _onLogoutRequested(
    AuthLogoutRequested event,
    Emitter<AuthState> emit,
  ) async {
    await _logout();
    emit(const AuthUnauthenticated());
  }
}
```

## Token Refresh Strategy [LOW]

Token refresh is handled by the **Dio Auth Interceptor** (see [api-client.md](api-client.md)), NOT by the BLoC.

Flow:
1. API request returns 401
2. Auth interceptor catches it (using `QueuedInterceptor` to serialize concurrent refreshes)
3. Reads refresh token from secure storage
4. Calls refresh endpoint with a **separate Dio instance** (no interceptors → no infinite loop)
5. If refresh succeeds → save new tokens → retry original request
6. If refresh fails → clear tokens → let 401 propagate → BLoC handles logout

**Critical:** The refresh Dio instance MUST NOT have the auth interceptor attached. Otherwise, a 401 on the refresh endpoint triggers another refresh → infinite loop.

## Biometric Auth (Optional) [MEDIUM]

```dart
// Using local_auth package
class BiometricService {
  final LocalAuthentication _auth = LocalAuthentication();

  Future<bool> isAvailable() async {
    final canCheck = await _auth.canCheckBiometrics;
    final isSupported = await _auth.isDeviceSupported();
    return canCheck && isSupported;
  }

  Future<bool> authenticate() async {
    return _auth.authenticate(
      localizedReason: 'Authenticate to access the app',
      options: const AuthenticationOptions(
        stickyAuth: true,
        biometricOnly: true,
      ),
    );
  }
}
```

## OAuth2 / OIDC (Optional) [MEDIUM]

```dart
// Using flutter_appauth
class OAuthService {
  final FlutterAppAuth _appAuth = const FlutterAppAuth();

  static const _clientId = 'your-client-id';
  static const _redirectUrl = 'com.example.app://callback';
  static const _issuer = 'https://auth.example.com';

  Future<AuthorizationTokenResponse?> login() async {
    return _appAuth.authorizeAndExchangeCode(
      AuthorizationTokenRequest(
        _clientId,
        _redirectUrl,
        issuer: _issuer,
        scopes: ['openid', 'profile', 'email'],
      ),
    );
  }

  Future<TokenResponse?> refreshToken(String refreshToken) async {
    return _appAuth.token(
      TokenRequest(
        _clientId,
        _redirectUrl,
        issuer: _issuer,
        refreshToken: refreshToken,
      ),
    );
  }
}
```

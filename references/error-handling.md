# Error Handling — Flutter

## Contents

- Exception vs Failure
- Failure class hierarchy
- Either pattern
- Global error handler
- UI error states
- Error code system

## Exception vs Failure [LOW]

Two distinct concepts — do NOT mix them:

| Concept | Layer | Purpose |
|---------|-------|---------|
| **Exception** | Data layer | Thrown by data sources when operations fail (network, parsing, storage) |
| **Failure** | Domain layer | Returned via `Either<Failure, T>` — represents a domain-level failure |

**Rule:** Exceptions are CAUGHT in the repository implementation and MAPPED to Failures. Failures are RETURNED (not thrown) to the presentation layer.

```
Data Source → throws Exception
    ↓
Repository Impl → catches Exception → returns Left(Failure)
    ↓
Use Case → passes Either through
    ↓
BLoC/Provider → fold() the Either → emit success or error state
    ↓
Widget → displays based on state
```

## Failure Class Hierarchy [LOW]

```dart
// core/error/failures.dart
import 'package:equatable/equatable.dart';

sealed class Failure extends Equatable {
  final String message;
  final int? code;

  const Failure(this.message, {this.code});

  @override
  List<Object?> get props => [message, code];
}

// ─── Network Failures ─────────────────────────────────
class ServerFailure extends Failure {
  final int? statusCode;
  const ServerFailure(super.message, {this.statusCode, super.code});

  @override
  List<Object?> get props => [message, statusCode, code];
}

class NetworkFailure extends Failure {
  const NetworkFailure([super.message = 'No internet connection']);
}

class TimeoutFailure extends Failure {
  const TimeoutFailure([super.message = 'Request timed out']);
}

// ─── Auth Failures ────────────────────────────────────
class AuthFailure extends Failure {
  const AuthFailure(super.message, {super.code});
}

class UnauthorizedFailure extends Failure {
  const UnauthorizedFailure([super.message = 'Unauthorized']);
}

class ForbiddenFailure extends Failure {
  const ForbiddenFailure([super.message = 'Access denied']);
}

class SessionExpiredFailure extends Failure {
  const SessionExpiredFailure([super.message = 'Session expired']);
}

// ─── Validation Failures ──────────────────────────────
class ValidationFailure extends Failure {
  final Map<String, List<String>> fieldErrors;
  const ValidationFailure(
    super.message, {
    this.fieldErrors = const {},
    super.code,
  });

  @override
  List<Object?> get props => [message, fieldErrors, code];
}

// ─── Storage Failures ─────────────────────────────────
class CacheFailure extends Failure {
  const CacheFailure([super.message = 'Cache operation failed']);
}

class StorageFailure extends Failure {
  const StorageFailure([super.message = 'Local storage error']);
}

// ─── General ──────────────────────────────────────────
class UnexpectedFailure extends Failure {
  const UnexpectedFailure([super.message = 'An unexpected error occurred']);
}
```

## Exception Classes (Data Layer) [LOW]

```dart
// core/error/exceptions.dart

class ServerException implements Exception {
  final String message;
  final int? statusCode;
  final Map<String, dynamic>? body;

  const ServerException(this.message, {this.statusCode, this.body});
}

class NetworkException implements Exception {
  final String message;
  const NetworkException([this.message = 'No internet connection']);
}

class CacheException implements Exception {
  final String message;
  const CacheException([this.message = 'Cache operation failed']);
}

class UnauthorizedException implements Exception {
  final String message;
  const UnauthorizedException([this.message = 'Unauthorized']);
}

class ParseException implements Exception {
  final String message;
  const ParseException([this.message = 'Failed to parse response']);
}
```

## Either Pattern [LOW]

Use `dartz` or `fpdart` for `Either<Failure, T>`.

### Repository returns Either

```dart
@override
Future<Either<Failure, User>> login(String email, String password) async {
  try {
    final model = await remoteDataSource.login(email, password);
    return Right(model.toEntity());
  } on ServerException catch (e) {
    return Left(ServerFailure(e.message, statusCode: e.statusCode));
  } on NetworkException {
    return Left(const NetworkFailure());
  } on UnauthorizedException catch (e) {
    return Left(AuthFailure(e.message));
  } catch (e) {
    return Left(UnexpectedFailure(e.toString()));
  }
}
```

### BLoC folds Either

```dart
// In BLoC event handler
Future<void> _onLoginRequested(
  LoginRequested event,
  Emitter<AuthState> emit,
) async {
  emit(const AuthLoading());

  final result = await loginUseCase(event.email, event.password);

  result.fold(
    (failure) => emit(AuthError(failure.message)),
    (user) => emit(AuthAuthenticated(user)),
  );
}
```

## Global Error Handler [LOW]

### Bootstrap Setup

```dart
// bootstrap.dart
void bootstrap() {
  // Catch Flutter framework errors
  FlutterError.onError = (details) {
    FlutterError.presentError(details);
    // Log to crashlytics in prod
    if (!EnvConfig.instance.enableLogging) {
      FirebaseCrashlytics.instance.recordFlutterFatalError(details);
    }
  };

  // Catch async errors not handled by Flutter
  PlatformDispatcher.instance.onError = (error, stack) {
    // Log to crashlytics in prod
    if (!EnvConfig.instance.enableLogging) {
      FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    }
    return true;
  };

  runApp(const App());
}
```

### Zone-Based Error Catching

```dart
void main() {
  runZonedGuarded(
    () {
      WidgetsFlutterBinding.ensureInitialized();
      EnvConfig.initialize(Environment.dev);
      configureDependencies();
      bootstrap();
    },
    (error, stackTrace) {
      // Handle uncaught errors
      debugPrint('Uncaught error: $error');
      debugPrint('Stack trace: $stackTrace');
    },
  );
}
```

## UI Error States [MEDIUM]

### Error View Widget

```dart
// core/widgets/error_view.dart
class ErrorView extends StatelessWidget {
  final String message;
  final VoidCallback? onRetry;

  const ErrorView({
    required this.message,
    this.onRetry,
  });

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(AppSpacing.lg),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(
              Icons.error_outline,
              size: 64,
              color: Theme.of(context).colorScheme.error,
            ),
            const SizedBox(height: AppSpacing.md),
            Text(
              message,
              style: Theme.of(context).textTheme.bodyLarge,
              textAlign: TextAlign.center,
            ),
            if (onRetry != null) ...[
              const SizedBox(height: AppSpacing.lg),
              ElevatedButton.icon(
                onPressed: onRetry,
                icon: const Icon(Icons.refresh),
                label: const Text('Retry'),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

### Loading / Empty Views

```dart
// core/widgets/loading_view.dart
class LoadingView extends StatelessWidget {
  final String? message;
  const LoadingView({this.message});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          const CircularProgressIndicator(),
          if (message != null) ...[
            const SizedBox(height: AppSpacing.md),
            Text(message!, style: Theme.of(context).textTheme.bodyMedium),
          ],
        ],
      ),
    );
  }
}

// core/widgets/empty_view.dart
class EmptyView extends StatelessWidget {
  final String message;
  final IconData icon;
  final VoidCallback? onAction;
  final String? actionLabel;

  const EmptyView({
    required this.message,
    this.icon = Icons.inbox_outlined,
    this.onAction,
    this.actionLabel,
  });

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Icon(icon, size: 64, color: Theme.of(context).disabledColor),
          const SizedBox(height: AppSpacing.md),
          Text(message, style: Theme.of(context).textTheme.bodyLarge),
          if (onAction != null && actionLabel != null) ...[
            const SizedBox(height: AppSpacing.lg),
            ElevatedButton(onPressed: onAction, child: Text(actionLabel!)),
          ],
        ],
      ),
    );
  }
}
```

### Usage in Screens

```dart
BlocBuilder<ItemBloc, ItemState>(
  builder: (context, state) => switch (state) {
    ItemInitial() => const SizedBox.shrink(),
    ItemLoading() => const LoadingView(),
    ItemLoaded(:final items) when items.isEmpty =>
      const EmptyView(message: 'No items yet'),
    ItemLoaded(:final items) => ItemList(items: items),
    ItemError(:final message) => ErrorView(
      message: message,
      onRetry: () => context.read<ItemBloc>().add(const ItemFetchRequested()),
    ),
  },
)
```

## Error Code System [MEDIUM]

Map API error codes to user-friendly messages:

```dart
// core/error/error_codes.dart
class ErrorCodes {
  ErrorCodes._();

  // Auth errors (30000 range — matches backend)
  static const invalidCredentials = 30001;
  static const tokenExpired = 30002;
  static const accountLocked = 30003;
  static const emailAlreadyExists = 30004;

  // Validation errors (20000 range)
  static const validationFailed = 20001;

  // Resource errors (40000 range)
  static const notFound = 40001;
  static const conflict = 40002;

  static String messageFor(int code) => switch (code) {
    30001 => 'Invalid email or password',
    30002 => 'Your session has expired. Please log in again.',
    30003 => 'Your account has been locked. Contact support.',
    30004 => 'An account with this email already exists.',
    40001 => 'The requested resource was not found.',
    _ => 'An unexpected error occurred.',
  };
}
```

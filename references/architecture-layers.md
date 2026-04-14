# Architecture Layers — Flutter Clean Architecture

## Contents

- Layer overview
- Project structure
- Dependency rule
- Each layer in detail
- Dependency injection
- Environment configuration
- Theme & design system
- i18n setup
- Version-specific best practice research

## Layer Overview

```
┌─────────────────────────────────────────┐
│           Presentation Layer            │
│   Screens, Widgets, BLoC/Providers      │
├─────────────────────────────────────────┤
│             Domain Layer                │
│   Entities, Use Cases, Repo Interfaces  │
├─────────────────────────────────────────┤
│              Data Layer                 │
│   Repo Impls, Data Sources, DTOs        │
│   (Remote API + Local Storage)          │
└─────────────────────────────────────────┘
```

**Dependency Rule:** Dependencies point INWARD only. Presentation → Domain ← Data. Domain knows nothing about Data or Presentation.

## Project Structure

```
lib/
├── main.dart                          # Entry point
├── app.dart                           # MaterialApp / GoRouter setup
├── bootstrap.dart                     # DI initialization, error handling
│
├── core/                              # Shared across all features
│   ├── config/
│   │   ├── env_config.dart            # Environment-aware config
│   │   └── app_constants.dart         # Magic values → named constants
│   ├── di/
│   │   ├── injection.dart             # GetIt setup
│   │   └── injection.config.dart      # Generated
│   ├── error/
│   │   ├── failures.dart              # Failure class hierarchy
│   │   ├── exceptions.dart            # Exception classes (data layer)
│   │   └── error_handler.dart         # Global error handler
│   ├── network/
│   │   ├── dio_client.dart            # Dio instance + interceptors
│   │   ├── api_endpoints.dart         # All endpoint paths
│   │   └── interceptors/
│   │       ├── auth_interceptor.dart
│   │       ├── logging_interceptor.dart
│   │       └── retry_interceptor.dart
│   ├── router/
│   │   ├── app_router.dart            # GoRouter config
│   │   ├── route_names.dart           # Named route constants
│   │   └── guards/
│   │       └── auth_guard.dart
│   ├── storage/
│   │   ├── secure_storage.dart        # Token storage
│   │   ├── local_storage.dart         # SharedPreferences wrapper
│   │   └── database/                  # Drift or Hive setup
│   ├── theme/
│   │   ├── app_theme.dart             # Light + Dark ThemeData
│   │   ├── app_colors.dart            # Color scheme
│   │   ├── app_typography.dart        # Text styles
│   │   └── app_spacing.dart           # Padding/margin constants
│   ├── l10n/                          # Localization
│   │   ├── app_en.arb
│   │   └── app_vi.arb
│   ├── utils/
│   │   ├── extensions/                # Dart extensions
│   │   └── helpers/                   # Utility functions
│   └── widgets/                       # Shared reusable widgets
│       ├── error_view.dart
│       ├── empty_view.dart
│       ├── loading_view.dart
│       └── app_button.dart
│
├── features/                          # Feature modules
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── auth_remote_data_source.dart
│   │   │   │   └── auth_local_data_source.dart
│   │   │   ├── models/
│   │   │   │   ├── user_model.dart        # DTO with fromJson/toJson
│   │   │   │   └── token_model.dart
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart              # Pure entity (no serialization)
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart   # Abstract class
│   │   │   └── usecases/
│   │   │       ├── login.dart
│   │   │       ├── register.dart
│   │   │       ├── logout.dart
│   │   │       ├── get_current_user.dart
│   │   │       └── refresh_token.dart
│   │   └── presentation/
│   │       ├── bloc/                      # or providers/
│   │       │   ├── auth_bloc.dart
│   │       │   ├── auth_event.dart
│   │       │   └── auth_state.dart
│   │       ├── screens/
│   │       │   ├── login_screen.dart
│   │       │   └── register_screen.dart
│   │       └── widgets/
│   │           └── auth_form.dart
│   │
│   ├── home/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │
│   └── settings/
│       ├── data/
│       ├── domain/
│       └── presentation/
│
test/
├── core/
│   ├── network/
│   ├── error/
│   └── utils/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── home/
├── helpers/                           # Test utilities
│   ├── test_helpers.dart
│   ├── mock_generators.dart
│   └── pump_app.dart                  # Widget test helper
└── fixtures/                          # JSON fixtures
    ├── user.json
    └── token.json

integration_test/
├── app_test.dart
└── auth_flow_test.dart
```

## Each Layer in Detail

### Domain Layer (innermost — no dependencies on Flutter or packages)

**Entities:** Pure Dart classes. No annotations, no serialization.

```dart
// domain/entities/user.dart
class User {
  final String id;
  final String email;
  final String firstName;
  final String lastName;
  final UserRole role;

  const User({
    required this.id,
    required this.email,
    required this.firstName,
    required this.lastName,
    required this.role,
  });
}
```

**Repository interfaces:** Abstract classes defining contracts.

```dart
// domain/repositories/auth_repository.dart
abstract class AuthRepository {
  Future<Either<Failure, User>> login(String email, String password);
  Future<Either<Failure, User>> register(RegisterParams params);
  Future<Either<Failure, void>> logout();
  Future<Either<Failure, User>> getCurrentUser();
  Future<Either<Failure, TokenPair>> refreshToken();
}
```

**Use Cases:** Single-responsibility classes. One public method: `call()`.

```dart
// domain/usecases/login.dart
class Login {
  final AuthRepository repository;

  const Login(this.repository);

  Future<Either<Failure, User>> call(String email, String password) {
    return repository.login(email, password);
  }
}
```

### Data Layer (implements Domain interfaces)

**Models (DTOs):** Extend or map to entities. Handle serialization.

```dart
// data/models/user_model.dart
@JsonSerializable()
class UserModel {
  final String id;
  final String email;
  @JsonKey(name: 'first_name')
  final String firstName;
  @JsonKey(name: 'last_name')
  final String lastName;
  final String role;

  const UserModel({...});

  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);

  Map<String, dynamic> toJson() => _$UserModelToJson(this);

  User toEntity() => User(
    id: id,
    email: email,
    firstName: firstName,
    lastName: lastName,
    role: UserRole.fromString(role),
  );
}
```

**Data Sources:** Direct interaction with APIs or local storage.

```dart
// data/datasources/auth_remote_data_source.dart
abstract class AuthRemoteDataSource {
  Future<UserModel> login(String email, String password);
  Future<UserModel> register(RegisterParams params);
  Future<void> logout();
  Future<UserModel> getCurrentUser();
  Future<TokenModel> refreshToken(String refreshToken);
}
```

**Repository Implementation:** Combines data sources, maps exceptions to failures.

```dart
// data/repositories/auth_repository_impl.dart
class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource remoteDataSource;
  final AuthLocalDataSource localDataSource;

  @override
  Future<Either<Failure, User>> login(String email, String password) async {
    try {
      final userModel = await remoteDataSource.login(email, password);
      await localDataSource.saveTokens(userModel.tokens);
      return Right(userModel.toEntity());
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message, e.statusCode));
    } on NetworkException {
      return Left(const NetworkFailure());
    }
  }
}
```

### Presentation Layer (UI + state management)

BLoC or Riverpod — depends on Phase 0 decision. See state management package docs for patterns.

## Dependency Injection

### GetIt + Injectable Setup

```dart
// core/di/injection.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'injection.config.dart';

final getIt = GetIt.instance;

@InjectableInit()
void configureDependencies() => getIt.init();
```

Register via annotations:

```dart
@lazySingleton
class DioClient { ... }

@LazySingleton(as: AuthRepository)
class AuthRepositoryImpl implements AuthRepository { ... }

@injectable
class Login {
  final AuthRepository repository;
  const Login(this.repository);
}
```

Run code generation:

```bash
dart run build_runner build --delete-conflicting-outputs
```

## Environment Configuration

### Flavor-Aware Config [LOW]

```dart
// core/config/env_config.dart
enum Environment { dev, staging, prod }

class EnvConfig {
  final Environment environment;
  final String apiBaseUrl;
  final bool enableLogging;

  const EnvConfig._({
    required this.environment,
    required this.apiBaseUrl,
    required this.enableLogging,
  });

  static late final EnvConfig instance;

  static void initialize(Environment env) {
    instance = switch (env) {
      Environment.dev => const EnvConfig._(
        environment: Environment.dev,
        apiBaseUrl: 'https://dev-api.example.com',
        enableLogging: true,
      ),
      Environment.staging => const EnvConfig._(
        environment: Environment.staging,
        apiBaseUrl: 'https://staging-api.example.com',
        enableLogging: true,
      ),
      Environment.prod => const EnvConfig._(
        environment: Environment.prod,
        apiBaseUrl: 'https://api.example.com',
        enableLogging: false,
      ),
    };
  }
}
```

### Entry Points Per Flavor

```dart
// main_dev.dart
void main() {
  EnvConfig.initialize(Environment.dev);
  bootstrap();
}

// main_staging.dart
void main() {
  EnvConfig.initialize(Environment.staging);
  bootstrap();
}

// main_prod.dart
void main() {
  EnvConfig.initialize(Environment.prod);
  bootstrap();
}
```

## Theme & Design System [MEDIUM]

### Material 3 ThemeData

```dart
// core/theme/app_theme.dart
class AppTheme {
  static ThemeData light() => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.primary,
      brightness: Brightness.light,
    ),
    textTheme: AppTypography.textTheme,
    appBarTheme: const AppBarTheme(centerTitle: true),
    inputDecorationTheme: _inputTheme,
    elevatedButtonTheme: _buttonTheme,
  );

  static ThemeData dark() => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: AppColors.primary,
      brightness: Brightness.dark,
    ),
    textTheme: AppTypography.textTheme,
  );
}
```

## i18n Setup [MEDIUM]

### Using flutter_localizations + intl

1. Enable in `pubspec.yaml`:

```yaml
flutter:
  generate: true

dependencies:
  flutter_localizations:
    sdk: flutter
  intl: any
```

2. Create `l10n.yaml`:

```yaml
arb-dir: lib/core/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
```

3. Create ARB files with translations.

4. Run `flutter gen-l10n`.

## Version-Specific Best Practice Research

Before implementing any code, research the EXACT Flutter/Dart version:

1. **WebSearch**: `"Flutter {version} best practices"`, `"Flutter {version} breaking changes"`
2. **WebFetch**: Official migration guide if upgrading
3. **Check for**:
   - Deprecated widgets or APIs (e.g., `FlatButton` → `TextButton`)
   - New recommended patterns (e.g., Material 3 adoption)
   - Dart language features available (e.g., records, patterns, sealed classes)
   - Package API changes (e.g., `go_router` route definition syntax changes between versions)
4. **Apply findings** to every step — not just architecture

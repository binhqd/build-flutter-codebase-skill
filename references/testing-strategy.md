# Testing Strategy — Flutter

## Contents

- Test structure
- Unit tests
- Widget tests
- Integration tests
- Test utilities and helpers
- Mocking strategy
- Coverage configuration

## Test Structure

```
test/
├── helpers/
│   ├── test_helpers.dart            # Shared setup, mock registration
│   ├── pump_app.dart                # Widget test helper (wraps with MaterialApp)
│   └── fixtures/
│       ├── user_fixture.dart        # Mock entities
│       └── json_fixtures.dart       # Raw JSON responses
├── core/
│   ├── network/
│   │   ├── dio_client_test.dart
│   │   └── interceptors/
│   │       ├── auth_interceptor_test.dart
│   │       └── error_interceptor_test.dart
│   ├── error/
│   │   ├── failures_test.dart
│   │   └── error_handler_test.dart
│   └── utils/
│       └── extensions_test.dart
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── auth_remote_data_source_test.dart
│   │   │   │   └── auth_local_data_source_test.dart
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl_test.dart
│   │   ├── domain/
│   │   │   └── usecases/
│   │   │       ├── login_test.dart
│   │   │       ├── register_test.dart
│   │   │       └── logout_test.dart
│   │   └── presentation/
│   │       ├── bloc/
│   │       │   └── auth_bloc_test.dart
│   │       └── screens/
│   │           ├── login_screen_test.dart
│   │           └── register_screen_test.dart
│   └── home/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── widget_test.dart                 # Default (can remove or keep)

integration_test/
├── app_test.dart                    # Full app flow
└── auth_flow_test.dart              # Auth-specific flow
```

## Unit Tests [LOW]

Unit tests cover business logic in isolation. No Flutter widgets, no real I/O.

### Use Case Tests

```dart
// test/features/auth/domain/usecases/login_test.dart
import 'package:mocktail/mocktail.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:dartz/dartz.dart';

class MockAuthRepository extends Mock implements AuthRepository {}

void main() {
  late Login useCase;
  late MockAuthRepository mockRepository;

  setUp(() {
    mockRepository = MockAuthRepository();
    useCase = Login(mockRepository);
  });

  const tEmail = 'test@example.com';
  const tPassword = 'Password1!';
  final tUser = User(
    id: '1',
    email: tEmail,
    firstName: 'Test',
    lastName: 'User',
    role: UserRole.user,
  );

  test('should return User when login succeeds', () async {
    when(() => mockRepository.login(tEmail, tPassword))
        .thenAnswer((_) async => Right(tUser));

    final result = await useCase(tEmail, tPassword);

    expect(result, Right(tUser));
    verify(() => mockRepository.login(tEmail, tPassword)).called(1);
    verifyNoMoreInteractions(mockRepository);
  });

  test('should return AuthFailure when login fails', () async {
    const failure = AuthFailure('Invalid credentials');
    when(() => mockRepository.login(tEmail, tPassword))
        .thenAnswer((_) async => const Left(failure));

    final result = await useCase(tEmail, tPassword);

    expect(result, const Left(failure));
  });
}
```

### Repository Tests

```dart
// test/features/auth/data/repositories/auth_repository_impl_test.dart
class MockAuthRemoteDataSource extends Mock implements AuthRemoteDataSource {}
class MockAuthLocalDataSource extends Mock implements AuthLocalDataSource {}

void main() {
  late AuthRepositoryImpl repository;
  late MockAuthRemoteDataSource mockRemote;
  late MockAuthLocalDataSource mockLocal;

  setUp(() {
    mockRemote = MockAuthRemoteDataSource();
    mockLocal = MockAuthLocalDataSource();
    repository = AuthRepositoryImpl(mockRemote, mockLocal);
  });

  group('login', () {
    test('should save tokens and return User on success', () async {
      when(() => mockRemote.login(any(), any()))
          .thenAnswer((_) async => tAuthResponse);
      when(() => mockLocal.saveTokens(
          accessToken: any(named: 'accessToken'),
          refreshToken: any(named: 'refreshToken')))
          .thenAnswer((_) async {});
      when(() => mockLocal.saveUser(any()))
          .thenAnswer((_) async {});

      final result = await repository.login('test@example.com', 'pass');

      expect(result.isRight(), true);
      verify(() => mockLocal.saveTokens(
          accessToken: tAuthResponse.accessToken,
          refreshToken: tAuthResponse.refreshToken)).called(1);
    });

    test('should return NetworkFailure when no connection', () async {
      when(() => mockRemote.login(any(), any()))
          .thenThrow(const NetworkException());

      final result = await repository.login('test@example.com', 'pass');

      expect(result, const Left(NetworkFailure()));
    });
  });
}
```

### BLoC Tests

```dart
// test/features/auth/presentation/bloc/auth_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';

class MockLogin extends Mock implements Login {}
class MockLogout extends Mock implements Logout {}
class MockGetCurrentUser extends Mock implements GetCurrentUser {}

void main() {
  late AuthBloc bloc;
  late MockLogin mockLogin;
  late MockLogout mockLogout;
  late MockGetCurrentUser mockGetCurrentUser;

  setUp(() {
    mockLogin = MockLogin();
    mockLogout = MockLogout();
    mockGetCurrentUser = MockGetCurrentUser();
    bloc = AuthBloc(mockLogin, mockLogout, mockGetCurrentUser);
  });

  tearDown(() => bloc.close());

  test('initial state is AuthInitial', () {
    expect(bloc.state, const AuthInitial());
  });

  blocTest<AuthBloc, AuthState>(
    'emits [AuthLoading, AuthAuthenticated] on successful login',
    build: () {
      when(() => mockLogin(any(), any()))
          .thenAnswer((_) async => Right(tUser));
      return bloc;
    },
    act: (bloc) => bloc.add(
      const AuthLoginRequested('test@example.com', 'Password1!'),
    ),
    expect: () => [
      const AuthLoading(),
      AuthAuthenticated(tUser),
    ],
  );

  blocTest<AuthBloc, AuthState>(
    'emits [AuthLoading, AuthError] on failed login',
    build: () {
      when(() => mockLogin(any(), any()))
          .thenAnswer((_) async => const Left(AuthFailure('Invalid')));
      return bloc;
    },
    act: (bloc) => bloc.add(
      const AuthLoginRequested('test@example.com', 'wrong'),
    ),
    expect: () => [
      const AuthLoading(),
      const AuthError('Invalid'),
    ],
  );
}
```

## Widget Tests [MEDIUM]

Widget tests verify UI rendering and interaction. Use `pumpApp` helper to wrap widgets with required providers.

### Pump App Helper

```dart
// test/helpers/pump_app.dart
extension PumpApp on WidgetTester {
  Future<void> pumpApp(
    Widget widget, {
    AuthBloc? authBloc,
    GoRouter? router,
  }) {
    return pumpWidget(
      MaterialApp(
        home: widget,
        // Add BlocProviders, theme, localization as needed
      ),
    );
  }

  Future<void> pumpRouterApp({
    required GoRouter router,
    AuthBloc? authBloc,
  }) {
    return pumpWidget(
      MaterialApp.router(
        routerConfig: router,
        // Add BlocProviders, theme, localization as needed
      ),
    );
  }
}
```

### Screen Tests

```dart
// test/features/auth/presentation/screens/login_screen_test.dart
void main() {
  late MockAuthBloc mockAuthBloc;

  setUp(() {
    mockAuthBloc = MockAuthBloc();
  });

  group('LoginScreen', () {
    testWidgets('renders email and password fields', (tester) async {
      when(() => mockAuthBloc.state).thenReturn(const AuthInitial());

      await tester.pumpApp(
        BlocProvider<AuthBloc>.value(
          value: mockAuthBloc,
          child: const LoginScreen(),
        ),
      );

      expect(find.byType(TextFormField), findsNWidgets(2));
      expect(find.text('Email'), findsOneWidget);
      expect(find.text('Password'), findsOneWidget);
      expect(find.text('Login'), findsOneWidget);
    });

    testWidgets('shows error when validation fails', (tester) async {
      when(() => mockAuthBloc.state).thenReturn(const AuthInitial());

      await tester.pumpApp(
        BlocProvider<AuthBloc>.value(
          value: mockAuthBloc,
          child: const LoginScreen(),
        ),
      );

      // Tap login without filling fields
      await tester.tap(find.text('Login'));
      await tester.pumpAndSettle();

      expect(find.text('Email is required'), findsOneWidget);
    });

    testWidgets('shows loading indicator when submitting', (tester) async {
      when(() => mockAuthBloc.state).thenReturn(const AuthLoading());

      await tester.pumpApp(
        BlocProvider<AuthBloc>.value(
          value: mockAuthBloc,
          child: const LoginScreen(),
        ),
      );

      expect(find.byType(CircularProgressIndicator), findsOneWidget);
    });

    testWidgets('shows error message on AuthError state', (tester) async {
      when(() => mockAuthBloc.state)
          .thenReturn(const AuthError('Invalid credentials'));

      await tester.pumpApp(
        BlocProvider<AuthBloc>.value(
          value: mockAuthBloc,
          child: const LoginScreen(),
        ),
      );

      expect(find.text('Invalid credentials'), findsOneWidget);
    });
  });
}
```

## Integration Tests [MEDIUM]

Integration tests run on a real device/emulator and test complete flows.

### Auth Flow Test

```dart
// integration_test/auth_flow_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Auth Flow', () {
    testWidgets('login → home → logout → login', (tester) async {
      await tester.pumpWidget(const App());
      await tester.pumpAndSettle();

      // Should start at login screen
      expect(find.text('Login'), findsOneWidget);

      // Fill in credentials
      await tester.enterText(
        find.byKey(const Key('emailField')),
        'admin@example.com',
      );
      await tester.enterText(
        find.byKey(const Key('passwordField')),
        'Admin123!',
      );

      // Tap login
      await tester.tap(find.byKey(const Key('loginButton')));
      await tester.pumpAndSettle();

      // Should navigate to home
      expect(find.text('Home'), findsOneWidget);

      // Logout
      await tester.tap(find.byIcon(Icons.logout));
      await tester.pumpAndSettle();

      // Should return to login
      expect(find.text('Login'), findsOneWidget);
    });
  });
}
```

### Running Integration Tests

```bash
# On a connected device/emulator
flutter test integration_test/

# Specific test
flutter test integration_test/auth_flow_test.dart
```

## Mocking Strategy

### Preferred: mocktail (no code generation)

```dart
import 'package:mocktail/mocktail.dart';

class MockAuthRepository extends Mock implements AuthRepository {}
class MockDioClient extends Mock implements DioClient {}
```

### Alternative: mockito (with code generation)

```dart
import 'package:mockito/annotations.dart';

@GenerateNiceMocks([
  MockSpec<AuthRepository>(),
  MockSpec<DioClient>(),
])
import 'auth_repository_test.mocks.dart';
```

### Mock BLoC for Widget Tests

```dart
import 'package:bloc_test/bloc_test.dart';

class MockAuthBloc extends MockBloc<AuthEvent, AuthState>
    implements AuthBloc {}
```

### Dio Mock Adapter

```dart
import 'package:http_mock_adapter/http_mock_adapter.dart';

final dio = Dio();
final dioAdapter = DioAdapter(dio: dio);

// Mock a response
dioAdapter.onPost(
  ApiEndpoints.login,
  (server) => server.reply(200, {
    'success': true,
    'data': {'access_token': 'mock-token', ...},
  }),
  data: {'email': 'test@example.com', 'password': 'pass'},
);
```

## Coverage Configuration [LOW]

### Run with Coverage

```bash
# Generate coverage
flutter test --coverage

# View report (requires lcov)
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```

### Coverage Thresholds

- **Target:** >90% line coverage on `lib/` (excluding generated files)
- **Must cover:** All use cases, repositories, BLoCs, interceptors
- **Can exclude:** Generated files (`*.g.dart`, `*.freezed.dart`), main entry points

### Exclude Generated Files from Coverage

```bash
# Remove generated files from coverage report
flutter test --coverage
lcov --remove coverage/lcov.info \
  '**/*.g.dart' \
  '**/*.freezed.dart' \
  '**/*.config.dart' \
  '**/*.mocks.dart' \
  '**/l10n/**' \
  -o coverage/lcov_cleaned.info
```

### CI Coverage Check

```yaml
# In CI pipeline
- name: Run tests with coverage
  run: flutter test --coverage

- name: Check coverage threshold
  run: |
    COVERAGE=$(lcov --summary coverage/lcov.info | grep 'lines' | grep -oP '\d+\.\d+')
    if (( $(echo "$COVERAGE < 90" | bc -l) )); then
      echo "Coverage $COVERAGE% is below 90% threshold"
      exit 1
    fi
```

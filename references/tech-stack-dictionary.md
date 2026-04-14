# Tech Stack Dictionary — Flutter

## Contents

- Flutter & Dart versions
- State management options
- Key packages by category
- Version research instructions
- Dependency security & compatibility verification
- Known deprecated/unmaintained packages

## Flutter & Dart Versions

| Component | Research URL |
|-----------|-------------|
| Flutter SDK | https://docs.flutter.dev/release/archive |
| Dart SDK | Bundled with Flutter — no separate install |

Always use **stable channel**. Never use beta/dev for production apps.

## State Management Options

Choose ONE. This is the most impactful architectural decision.

### Option A: BLoC (flutter_bloc)

| Aspect | Details |
|--------|---------|
| Package | `flutter_bloc` |
| Pattern | Event → BLoC → State (unidirectional) |
| Best for | Large teams, strict separation, testable business logic |
| Learning curve | Higher — requires understanding Events, States, BLoC classes |
| Boilerplate | More — but predictable and consistent |
| Testing | Excellent — `bloc_test` package, deterministic state transitions |
| Recommended when | Enterprise apps, teams >3 devs, complex state flows |

### Option B: Riverpod

| Aspect | Details |
|--------|---------|
| Package | `flutter_riverpod` + `riverpod_annotation` |
| Pattern | Provider-based with code generation |
| Best for | Smaller teams, rapid iteration, compile-safe DI |
| Learning curve | Medium — simpler than BLoC but unique concepts |
| Boilerplate | Less with code generation |
| Testing | Excellent — provider overrides in tests |
| Recommended when | Small-medium teams, projects needing fast iteration |

## Key Packages by Category

### Core Architecture

| Category | Package | Purpose |
|----------|---------|---------|
| State (BLoC) | `flutter_bloc` | BLoC pattern implementation |
| State (BLoC) | `bloc_test` | Testing utilities for BLoC |
| State (Riverpod) | `flutter_riverpod` | Riverpod state management |
| State (Riverpod) | `riverpod_annotation` | Code generation for Riverpod |
| DI | `get_it` | Service locator |
| DI | `injectable` + `injectable_generator` | Code generation for GetIt |
| Code Gen | `build_runner` | Build system for code generation |
| Code Gen | `freezed` + `freezed_annotation` | Immutable data classes + unions |
| Code Gen | `json_serializable` + `json_annotation` | JSON serialization |
| Functional | `dartz` or `fpdart` | Either, Option for error handling |
| Equality | `equatable` | Value equality for states/entities |

### Networking & Data

| Category | Package | Purpose |
|----------|---------|---------|
| HTTP | `dio` | HTTP client with interceptors |
| Local DB | `drift` + `drift_dev` | Type-safe SQLite (formerly moor) |
| Local DB (alt) | `hive` + `hive_flutter` | NoSQL local storage |
| Key-Value | `shared_preferences` | Simple key-value persistence |
| Secure Storage | `flutter_secure_storage` | Encrypted storage for tokens |
| Connectivity | `connectivity_plus` | Network status monitoring |

### Navigation & UI

| Category | Package | Purpose |
|----------|---------|---------|
| Routing | `go_router` | Declarative routing with deep linking |
| i18n | `intl` + `flutter_localizations` | Internationalization (official) |
| i18n (alt) | `easy_localization` | Simpler i18n alternative |
| Images | `cached_network_image` | Image caching |
| SVG | `flutter_svg` | SVG rendering |
| Shimmer | `shimmer` | Loading placeholder animations |

### Auth & Security

| Category | Package | Purpose |
|----------|---------|---------|
| Biometric | `local_auth` | Fingerprint / Face ID |
| OAuth | `flutter_appauth` | OAuth2 / OIDC client |

### Push Notifications & Firebase

| Category | Package | Purpose |
|----------|---------|---------|
| Firebase Core | `firebase_core` | Firebase initialization |
| Push | `firebase_messaging` | FCM push notifications |
| Analytics | `firebase_analytics` | Event tracking |
| Crashlytics | `firebase_crashlytics` | Crash reporting |

### Testing

| Category | Package | Purpose |
|----------|---------|---------|
| Mocking | `mocktail` | Mock generation (no codegen) |
| Mocking (alt) | `mockito` + `build_runner` | Mock generation (with codegen) |
| BLoC test | `bloc_test` | BLoC-specific test utilities |
| Network mock | `http_mock_adapter` | Dio mock adapter |
| Golden | `golden_toolkit` | Golden (snapshot) testing |

### Dev Tools

| Category | Package | Purpose |
|----------|---------|---------|
| Linting | `flutter_lints` or `very_good_analysis` | Lint rules |
| Env | `envied` + `envied_generator` | Compile-time env vars |
| Logging | `logger` | Formatted console logging |

## Version Research Instructions

When starting a new project, search for current stable versions:

1. **Flutter SDK**: Search `"flutter stable channel latest version"` or check https://docs.flutter.dev/release/archive
2. **Dart SDK**: Comes bundled — check `flutter --version` output
3. **Each package**: Check pub.dev for latest version and SDK constraints
4. **Compatibility matrix**: Verify all packages support the same Dart SDK lower bound

## Dependency Security & Compatibility Verification

For EVERY package before adopting:

### 1. Compatibility Check
- Visit `pub.dev/packages/{package}` → check "SDK" and "Flutter" constraints
- Ensure it supports the confirmed Dart SDK version
- Check for platform support markers (Android, iOS, Web, etc.)

### 2. CVE / Security Check
- WebSearch `"{package} CVE vulnerability"`
- Check GitHub repo Issues tab for security-related issues
- Check pub.dev for "discontinued" badge

### 3. Maintenance Check
- Last publish date on pub.dev (must be <12 months)
- GitHub repo: last commit date, open issues ratio
- pub.dev score (prefer likes >100, pub points >100)
- No "discontinued" or "unlisted" markers

### 4. Decision
- **Use**: Passes all 3 checks
- **Skip + find alternative**: Fails any check

## Known Deprecated / Unmaintained Packages

| Package | Status | Replacement |
|---------|--------|-------------|
| `moor` | Renamed | `drift` |
| `provider` | Maintained but older pattern | `flutter_riverpod` (if greenfield) |
| `http` | Limited | `dio` (if interceptors needed) |
| `sqflite` | Maintained but raw SQL | `drift` (type-safe) |
| `flutter_bloc` <8.0 | Outdated API | `flutter_bloc` >=8.0 |
| `get` (GetX) | Controversial architecture | `flutter_bloc` or `riverpod` |
| `pedantic` | Deprecated | `flutter_lints` or `very_good_analysis` |

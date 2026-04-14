# Pre-flight Check — Flutter

## Purpose

Run these checks before starting ANY workflow (except Build Codebase Phase 0-2, where the project doesn't exist yet). All checks must pass before proceeding.

## Checks [LOW]

### 1. Flutter SDK Available

```bash
flutter --version
```

- Must show stable channel
- Version must match project's `pubspec.yaml` SDK constraint
- If mismatch: `flutter channel stable && flutter upgrade`

### 2. Dependencies Resolved

```bash
flutter pub get
```

- Must complete without errors
- If `pubspec.lock` is missing or outdated, this regenerates it

### 3. Code Generation Up-to-date

```bash
dart run build_runner build --delete-conflicting-outputs
```

- Must complete without errors
- Generated files (`*.g.dart`, `*.freezed.dart`, `*.config.dart`) must be current
- If errors: check `pubspec.yaml` for version conflicts in build_runner dependencies

### 4. Static Analysis Clean

```bash
flutter analyze
```

- Must report zero errors
- Warnings are acceptable but should be reviewed
- If errors: fix before proceeding — do NOT suppress with `// ignore`

### 5. Existing Tests Pass

```bash
flutter test
```

- All existing tests must pass
- If tests fail: investigate and fix before starting new work
- Never start a workflow on a broken test suite

### 6. Project Structure Intact

Verify the Clean Architecture structure exists:

```
lib/
├── core/
│   ├── config/
│   ├── di/
│   ├── error/
│   ├── network/
│   ├── router/
│   ├── storage/
│   ├── theme/
│   └── widgets/
├── features/
│   └── auth/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── main_dev.dart (or main.dart)
```

- If folders are missing: the codebase may not have been built yet → use Build Codebase workflow first

### 7. Environment Config Present

- `.env` or equivalent config file exists (if applicable)
- Flavors configured (dev/staging/prod entry points)
- At least `main_dev.dart` exists and is runnable

### 8. Device / Emulator Available

```bash
flutter devices
```

- At least one device or emulator must be listed
- For widget/unit tests: no device needed
- For integration tests: device or emulator required

## Result

| Check | Status |
|-------|--------|
| Flutter SDK | [ ] pass |
| Dependencies | [ ] pass |
| Code generation | [ ] pass |
| Static analysis | [ ] pass |
| Tests pass | [ ] pass |
| Project structure | [ ] pass |
| Environment config | [ ] pass |
| Device available | [ ] pass |

**All pass** → Proceed with workflow.
**Any fail** → Fix before proceeding. Do not skip checks.

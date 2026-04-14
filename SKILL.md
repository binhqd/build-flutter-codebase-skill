---
name: building-flutter-app
description: Scaffolds production-ready Flutter mobile app codebases with Clean Architecture (Presentation → Domain → Data), state management (BLoC/Riverpod), Dio API client, local storage, JWT auth (client-side), error handling (Either/Failure pattern), theming, i18n, navigation (GoRouter), flavors (dev/staging/prod), and testing (unit + widget + integration, >90% coverage). Use when creating a new Flutter app, setting up mobile boilerplate, scaffolding app architecture, or bootstrapping a cross-platform mobile project.
---

# Building Flutter App

Build production-ready Flutter mobile app codebases following proven architectural patterns.

## Freedom Levels

- **[LOW]** — Follow exactly. Use exact templates/commands from references. No improvisation.
- **[MEDIUM]** — Preferred pattern exists, adapt syntax to project context.
- **[HIGH]** — Multiple approaches valid. Use judgment based on context.

## Before Starting: Initialize Tracker + Memory [LOW]

1. Look for `.build-tracker.md` in project root.
   - **Found**: Read it → resume from current phase → log new session in Session History.
   - **Not found**: Read [references/tracker-template.md](references/tracker-template.md) → create `.build-tracker.md` in project root.
2. If running in Claude Code, save a memory entry (project name, directory, tech stack, current phase). Update memory when a phase completes or a major decision changes.
3. **Tracker discipline**: At every step transition, read tracker → mark completed `[x] done` → mark next `[~] in_progress` → update "Current Phase" and "Last Updated" → write back. The tracker is the source of truth across sessions.

---

## Workflow

### Phase 0: Tech Stack Discovery

**Step 0.1 — Confirm Flutter + Dart [HIGH]**

Confirm the user wants Flutter. If they mentioned it already, acknowledge rather than re-asking. Ask about target platforms (iOS, Android, or both).

**Step 0.2 — Choose state management approach [HIGH]**

Read [references/tech-stack-dictionary.md](references/tech-stack-dictionary.md). Present the two recommended approaches (BLoC vs Riverpod) with trade-offs. Let the user pick.

**Step 0.3 — Research latest stable versions [HIGH]**

Use **WebSearch** to find current stable versions:
1. Flutter SDK stable channel
2. Dart SDK (bundled with Flutter)
3. Key packages from tech-stack-dictionary.md
4. Present version table. Ask user to confirm.

Update tracker: Decisions D1-D5.

**Step 0.4 — Fetch latest Flutter documentation [MEDIUM]**

Use **WebFetch** to retrieve official docs for the confirmed Flutter + Dart version:
1. Flutter "getting started" or migration guide
2. Breaking changes in current stable
3. State management package docs (BLoC or Riverpod)
4. Note deprecated APIs, new features

This prevents using outdated patterns from training data. Save findings as context.

**Step 0.5 — Dependency security & compatibility check [LOW]**

For EVERY package from the dictionary's key packages table, verify before adopting. Follow the "Dependency Security & Compatibility Verification" section in [references/tech-stack-dictionary.md](references/tech-stack-dictionary.md):

1. **Compatibility**: WebSearch each package to confirm it supports the confirmed Flutter/Dart version. Check pub.dev for SDK constraints.
2. **CVE check**: WebSearch `"{package} CVE vulnerability"` + check GitHub Advisories.
3. **Maintenance check**: Verify last publish <12 months on pub.dev, no "discontinued" marker.
4. **Replace flagged packages**: If a package is unmaintained or has CVEs, find alternative from dictionary or via search.
5. **Present Security Report**: Show table with each package's compatibility, CVE status, maintenance status, and action (Use / Skip + alternative).

Only proceed with packages that pass all three checks. Update tracker with final package list.

**Step 0.6 — Present stack summary [HIGH]**

From the dictionary + fetched docs + security report, present:
1. Architecture pattern (Clean Architecture layers)
2. Recommended project structure
3. **Verified packages** table (only packages that passed security check)
4. Key DO/DON'T rules from [references/implementation-guidelines.md](references/implementation-guidelines.md)

Ask user to confirm. Update tracker: Decisions D6-D7. Save confirmed stack to memory.

---

### Phase 1: Project Assessment [HIGH]

Identify required features through conversation:

| Feature | Default | Ask if unclear |
|---------|---------|---------------|
| Target platforms | iOS + Android | Web, macOS, Windows |
| Auth method | JWT (client-side) | OAuth2, SAML2, biometric |
| State management | (from Phase 0) | — |
| Local storage | SharedPreferences + SQLite | Hive, Isar |
| Offline support | none | offline-first, cache-only |
| Push notifications | none | FCM, APNs |
| i18n | none | multi-language |
| Deep linking | none | universal links, app links |
| File upload | none | camera, gallery, documents |
| Analytics | none | Firebase, Mixpanel |

Update tracker: Decisions D8-D15. Read relevant reference files based on confirmed features.

---

### Phase 2: Project Scaffold [MIXED]

**Step 2.1 — Run flutter create [LOW]**

```bash
flutter create --org <org_domain> --project-name <name> --platforms android,ios .
```

Use the official scaffold command. Do NOT manually create Flutter project files.

**Step 2.2 — Configure analysis_options.yaml [LOW]**

Apply strict lint rules. See [references/implementation-guidelines.md](references/implementation-guidelines.md) for the exact configuration.

**Step 2.3 — Set up flavors (dev / staging / prod) [LOW]**

See [references/build-deploy.md](references/build-deploy.md). Configure:
- Android: `productFlavors` in `build.gradle`
- iOS: Xcode schemes + xcconfig files
- Dart: `--dart-define` or `--flavor` flag

**Step 2.4 — Restructure to Clean Architecture [MEDIUM]**

Adapt generated structure to match the layered architecture from [references/architecture-layers.md](references/architecture-layers.md). Create the full directory tree (core/, features/, shared/).

**Step 2.5 — Install dependencies [LOW]**

Add verified packages from Phase 0 security report to `pubspec.yaml`. Use exact version constraints (`^x.y.z`). Run `flutter pub get`.

**Step 2.6 — Verify scaffold runs [LOW]**

```bash
flutter analyze   # No errors
flutter run       # App starts on emulator/device
```

Fix before proceeding. Update tracker after each step.

---

### Phase 3: Implement Core Layer [MIXED]

**Before writing ANY code in this phase**, do both:

1. Read [references/implementation-guidelines.md](references/implementation-guidelines.md) for Dart/Flutter rules.
2. **Research version-specific best practices [MEDIUM]** — Use **WebSearch** and **WebFetch** to find the official best practices for the EXACT Flutter/Dart version confirmed in Phase 0.

**Step 3.1 — Environment Configuration [LOW]**
- Flavor-aware config (API base URL, feature flags per environment)
- See [references/architecture-layers.md](references/architecture-layers.md)

**Step 3.2 — Theme & Design System [MEDIUM]**
- Material 3 ThemeData (light + dark)
- Color scheme, typography, spacing constants
- See [references/architecture-layers.md](references/architecture-layers.md)

**Step 3.3 — Navigation / Routing [LOW]**
- GoRouter setup with typed routes
- Auth-aware redirect logic
- Deep linking config
- See [references/navigation-routing.md](references/navigation-routing.md)

**Step 3.4 — Error Handling [LOW]**
- Failure class hierarchy
- Either<Failure, T> pattern
- Global error handler (FlutterError, PlatformDispatcher)
- See [references/error-handling.md](references/error-handling.md)

**Step 3.5 — API Client (Dio) [LOW]**
- Dio instance with interceptors: auth token, logging, error mapping, retry
- Base API service class
- See [references/api-client.md](references/api-client.md)

**Step 3.6 — Local Storage [MEDIUM]**
- SharedPreferences wrapper for simple key-value
- SQLite/Drift or Hive for structured data (if needed)
- flutter_secure_storage for tokens
- See [references/local-storage.md](references/local-storage.md)

**Step 3.7 — Dependency Injection [LOW]**
- GetIt + Injectable setup
- Register all services, repositories, blocs/providers
- See [references/architecture-layers.md](references/architecture-layers.md)

**Step 3.8 — i18n Setup [MEDIUM]** (if confirmed in Phase 1)
- flutter_localizations + intl or easy_localization
- ARB files for each locale
- See [references/architecture-layers.md](references/architecture-layers.md)

**Feedback loop for each step:** After implementing, validate immediately:
1. Run `flutter analyze` — must have zero errors
2. Run existing tests if any — must pass
3. Update tracker after EACH step

---

### Phase 4: Implement Auth Feature [LOW]

This is the first full feature, serving as a reference implementation for the architecture.

**Before writing code**, read [references/authentication.md](references/authentication.md).

**Step 4.1 — Auth Data Layer [LOW]**
- AuthRemoteDataSource: login, register, refreshToken, logout API calls via Dio
- AuthLocalDataSource: save/read/delete tokens via flutter_secure_storage
- AuthRepository implementation

**Step 4.2 — Auth Domain Layer [LOW]**
- User entity
- AuthRepository abstract class (interface)
- Use cases: Login, Register, Logout, GetCurrentUser, RefreshToken

**Step 4.3 — Auth Presentation Layer [MEDIUM]**
- Auth BLoC/Riverpod (state: initial, loading, authenticated, unauthenticated, error)
- Login screen (email + password)
- Register screen
- Form validation

**Step 4.4 — Auth-aware Navigation [LOW]**
- GoRouter redirect: unauthenticated → login, authenticated → home
- Token refresh on 401 via Dio interceptor
- Auto-logout on refresh failure

**Step 4.5 — Verify Auth Flow [LOW]**
- Run app → login screen appears
- Login with valid credentials → navigates to home
- Kill app → reopen → still authenticated (token persisted)
- Token expired → auto-refresh → still works
- Refresh token expired → redirected to login

Update tracker after each step.

---

### Phase 5: Implement Base Screens [MEDIUM]

**Step 5.1 — Splash Screen [LOW]**
- Check auth state, route accordingly
- App branding

**Step 5.2 — Home Screen [MEDIUM]**
- Bottom navigation or drawer (based on app needs)
- Demonstrates list → detail pattern

**Step 5.3 — Profile / Settings Screen [MEDIUM]**
- Display user info from auth state
- Theme toggle (light/dark)
- Language switch (if i18n enabled)
- Logout action

**Step 5.4 — Error / Empty / Loading States [LOW]**
- Reusable widgets: ErrorView, EmptyView, LoadingView
- Consistent across all screens
- See [references/error-handling.md](references/error-handling.md)

Update tracker after each step.

---

### Phase 6: Comprehensive Testing [MIXED]

Target: **>90% coverage** on business logic. Read [references/testing-strategy.md](references/testing-strategy.md).

**Step 6.1 — Test config + utilities [MEDIUM]**
- Test setup: mock generation (mockito/mocktail), test helpers
- Run `dart run build_runner build` for generated mocks

**Step 6.2-6.6 — Unit Tests**

| Step | Target | Freedom | Must Cover |
|------|--------|---------|------------|
| 6.2 | Use cases | LOW | All business logic, edge cases, failures |
| 6.3 | Repositories | LOW | Success + failure paths, data source mapping |
| 6.4 | BLoC / Providers | LOW | All state transitions, events, error states |
| 6.5 | API client & interceptors | MEDIUM | Token injection, refresh on 401, error mapping |
| 6.6 | Utilities & extensions | LOW | All helper functions |

**Step 6.7-6.10 — Widget Tests**

| Step | Target | Freedom | Must Cover |
|------|--------|---------|------------|
| 6.7 | Login screen | LOW | Render, validation, submit, error display |
| 6.8 | Register screen | LOW | Render, validation, submit |
| 6.9 | Home screen | MEDIUM | Render with data, empty state, error state |
| 6.10 | Reusable widgets | LOW | ErrorView, EmptyView, LoadingView |

**Step 6.11-6.12 — Integration Tests**

| Step | Target | Freedom | Must Cover |
|------|--------|---------|------------|
| 6.11 | Auth flow | LOW | Login → home → logout → login |
| 6.12 | Navigation | LOW | Deep linking, redirect guards |

**Step 6.13 — Run tests + verify [LOW]**

```bash
flutter test --coverage
```

All must pass. Coverage must be >90% on lib/ (excluding generated code). Fix failures before proceeding.

Update tracker after each step group.

---

### Phase 7: Build & Deploy Setup [LOW]

Read [references/build-deploy.md](references/build-deploy.md).

| Step | Task | Freedom | Deliverable |
|------|------|---------|-------------|
| 7.1 | Android signing config | LOW | keystore setup, build.gradle signing configs |
| 7.2 | iOS signing config | LOW | Xcode provisioning profiles, export options |
| 7.3 | Build scripts | LOW | Makefile or shell scripts for each flavor |
| 7.4 | CI/CD pipeline | MEDIUM | GitHub Actions / Codemagic / Fastlane config |
| 7.5 | Verify builds | LOW | `flutter build apk --flavor dev` + `flutter build ios --flavor dev` |

Update tracker after each step.

---

### Phase 8: Completion Report [HIGH]

Present full summary to user including:
- Tech stack with versions
- All features included
- Architecture overview (layer diagram)
- Quick start commands (copy-pasteable)
- Project structure tree
- Key files with descriptions
- Build commands per flavor
- Test coverage report

Update memory with: project complete, final stack, any issues encountered.

---

### Phase 9: Final Verification [LOW]

Walk through the verification checklist in the tracker (Phase 9 section). Every item must pass. If any item fails, return to the relevant phase and fix before marking complete.

Update memory: project verification complete.

---

## Reference Files

Read based on current phase:

**Phase 0 (Discovery):**
- [references/tech-stack-dictionary.md](references/tech-stack-dictionary.md) — Flutter, Dart, packages, versions

**Phase 2-3 (Scaffold + Core):**
- [references/architecture-layers.md](references/architecture-layers.md) — Clean Architecture, DI, theming, i18n
- [references/implementation-guidelines.md](references/implementation-guidelines.md) — Dart/Flutter DO/DON'T, lint rules
- [references/navigation-routing.md](references/navigation-routing.md) — GoRouter, deep linking
- [references/error-handling.md](references/error-handling.md) — Failures, Either, UI error states
- [references/api-client.md](references/api-client.md) — Dio, interceptors, offline
- [references/local-storage.md](references/local-storage.md) — SQLite, Hive, secure storage

**Phase 4 (Auth):**
- [references/authentication.md](references/authentication.md) — Token storage, biometric, OAuth client

**Phase 6 (Testing):**
- [references/testing-strategy.md](references/testing-strategy.md) — Unit, widget, integration tests

**Phase 7 (Build):**
- [references/build-deploy.md](references/build-deploy.md) — Flavors, signing, CI/CD, store deploy

**Tracker:**
- [references/tracker-template.md](references/tracker-template.md) — Tracker template and update rules

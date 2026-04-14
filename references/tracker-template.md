# Build Tracker Template

## About This File

This tracker is created at the start of the skill workflow and updated at every step. It serves two purposes:

1. **Progress tracking** — know exactly where work left off
2. **Decision log** — preserve all choices so a new session has full context to resume

Create this file as `.build-tracker.md` in the project root directory.

---

## Tracker Template

Copy the content below into `.build-tracker.md` when starting a new project:

```markdown
# Build Tracker

> **Project**: {project-name}
> **Created**: {date}
> **Last Updated**: {date + time}
> **Current Phase**: Phase 0 — Tech Stack Discovery

---

## Decisions Log

Record every user decision here so any session can resume with full context.

| # | Decision | Value | Confirmed | Date |
|---|----------|-------|-----------|------|
| D1 | Flutter SDK Version | - | [ ] | - |
| D2 | Dart SDK Version | - | [ ] | - |
| D3 | State Management | - | [ ] | - |
| D4 | Target Platforms | iOS + Android | [ ] | - |
| D5 | Architecture Pattern | Clean Architecture | [ ] | - |
| D6 | Project Structure | - | [ ] | - |
| D7 | Verified Packages | - | [ ] | - |
| D8 | Auth Method | JWT (client-side) | [ ] | - |
| D9 | Local Storage | SharedPreferences + SQLite | [ ] | - |
| D10 | Offline Support | none | [ ] | - |
| D11 | Push Notifications | none | [ ] | - |
| D12 | i18n | none | [ ] | - |
| D13 | Deep Linking | none | [ ] | - |
| D14 | File Upload | none | [ ] | - |
| D15 | Analytics | none | [ ] | - |

---

## Phase 0: Tech Stack Discovery [HIGH FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 0.1 | Confirm Flutter + Dart | HIGH | [ ] pending | |
| 0.2 | Choose state management | HIGH | [ ] pending | BLoC or Riverpod |
| 0.3 | Research latest stable versions | HIGH | [ ] pending | |
| 0.4 | Fetch latest Flutter docs | MEDIUM | [ ] pending | |
| 0.5 | Dependency security & compatibility check | LOW | [ ] pending | |
| 0.6 | Present stack summary | HIGH | [ ] pending | |

## Phase 1: Project Assessment [HIGH FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 1.1 | Identify target platforms | HIGH | [ ] pending | |
| 1.2 | Identify auth method | HIGH | [ ] pending | |
| 1.3 | Identify local storage needs | HIGH | [ ] pending | |
| 1.4 | Identify offline support | HIGH | [ ] pending | |
| 1.5 | Identify push notifications | HIGH | [ ] pending | |
| 1.6 | Identify i18n needs | HIGH | [ ] pending | |
| 1.7 | Identify deep linking | HIGH | [ ] pending | |
| 1.8 | Identify file upload | HIGH | [ ] pending | |
| 1.9 | Identify analytics | HIGH | [ ] pending | |

## Phase 2: Project Scaffold [MIXED FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 2.1 | Run flutter create | LOW | [ ] pending | |
| 2.2 | Configure analysis_options.yaml | LOW | [ ] pending | |
| 2.3 | Set up flavors (dev/staging/prod) | LOW | [ ] pending | Android + iOS |
| 2.4 | Restructure to Clean Architecture | MEDIUM | [ ] pending | |
| 2.5 | Install dependencies | LOW | [ ] pending | Verified packages only |
| 2.6 | Verify scaffold runs | LOW | [ ] pending | flutter analyze + flutter run |

## Phase 3: Implement Core Layer [MIXED FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 3.1 | Environment configuration | LOW | [ ] pending | Flavor-aware config |
| 3.2 | Theme & design system | MEDIUM | [ ] pending | Material 3, light + dark |
| 3.3 | Navigation / routing | LOW | [ ] pending | GoRouter + auth redirect |
| 3.4 | Error handling | LOW | [ ] pending | Failure hierarchy + Either |
| 3.5 | API client (Dio) | LOW | [ ] pending | Interceptors: auth, logging, error |
| 3.6 | Local storage | MEDIUM | [ ] pending | Secure storage + SharedPreferences |
| 3.7 | Dependency injection | LOW | [ ] pending | GetIt + Injectable |
| 3.8 | i18n setup | MEDIUM | [ ] pending | If confirmed in Phase 1 |

## Phase 4: Implement Auth Feature [LOW FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 4.1 | Auth data layer | LOW | [ ] pending | Remote + Local data sources |
| 4.2 | Auth domain layer | LOW | [ ] pending | Entities, repo interface, use cases |
| 4.3 | Auth presentation layer | MEDIUM | [ ] pending | BLoC/Provider + screens |
| 4.4 | Auth-aware navigation | LOW | [ ] pending | GoRouter redirect + token refresh |
| 4.5 | Verify auth flow | LOW | [ ] pending | End-to-end manual test |

## Phase 5: Implement Base Screens [MEDIUM FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 5.1 | Splash screen | LOW | [ ] pending | Auth check + routing |
| 5.2 | Home screen | MEDIUM | [ ] pending | Bottom nav or drawer |
| 5.3 | Profile / Settings screen | MEDIUM | [ ] pending | User info, theme toggle, logout |
| 5.4 | Error / Empty / Loading states | LOW | [ ] pending | Reusable widgets |

## Phase 6: Comprehensive Testing [MIXED FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 6.1 | Test config + utilities | MEDIUM | [ ] pending | Mocks, helpers, fixtures |
| 6.2 | Unit: Use cases | LOW | [ ] pending | All business logic |
| 6.3 | Unit: Repositories | LOW | [ ] pending | Success + failure paths |
| 6.4 | Unit: BLoC / Providers | LOW | [ ] pending | All state transitions |
| 6.5 | Unit: API client & interceptors | MEDIUM | [ ] pending | Token, refresh, error mapping |
| 6.6 | Unit: Utilities & extensions | LOW | [ ] pending | All helpers |
| 6.7 | Widget: Login screen | LOW | [ ] pending | Render, validate, submit, error |
| 6.8 | Widget: Register screen | LOW | [ ] pending | Render, validate, submit |
| 6.9 | Widget: Home screen | MEDIUM | [ ] pending | Data, empty, error states |
| 6.10 | Widget: Reusable widgets | LOW | [ ] pending | ErrorView, EmptyView, LoadingView |
| 6.11 | Integration: Auth flow | LOW | [ ] pending | Login → home → logout |
| 6.12 | Integration: Navigation | LOW | [ ] pending | Deep linking, guards |
| 6.13 | Run tests + verify coverage | LOW | [ ] pending | All pass, coverage >90% |

## Phase 7: Build & Deploy Setup [LOW FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 7.1 | Android signing config | LOW | [ ] pending | Keystore + build.gradle |
| 7.2 | iOS signing config | LOW | [ ] pending | Provisioning + export options |
| 7.3 | Build scripts (Makefile) | LOW | [ ] pending | All flavors |
| 7.4 | CI/CD pipeline | MEDIUM | [ ] pending | GitHub Actions / Codemagic |
| 7.5 | Verify builds | LOW | [ ] pending | APK + IPA for dev flavor |

## Phase 8: Completion Report [HIGH FREEDOM]

| Step | Task | Freedom | Status | Notes |
|------|------|---------|--------|-------|
| 8.1 | Generate completion summary | HIGH | [ ] pending | Stack, features, commands, structure |

## Phase 9: Verification Checklist [LOW FREEDOM]

Every item must pass. If any fails, return to the relevant phase.

### Scaffold
- [ ] flutter analyze: zero errors
- [ ] Flavors: dev, staging, prod all build
- [ ] Clean Architecture folder structure in place

### Core
- [ ] Environment config: flavor-aware, all envs work
- [ ] Theme: light + dark, Material 3
- [ ] GoRouter: auth redirect working
- [ ] Dio: interceptors (auth, logging, error) attached
- [ ] Secure storage: tokens save/read/delete
- [ ] DI: all dependencies registered and resolvable

### Auth
- [ ] Login: email + password → tokens saved → home
- [ ] Token persistence: kill app → reopen → still authenticated
- [ ] Token refresh: expired access → auto-refresh → works
- [ ] Logout: tokens cleared → login screen
- [ ] Session expired: refresh fails → login screen

### Error Handling
- [ ] Failure hierarchy implemented (sealed class)
- [ ] Either pattern in repositories
- [ ] Global error handler catches uncaught errors
- [ ] UI: ErrorView, EmptyView, LoadingView reusable

### Testing
- [ ] Unit tests: use cases, repositories, BLoCs
- [ ] Widget tests: screens render correctly
- [ ] Integration tests: auth flow works on device
- [ ] All tests passing
- [ ] Coverage >90% (excluding generated code)

### Build
- [ ] Android: APK builds for all flavors
- [ ] iOS: builds for all flavors
- [ ] Makefile: all commands work
- [ ] CI pipeline configured

---

## Blockers & Issues

| # | Issue | Status | Resolution |
|---|-------|--------|------------|
| - | - | - | - |

---

## Session History

| Session | Date | Phases Worked | Summary |
|---------|------|---------------|---------|
| 1 | {date} | Phase 0 | Initial setup, tech stack discovery |
```

## Tracker Update Rules

1. **Before starting any step**: Read `.build-tracker.md` to know current state
2. **After completing a step**: Update status from `[ ] pending` to `[x] done` and add notes
3. **When a decision is made**: Update the Decisions Log table immediately
4. **When blocked**: Add entry to Blockers & Issues table
5. **At start of new session**: Add entry to Session History, read all decisions and current phase
6. **Update "Last Updated" and "Current Phase"** at every tracker write

## Status Values

- `[ ] pending` — Not started
- `[~] in_progress` — Currently working on
- `[x] done` — Completed successfully
- `[!] blocked` — Blocked, see Blockers table
- `[-] skipped` — Intentionally skipped (note reason)

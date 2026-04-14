# Workflow: Develop Epic

Implement all features in an Epic with proper dependency ordering.

## Before Starting [LOW]

1. Run pre-flight check — see [pre-flight-check.md](pre-flight-check.md). All checks must pass.
2. Read [implementation-guidelines.md](implementation-guidelines.md) for Dart/Flutter rules.
3. Confirm the user provides or points to the Epic document (features list, user stories, acceptance criteria).

---

## Phase 1: Epic Analysis & Planning [MEDIUM]

**Step 1.1 — Read & Understand Epic [LOW]**

Read the Epic document. Extract:
- Epic goal / business objective
- List of all Features in the Epic
- User Stories per Feature (if available)
- Acceptance criteria per Story
- Dependencies between Features (which must come first)

**Step 1.2 — Dependency Graph [MEDIUM]**

Map Feature dependencies:

```
Feature A (no deps)        → implement first
Feature B (depends on A)   → implement second
Feature C (depends on A)   → can parallel with B
Feature D (depends on B+C) → implement last
```

Rules:
- Features with no dependencies go first
- Features depending on the same parent can be implemented in parallel (sequentially in practice, but no blocking)
- Present the dependency graph to the user for confirmation

**Step 1.3 — Determine Implementation Order [MEDIUM]**

From the dependency graph, produce a linear implementation order:

1. Shared domain entities (used by multiple features)
2. Independent features (no deps)
3. Dependent features (in topological order)
4. Integration points (cross-feature interactions)

Present the ordered list. Ask user to confirm or reorder.

**Step 1.4 — Create Epic Tracker [LOW]**

Create `EPIC_TRACKER.md` in the project root:

```markdown
# Epic Tracker: {Epic Name}

> **Created**: {date}
> **Last Updated**: {date + time}
> **Current Feature**: Feature 1

## Features

| # | Feature | Dependencies | Status | Notes |
|---|---------|-------------|--------|-------|
| 1 | {name} | none | [ ] pending | |
| 2 | {name} | F1 | [ ] pending | |
| 3 | {name} | F1 | [ ] pending | |
| 4 | {name} | F2, F3 | [ ] pending | |

## Implementation Log

| Feature | Step | Status | Notes |
|---------|------|--------|-------|
```

Update tracker: mark Phase 1 done.

---

## Phase 2: Technical Design [MEDIUM]

For EACH feature (in implementation order), before coding:

**Step 2.1 — Feature Decomposition [MEDIUM]**

Break the feature into implementation units following Clean Architecture:

1. **Domain**: Entities, Repository interfaces, Use Cases
2. **Data**: Models (DTOs), Data Sources (remote + local), Repository implementations
3. **Presentation**: BLoC/Provider, Screens, Widgets

**Step 2.2 — API Contract Review [LOW]**

For each feature that calls the backend:
- Identify required API endpoints
- Check if the backend API already exists (review API docs or OpenAPI spec)
- Document request/response format for each endpoint
- Create corresponding Dart models (DTOs)

If the API doesn't exist yet:
- Flag it as a blocker
- Ask user if they want to proceed with mock data or wait

**Step 2.3 — Shared Components Check [MEDIUM]**

Before implementing, check:
- Are there existing widgets/utilities that can be reused?
- Does this feature need new shared widgets that other features will also use?
- If yes → implement shared components first, in `core/widgets/` or `core/utils/`

**Step 2.4 — GATE: Present Technical Design [HIGH]**

Present the technical design to the user:
- Files to create / modify (list with paths)
- New dependencies needed (if any)
- Estimated complexity (simple / medium / complex per unit)
- Risks or assumptions

Wait for user confirmation before proceeding to implementation.

---

## Phase 3: Feature-by-Feature Implementation [LOW]

For EACH feature (in the confirmed order from Phase 1):

**Step 3.1 — Implement Domain Layer [LOW]**

1. Create/update entities in `features/{feature}/domain/entities/`
2. Define repository interface in `features/{feature}/domain/repositories/`
3. Create use cases in `features/{feature}/domain/usecases/`

Rules:
- Entities are pure Dart classes — no annotations, no serialization
- Repository interfaces return `Future<Either<Failure, T>>`
- Each use case has a single `call()` method

**Step 3.2 — Implement Data Layer [LOW]**

1. Create models (DTOs) in `features/{feature}/data/models/` — with `fromJson`/`toJson`
2. Create data sources in `features/{feature}/data/datasources/`
   - Remote: API calls via Dio
   - Local: cache via Drift/Hive/SharedPreferences (if needed)
3. Implement repository in `features/{feature}/data/repositories/`
   - Catch exceptions → return `Left(Failure)`
   - Map models to entities via `.toEntity()`

**Step 3.3 — Implement Presentation Layer [MEDIUM]**

1. Create BLoC/Provider in `features/{feature}/presentation/bloc/` or `providers/`
   - Define all events/states (sealed classes)
   - Handle every event → emit appropriate state
2. Create screens in `features/{feature}/presentation/screens/`
3. Create feature-specific widgets in `features/{feature}/presentation/widgets/`
4. Add routes to GoRouter config

**Step 3.4 — Register DI [LOW]**

Register new classes in the DI container:
- Data sources (as singletons)
- Repository implementation (as singleton, bound to interface)
- Use cases (as factory)
- BLoC/Provider (as factory)

Run `dart run build_runner build --delete-conflicting-outputs` if using Injectable.

**Step 3.5 — Write Tests [LOW]**

For each feature, write tests BEFORE or ALONGSIDE implementation:

| Layer | Test Type | Must Cover |
|-------|-----------|------------|
| Domain / Use Cases | Unit | All logic paths, failure cases |
| Data / Repository | Unit | Success + failure, exception → failure mapping |
| Data / Data Sources | Unit | API call correctness, cache operations |
| Presentation / BLoC | Unit | All state transitions |
| Presentation / Screens | Widget | Render, interaction, error/loading/empty states |

Run after each feature:
```bash
flutter test
flutter analyze
```

All must pass before moving to the next feature.

**Step 3.6 — Update Epic Tracker [LOW]**

Mark feature as `[x] done` in `EPIC_TRACKER.md`. Move to next feature.

---

## Phase 4: Integration & QA [LOW]

After ALL features are implemented:

**Step 4.1 — Cross-Feature Integration [MEDIUM]**

- Verify navigation between features works
- Verify shared state (e.g., user auth) is consistent across features
- Verify data flows correctly between features that depend on each other

**Step 4.2 — Full Test Suite [LOW]**

```bash
flutter test --coverage
flutter analyze
```

- All tests must pass
- Coverage must be >90% (excluding generated code)
- Zero analysis errors

**Step 4.3 — Integration Tests [LOW]**

Write integration tests for the key user flows spanning multiple features:

```dart
// integration_test/epic_{name}_test.dart
// Test the end-to-end flow across features
```

Run on device/emulator:
```bash
flutter test integration_test/
```

**Step 4.4 — GATE: Epic Completion Report [HIGH]**

Present to the user:
- All features implemented (checklist)
- Test coverage report
- Files created/modified (summary)
- Any known issues or tech debt
- Remaining work (if any)

Update `EPIC_TRACKER.md`: mark all features done, update final status.

---
name: flutter-development
description: Develops Flutter mobile apps with Clean Architecture (Presentation → Domain → Data), state management (BLoC/Riverpod), Dio API client, local storage, JWT auth (client-side), error handling (Either/Failure pattern), theming, i18n, navigation (GoRouter), flavors (dev/staging/prod), and testing (unit + widget + integration, >90% coverage). Multiple workflows — Build Codebase, Develop Epic, Change Request, Fix Bug, Design-to-Code. Use when creating a new Flutter app, implementing features, fixing bugs, handling change requests, or any Flutter mobile development work.
---

# Flutter Development

Develop production-ready Flutter mobile apps following Clean Architecture and proven patterns.

## Freedom Levels

- **[LOW]** — Follow exactly. Use exact templates/commands from references. No improvisation.
- **[MEDIUM]** — Preferred pattern exists, adapt syntax to project context.
- **[HIGH]** — Multiple approaches valid. Use judgment based on context.

---

## Step 0: Workflow Selection [LOW]

**This step is mandatory.** Before doing ANY work, confirm which workflow to follow.

Ask the user:

> Which workflow do you need?
>
> 1. **Build Codebase** — Scaffold a new Flutter project from scratch (Clean Architecture, auth, theming, testing, CI/CD)
> 2. **Develop Epic** — Implement all features in an Epic with dependency ordering
> 3. **Change Request** — Modify existing functionality based on a change request (with BA traceability)
> 4. **Fix Bug** — Diagnose and fix a reported bug
> 5. **Design-to-Code** — Convert design specs/mockups into Flutter screens

If the user's message already implies a workflow (e.g., "create a new Flutter app" → Build Codebase, "implement this epic" → Develop Epic), confirm rather than re-asking.

After confirmation, follow the corresponding workflow file:

| Workflow | Reference | When to use |
|----------|-----------|-------------|
| Build Codebase | [references/build-codebase-workflow.md](references/build-codebase-workflow.md) | New project, scaffold from scratch |
| Develop Epic | [references/develop-epic-workflow.md](references/develop-epic-workflow.md) | Implement multiple features in an epic |
| Change Request | [references/change-request-workflow.md](references/change-request-workflow.md) | Modify existing functionality |
| Fix Bug | [references/fix-bug-workflow.md](references/fix-bug-workflow.md) | Diagnose and fix bugs |
| Design-to-Code | [references/design-to-code-workflow.md](references/design-to-code-workflow.md) | Convert designs to Flutter screens |

---

## Pre-flight Check (All Workflows) [LOW]

Before starting ANY workflow (except Build Codebase Phase 0-2), run the pre-flight check.

See [references/pre-flight-check.md](references/pre-flight-check.md).

---

## Shared Reference Files

These references are used across multiple workflows. Read based on what you need:

**Architecture & Patterns:**
- [references/architecture-layers.md](references/architecture-layers.md) — Clean Architecture, DI, theming, i18n
- [references/implementation-guidelines.md](references/implementation-guidelines.md) — Dart/Flutter DO/DON'T, lint rules

**Core Infrastructure:**
- [references/navigation-routing.md](references/navigation-routing.md) — GoRouter, deep linking, auth redirects
- [references/error-handling.md](references/error-handling.md) — Failures, Either, UI error states
- [references/api-client.md](references/api-client.md) — Dio, interceptors, connectivity
- [references/local-storage.md](references/local-storage.md) — Secure storage, SharedPreferences, Drift, Hive
- [references/authentication.md](references/authentication.md) — Token storage, biometric, OAuth client

**Quality & Delivery:**
- [references/testing-strategy.md](references/testing-strategy.md) — Unit, widget, integration tests
- [references/build-deploy.md](references/build-deploy.md) — Flavors, signing, CI/CD, store deploy

**Discovery (Build Codebase only):**
- [references/tech-stack-dictionary.md](references/tech-stack-dictionary.md) — Flutter, Dart, packages, versions

**Tracker:**
- [references/tracker-template.md](references/tracker-template.md) — Build tracker template and update rules

---

## Workflow Overviews

### 1. Build Codebase

Scaffold a complete Flutter project from zero to production-ready. 10 phases:

| Phase | Name | Freedom |
|-------|------|---------|
| 0 | Tech Stack Discovery | HIGH |
| 1 | Project Assessment | HIGH |
| 2 | Project Scaffold | MIXED |
| 3 | Implement Core Layer | MIXED |
| 4 | Implement Auth Feature | LOW |
| 5 | Implement Base Screens | MEDIUM |
| 6 | Comprehensive Testing | MIXED |
| 7 | Build & Deploy Setup | LOW |
| 8 | Completion Report | HIGH |
| 9 | Final Verification | LOW |

Full details: [references/build-codebase-workflow.md](references/build-codebase-workflow.md)

### 2. Develop Epic

Implement all features in an Epic with proper dependency ordering. 4 phases:

| Phase | Name | Freedom |
|-------|------|---------|
| 1 | Epic Analysis & Planning | MEDIUM |
| 2 | Technical Design | MEDIUM |
| 3 | Feature-by-Feature Implementation | LOW |
| 4 | Integration & QA | LOW |

Full details: [references/develop-epic-workflow.md](references/develop-epic-workflow.md)

### 3. Change Request

Modify existing functionality based on a CR, with BA document traceability. 8 steps:

| Step | Name | Freedom |
|------|------|---------|
| 1 | Receive & Understand CR | HIGH |
| 2 | Trace BA Impact | LOW |
| 3 | Code Impact Analysis | MEDIUM |
| 4 | Impact Report & Confirmation | HIGH |
| 5 | Implementation Plan | MEDIUM |
| 6 | Implement Changes | LOW |
| 7 | Update Tests | LOW |
| 8 | Verification & QA | LOW |

Full details: [references/change-request-workflow.md](references/change-request-workflow.md)

### 4. Fix Bug

Diagnose and fix a reported bug with proper regression prevention. 6 steps:

| Step | Name | Freedom |
|------|------|---------|
| 1 | Reproduce & Understand | LOW |
| 2 | Diagnose Root Cause | MEDIUM |
| 3 | Write Failing Test | LOW |
| 4 | Implement Fix | MEDIUM |
| 5 | Regression Testing | LOW |
| 6 | Verification | LOW |

Full details: [references/fix-bug-workflow.md](references/fix-bug-workflow.md)

### 5. Design-to-Code

Convert design specs or mockups into Flutter screens. 6 steps:

| Step | Name | Freedom |
|------|------|---------|
| 1 | Analyze Design Specs | LOW |
| 2 | Map to Architecture | MEDIUM |
| 3 | Component Breakdown | MEDIUM |
| 4 | Implement Screens | LOW |
| 5 | Responsive & Adaptive | MEDIUM |
| 6 | Visual QA | LOW |

Full details: [references/design-to-code-workflow.md](references/design-to-code-workflow.md)

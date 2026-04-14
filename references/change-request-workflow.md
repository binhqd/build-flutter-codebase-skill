# Workflow: Change Request

Modify existing functionality based on a change request, with BA document traceability.

## Before Starting [LOW]

1. Run pre-flight check — see [pre-flight-check.md](pre-flight-check.md). All checks must pass.
2. Read [implementation-guidelines.md](implementation-guidelines.md) for Dart/Flutter rules.
3. Confirm the user provides the Change Request details (what to change, why, acceptance criteria).

---

## Step 1: Receive & Understand CR [HIGH]

**1.1 — Read the Change Request**

Extract from the CR:
- **What** is changing (feature, behavior, UI, data flow)
- **Why** it's changing (business reason, user feedback, bug, regulation)
- **Acceptance criteria** (what "done" looks like)
- **Scope boundaries** (what should NOT change)

**1.2 — Clarify Ambiguities**

If the CR is unclear, ask the user specific questions:
- Which screens/flows are affected?
- Is this additive (new behavior) or replacing existing behavior?
- Are there backward compatibility concerns?
- Does the API change? Or only the client-side?

---

## Step 2: Trace BA Impact [LOW]

**2.1 — Identify Affected Features**

From the CR, determine which `features/` directories are affected:
- Which domain entities change?
- Which use cases change?
- Which screens/widgets change?
- Which API calls change?

**2.2 — Trace Architecture Layers**

For each affected feature, trace the change through all layers:

```
CR affects "user profile editing"
  → Domain: User entity fields? Use case logic?
  → Data: API endpoint change? Model fields? Local cache?
  → Presentation: BLoC states/events? Screen layout? Validation?
  → Navigation: Route changes? New screens?
  → Shared: Core widgets affected? Theme changes?
```

**2.3 — Identify Ripple Effects**

Check if the change affects other features:
- Does another feature depend on the changed entity?
- Does another screen navigate to/from the changed screen?
- Does a shared widget need updating?
- Do existing tests need updating?

Document all affected files.

---

## Step 3: Code Impact Analysis [MEDIUM]

**3.1 — Read Affected Code**

Read every file identified in Step 2. Understand the current implementation before planning changes.

**3.2 — Assess Change Complexity**

For each affected file, classify:
- **Minor**: Field rename, text change, style adjustment
- **Moderate**: Logic change, new state, API parameter change
- **Major**: New layer, architecture change, breaking change

**3.3 — Identify Risks**

- Will this break existing tests?
- Will this break other features?
- Is there a migration needed (local DB schema change)?
- Is there a backward compatibility issue (API version)?

---

## Step 4: Impact Report & Confirmation [HIGH]

**GATE: Present Impact Report to user before proceeding.**

```markdown
## Change Request Impact Report

### Summary
{1-2 sentence description of the change}

### Affected Files
| File | Layer | Change Type | Description |
|------|-------|-------------|-------------|
| ... | Domain | Moderate | Add new field to entity |
| ... | Data | Minor | Update DTO fromJson |
| ... | Presentation | Moderate | New form field + validation |

### Ripple Effects
- {Other features/files indirectly affected}

### Risks
- {List any risks}

### Tests to Update
- {List existing tests that will need changes}
- {List new tests needed}

### Estimated Scope
- Files to modify: {N}
- New files: {N}
- Tests to update: {N}
- New tests: {N}
```

Wait for user confirmation. Adjust plan if user provides feedback.

---

## Step 5: Implementation Plan [MEDIUM]

**5.1 — Order Changes**

Plan the implementation order (same as feature development):
1. Domain layer first (entities, use cases)
2. Data layer second (models, data sources, repositories)
3. Presentation layer last (BLoC/Provider, screens, widgets)

**5.2 — Plan Test Updates**

For each changed file:
- Identify which existing test needs updating
- Plan new test cases for new behavior
- Ensure the old behavior is no longer tested if it's replaced

---

## Step 6: Implement Changes [LOW]

**6.1 — Implement Domain Changes**

- Update entities (add/modify/remove fields)
- Update repository interfaces (new methods, changed signatures)
- Update use cases (new logic, changed parameters)

**6.2 — Implement Data Changes**

- Update models/DTOs (new fields, changed JSON keys)
- Update data sources (new API calls, changed cache logic)
- Update repository implementations
- If local DB schema changes: create migration

**6.3 — Implement Presentation Changes**

- Update BLoC/Provider (new events, new states, changed transitions)
- Update screens (new widgets, changed layout, new form fields)
- Update navigation (if routes change)

**6.4 — Validate After Each Layer**

```bash
flutter analyze  # Must be zero errors after each layer
```

---

## Step 7: Update Tests [LOW]

**7.1 — Update Existing Tests**

- Fix tests broken by the change
- Remove tests for removed behavior
- Update assertions for changed behavior

**7.2 — Write New Tests**

- Unit tests for new/changed use cases and repositories
- Widget tests for changed screens
- Test edge cases specific to the CR

**7.3 — Run Full Suite**

```bash
flutter test --coverage
```

- ALL tests must pass (not just the changed ones)
- Coverage must remain >90%
- If coverage dropped: write additional tests

---

## Step 8: Verification & QA [LOW]

**8.1 — Verify Acceptance Criteria**

Walk through each acceptance criterion from Step 1:
- [ ] Criterion 1: {verify}
- [ ] Criterion 2: {verify}
- [ ] ...

**8.2 — Verify No Regressions**

- Run the app and manually test the changed flow
- Test adjacent features that might be affected
- Verify the change works on both platforms (if applicable)

**8.3 — GATE: Present Completion Summary**

```markdown
## Change Request Complete

### Changes Made
| File | Change |
|------|--------|
| ... | ... |

### Tests
- Updated: {N} tests
- New: {N} tests
- All passing: yes/no
- Coverage: {X}%

### Acceptance Criteria
- [x] Criterion 1
- [x] Criterion 2

### Notes
- {Any caveats or follow-up items}
```

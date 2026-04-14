# Workflow: Fix Bug

Diagnose and fix a reported bug with proper regression prevention.

## Before Starting [LOW]

1. Run pre-flight check — see [pre-flight-check.md](pre-flight-check.md). All checks must pass.
2. Confirm the user provides the bug report (expected vs actual behavior, steps to reproduce).

---

## Step 1: Reproduce & Understand [LOW]

**1.1 — Read the Bug Report**

Extract:
- **Expected behavior**: What should happen
- **Actual behavior**: What happens instead
- **Steps to reproduce**: Exact sequence of actions
- **Environment**: Device, OS version, app version, flavor
- **Frequency**: Always, intermittent, specific conditions
- **Severity**: Crash, incorrect data, visual glitch, performance

**1.2 — Reproduce the Bug**

Try to reproduce the bug:
- Follow the exact steps from the report
- If possible, run the app on the same device/OS version
- Note any error messages, stack traces, or logs

If cannot reproduce:
- Ask user for more details
- Check if it's environment-specific (flavor, platform, OS version)
- Check if it's data-specific (certain user, certain input)

**1.3 — Collect Evidence**

Gather:
- Stack trace (if crash)
- Console logs (filtered to relevant output)
- Network traffic (if API related)
- UI state at the time of the bug

---

## Step 2: Diagnose Root Cause [MEDIUM]

**2.1 — Locate the Code**

From the stack trace or bug description, identify:
- Which feature is affected (`features/{name}/`)
- Which layer the bug likely originates from:
  - **Presentation**: UI rendering, state management, user interaction
  - **Domain**: Business logic, validation, use case flow
  - **Data**: API parsing, local storage, network handling

**2.2 — Read the Code**

Read the relevant code files. Understand the current implementation.

**2.3 — Identify Root Cause**

Determine WHY the bug occurs:
- Logic error in a use case or BLoC?
- Incorrect data mapping (model ↔ entity)?
- Missing error handling (unhandled exception)?
- Race condition (async issue)?
- Null safety violation (null where non-null expected)?
- State management issue (wrong state emitted)?
- API contract mismatch (backend changed, frontend didn't)?
- Platform-specific issue (iOS vs Android behavior difference)?

**2.4 — Document Root Cause**

Write a brief root cause statement:

```
Root cause: The `UserBloc` emits `UserLoaded` state before the API response
is fully parsed, causing `UserModel.fromJson` to throw on the `roles` field
when it's null (backend returns null for newly registered users without roles).
```

---

## Step 3: Write Failing Test [LOW]

**BEFORE fixing the bug**, write a test that reproduces it:

```dart
test('should handle null roles in user response', () async {
  // Arrange: mock API response with null roles
  when(() => mockRemote.getCurrentUser())
      .thenAnswer((_) async => UserModel.fromJson({
        'id': '1',
        'email': 'test@example.com',
        'roles': null,  // This is what the backend returns
      }));

  // Act
  final result = await repository.getCurrentUser();

  // Assert: should not crash, should return user with empty roles
  expect(result.isRight(), true);
  result.fold(
    (failure) => fail('Should not fail'),
    (user) => expect(user.roles, isEmpty),
  );
});
```

Run the test — it MUST FAIL (proving the bug exists):

```bash
flutter test test/features/{feature}/{test_file}.dart
```

If the test passes, your test doesn't reproduce the bug — revise it.

---

## Step 4: Implement Fix [MEDIUM]

**4.1 — Fix the Root Cause**

Apply the minimal fix that addresses the root cause:
- Fix the specific logic error
- Add the missing null check / error handling
- Correct the data mapping
- Fix the state transition

**Rules:**
- Fix only the bug — do NOT refactor surrounding code
- Fix only the root cause — do NOT add defensive code everywhere
- Keep the change as small as possible
- Follow existing code patterns and style

**4.2 — Verify the Failing Test Now Passes**

```bash
flutter test test/features/{feature}/{test_file}.dart
```

The test from Step 3 MUST now pass.

**4.3 — Run Static Analysis**

```bash
flutter analyze
```

Must be zero errors.

---

## Step 5: Regression Testing [LOW]

**5.1 — Run Full Test Suite**

```bash
flutter test --coverage
```

- ALL tests must pass (not just the new one)
- If other tests break: the fix has side effects — investigate
- Coverage must not decrease

**5.2 — Write Additional Edge Case Tests**

Based on the root cause, consider related edge cases:
- What if the field is empty (not null)?
- What if the API returns a different unexpected value?
- Does this same pattern exist in other models/features?

Write tests for any discovered edge cases.

**5.3 — Manual Smoke Test**

If possible, run the app and:
- Verify the bug is fixed (follow original reproduction steps)
- Verify the happy path still works
- Verify adjacent functionality is not broken

---

## Step 6: Verification [LOW]

**GATE: Present Fix Summary**

```markdown
## Bug Fix Summary

### Bug
{Brief description of the bug}

### Root Cause
{Root cause statement from Step 2.4}

### Fix
| File | Change |
|------|--------|
| ... | ... |

### Tests
- Failing test (proves bug): {test name}
- Edge case tests: {N} added
- Full suite: all passing
- Coverage: {X}%

### Verification
- [x] Bug no longer reproducible
- [x] Happy path works
- [x] No regressions
- [x] All tests pass
```

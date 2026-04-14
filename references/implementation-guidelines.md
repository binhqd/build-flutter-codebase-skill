# Implementation Guidelines — Dart / Flutter

## Contents

- Dart coding rules
- Flutter widget rules
- analysis_options.yaml
- Naming conventions
- File organization rules
- Performance guidelines

## Dart Coding Rules

### DO

- **DO** use `final` for all variables that are not reassigned
- **DO** use `const` constructors wherever possible — widgets, values, lists
- **DO** use named parameters for functions with >2 parameters
- **DO** use `sealed class` or `freezed` unions for state/event types
- **DO** use `Either<Failure, T>` return type for operations that can fail
- **DO** use `extension` methods to add functionality to existing types
- **DO** use `typedef` for complex function signatures
- **DO** use cascade notation (`..`) for sequential operations on the same object
- **DO** use collection-if and collection-for instead of imperative list building
- **DO** use `switch` expressions (Dart 3+) instead of `if-else` chains for enums
- **DO** return `Widget` type from build methods, not specific widget types
- **DO** make entity classes immutable (all fields `final`)
- **DO** use `Equatable` or `freezed` for value equality on states and entities

### DON'T

- **DON'T** use `dynamic` — always type explicitly. Use `Object?` if truly generic.
- **DON'T** use `print()` — use `Logger` or `debugPrint()` for dev, structured logging for prod
- **DON'T** catch `Exception` or `Object` broadly — catch specific exception types
- **DON'T** use `late` unless you can guarantee initialization before access
- **DON'T** use `!` (null assertion) without a preceding null check — prefer `?` or `??`
- **DON'T** use string interpolation for API paths — use `ApiEndpoints` constants
- **DON'T** import `dart:io` in shared code — it breaks web compatibility
- **DON'T** use `setState` for anything beyond simple local UI state (e.g., animation toggle)
- **DON'T** put business logic in widgets — extract to use cases or BLoC/providers
- **DON'T** use `GetX` — it violates separation of concerns and makes testing hard
- **DON'T** use `BuildContext` across async gaps — capture values before await
- **DON'T** create "god widgets" with >200 lines — extract sub-widgets
- **DON'T** hardcode strings, colors, or dimensions — use theme/constants/l10n
- **DON'T** use `MediaQuery.of(context)` repeatedly — cache the result
- **DON'T** use mutable state in entities or models — all fields must be `final`
- **DON'T** expose streams or notifiers directly from repositories — return `Future<Either<Failure, T>>`

## Flutter Widget Rules

### Stateless First

Default to `StatelessWidget`. Only use `StatefulWidget` when you need:
- `TextEditingController`, `ScrollController`, `AnimationController`
- `initState` / `dispose` lifecycle hooks
- Truly local, ephemeral UI state (e.g., expand/collapse toggle)

For everything else, use BLoC/Riverpod at the presentation layer.

### Widget Decomposition

```
// BAD — one giant build method
class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // 50 lines of header code
          // 80 lines of body code
          // 30 lines of footer code
        ],
      ),
    );
  }
}

// GOOD — decomposed into focused widgets
class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          ProfileHeader(user: user),
          ProfileBody(user: user),
          ProfileFooter(onLogout: _handleLogout),
        ],
      ),
    );
  }
}
```

### Keys

- **DO** use `ValueKey` or `ObjectKey` on list items in `ListView.builder`
- **DO** use `GlobalKey` only for `Form`, `Scaffold`, `Navigator` (sparingly)
- **DON'T** use `UniqueKey` unless you intentionally want to rebuild every frame

## analysis_options.yaml [LOW]

```yaml
include: package:flutter_lints/flutter.yaml
# Or for stricter rules:
# include: package:very_good_analysis/analysis_options.yaml

analyzer:
  errors:
    missing_return: error
    dead_code: warning
    unused_import: warning
    unused_local_variable: warning
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
    - "**/*.config.dart"
    - "**/*.mocks.dart"

linter:
  rules:
    # Error rules
    - always_use_package_imports
    - avoid_dynamic_calls
    - avoid_print
    - avoid_relative_lib_imports
    - avoid_returning_null_for_future
    - avoid_slow_async_io
    - avoid_type_to_string
    - cancel_subscriptions
    - close_sinks
    - literal_only_boolean_expressions
    - no_adjacent_strings_in_list
    - throw_in_finally
    - unnecessary_statements
    # Style rules
    - always_declare_return_types
    - annotate_overrides
    - avoid_bool_literals_in_conditional_expressions
    - avoid_catches_without_on_clauses
    - avoid_catching_errors
    - avoid_equals_and_hash_code_on_mutable_classes
    - avoid_escaping_inner_quotes
    - avoid_field_initializers_in_const_classes
    - avoid_final_parameters
    - avoid_multiple_declarations_per_line
    - avoid_positional_boolean_parameters
    - avoid_redundant_argument_values
    - avoid_returning_this
    - avoid_setters_without_getters
    - avoid_unused_constructor_parameters
    - avoid_void_async
    - cascade_invocations
    - cast_nullable_to_non_nullable
    - combinators_ordering
    - conditional_uri_does_not_exist
    - deprecated_consistency
    - directives_ordering
    - eol_at_end_of_file
    - join_return_with_assignment
    - leading_newlines_in_multiline_strings
    - missing_whitespace_between_adjacent_strings
    - no_literal_bool_comparisons
    - no_runtimeType_toString
    - noop_primitive_operations
    - omit_local_variable_types
    - one_member_abstracts
    - only_throw_errors
    - parameter_assignments
    - prefer_asserts_in_initializer_lists
    - prefer_constructors_over_static_methods
    - prefer_final_in_for_each
    - prefer_final_locals
    - prefer_if_elements_to_conditional_expressions
    - prefer_int_literals
    - prefer_mixin
    - prefer_null_aware_method_calls
    - prefer_single_quotes
    - require_trailing_commas
    - sort_constructors_first
    - sort_unnamed_constructors_first
    - type_annotate_public_apis
    - unawaited_futures
    - unnecessary_await_in_return
    - unnecessary_breaks
    - unnecessary_lambdas
    - unnecessary_null_aware_assignments
    - unnecessary_null_checks
    - unnecessary_parenthesis
    - unnecessary_raw_strings
    - unnecessary_to_list_in_spreads
    - unreachable_from_main
    - use_colored_box
    - use_decorated_box
    - use_enums
    - use_if_null_to_convert_nulls_to_bools
    - use_is_even_rather_than_modulo
    - use_named_constants
    - use_raw_strings
    - use_string_buffers
    - use_super_parameters
    - use_to_and_as_if_applicable
```

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Files | snake_case | `auth_repository.dart` |
| Classes | PascalCase | `AuthRepository` |
| Variables, functions | camelCase | `getUserById` |
| Constants | camelCase | `defaultTimeout` |
| Enums | PascalCase (type), camelCase (values) | `UserRole.admin` |
| Private | Prefix `_` | `_cachedUser` |
| BLoC Events | PascalCase, past tense | `LoginRequested` |
| BLoC States | PascalCase, adjective | `AuthAuthenticated` |
| Test files | `{source}_test.dart` | `auth_repository_test.dart` |
| Mock files | `{source}.mocks.dart` | `auth_repository.mocks.dart` |

## File Organization Rules

- One public class per file (private helpers are fine in the same file)
- File name matches the primary class name in snake_case
- Group imports: `dart:`, `package:`, relative — separated by blank lines
- Feature folders mirror the 3-layer structure: `data/`, `domain/`, `presentation/`
- Shared code goes in `core/`, NOT duplicated across features

## Performance Guidelines

- **DO** use `const` widgets — they are not rebuilt
- **DO** use `ListView.builder` (not `ListView`) for long lists — lazy rendering
- **DO** use `RepaintBoundary` to isolate frequently changing widgets
- **DO** cache expensive computations with `useMemoized` or BLoC state
- **DON'T** call `setState` in `build()` — causes infinite rebuilds
- **DON'T** create objects inside `build()` that could be const or cached
- **DON'T** use `Opacity` widget for hiding — use `Visibility` or conditional rendering
- **DON'T** nest `SingleChildScrollView` + `Column` + `ListView` — use `CustomScrollView` with slivers

# Workflow: Design-to-Code

Convert design specs or mockups into Flutter screens following the existing architecture.

## Before Starting [LOW]

1. Run pre-flight check — see [pre-flight-check.md](pre-flight-check.md). All checks must pass.
2. Read [implementation-guidelines.md](implementation-guidelines.md) for Dart/Flutter rules.
3. Confirm the user provides design specs (Figma link, screenshots, design document, or verbal description).

---

## Step 1: Analyze Design Specs [LOW]

**1.1 — Gather Design Materials**

Collect from the user:
- Screen mockups or screenshots (read image files if provided)
- Design tokens (colors, typography, spacing) — if specified
- Interaction specs (tap targets, transitions, animations)
- Responsive behavior (how it adapts to different screen sizes)
- State variations (loading, empty, error, data)

**1.2 — Inventory All Screens**

Create a list of all screens to implement:

| # | Screen | States | Priority | Dependencies |
|---|--------|--------|----------|-------------|
| 1 | Product List | loading, data, empty, error | high | none |
| 2 | Product Detail | loading, data, error | high | S1 |
| 3 | Cart | empty, with items | medium | S2 |

**1.3 — Identify Design Tokens**

Extract from the design:
- Colors (primary, secondary, surface, error, text variants)
- Typography (heading sizes, body sizes, weights)
- Spacing (padding, margin, gap values)
- Border radius values
- Shadow / elevation values
- Icon set (Material, custom)

Compare with existing `core/theme/` — reuse existing tokens where possible, extend only where needed.

---

## Step 2: Map to Architecture [MEDIUM]

**2.1 — Determine Feature Boundaries**

For each screen, identify:
- Which `features/` directory it belongs to (existing or new)
- What data it displays (which entities/models)
- What actions are available (which use cases)
- What state management is needed (BLoC events/states or providers)

**2.2 — Identify API Requirements**

For each screen:
- What data comes from the API? (list endpoints)
- What data comes from local storage?
- What user actions trigger API calls?

**2.3 — Check Existing Components**

Before creating new widgets, check:
- `core/widgets/` — shared reusable widgets
- `core/theme/` — existing design tokens
- Other `features/*/presentation/widgets/` — widgets that could be reused
- Material Design built-in widgets that match the design

List what exists vs what needs to be created.

---

## Step 3: Component Breakdown [MEDIUM]

**3.1 — Decompose Each Screen**

Break each screen into a widget tree:

```
ProductListScreen
├── AppBar (standard or custom)
├── SearchBar (if applicable)
├── ProductListBody
│   ├── LoadingView (shimmer placeholder)
│   ├── EmptyView (no results)
│   ├── ErrorView (retry button)
│   └── ProductGrid / ProductList
│       └── ProductCard
│           ├── ProductImage
│           ├── ProductTitle
│           ├── ProductPrice
│           └── AddToCartButton
└── FloatingActionButton (if applicable)
```

**3.2 — Classify Each Component**

| Component | Type | Location | Reusable? |
|-----------|------|----------|-----------|
| ProductCard | Feature widget | `features/product/presentation/widgets/` | Within feature |
| SearchBar | Shared widget | `core/widgets/` | Across features |
| ProductImage | Feature widget | `features/product/presentation/widgets/` | Within feature |
| LoadingView | Shared widget | `core/widgets/` (already exists) | Across features |

**3.3 — GATE: Present Component Plan**

Present the component breakdown to the user:
- Widget tree per screen
- New vs existing components
- Design token changes (if any)
- Estimated number of new files

Wait for confirmation.

---

## Step 4: Implement Screens [LOW]

Follow Clean Architecture layer order for any new data/domain code needed.

**4.1 — Update Theme (if needed) [MEDIUM]**

If the design requires new design tokens:
- Add new colors to `core/theme/app_colors.dart`
- Add new text styles to `core/theme/app_typography.dart`
- Add new spacing values to `core/theme/app_spacing.dart`
- Do NOT duplicate existing values — reuse or rename

**4.2 — Create Shared Widgets First [LOW]**

Build shared widgets (in `core/widgets/`) before feature-specific screens:
- These are used by multiple screens
- Build them with no business logic — pure UI, driven by parameters

**4.3 — Implement Domain + Data Layers (if needed) [LOW]**

If the screens require new data:
1. Create entities in `features/{feature}/domain/entities/`
2. Define use cases in `features/{feature}/domain/usecases/`
3. Create models + data sources in `features/{feature}/data/`
4. Implement repository

Skip this step if the screens only display existing data.

**4.4 — Implement BLoC / Provider [LOW]**

For each screen with dynamic data:
- Define events (user actions)
- Define states (loading, loaded, error, empty)
- Implement state transitions
- Register in DI

**4.5 — Implement Screens [MEDIUM]**

Build each screen following the widget tree from Step 3:

Rules:
- Use `const` constructors wherever possible
- Use theme values (not hardcoded colors/sizes)
- Handle ALL states (loading, data, empty, error)
- Use `Key` for list items
- Extract sub-widgets when `build()` exceeds ~50 lines
- Follow naming convention: `{Name}Screen` for pages, `{Name}Widget` or just `{Name}` for components

**4.6 — Add Routes [LOW]**

Register new screens in GoRouter:
- Add route definition
- Add route name constant
- Configure any path/query parameters

**4.7 — Validate [LOW]**

After each screen:
```bash
flutter analyze
flutter test
```

---

## Step 5: Responsive & Adaptive [MEDIUM]

**5.1 — Screen Size Handling**

- Use `LayoutBuilder` or `MediaQuery` for responsive layouts
- Consider phone vs tablet breakpoints
- Test on multiple screen sizes (if possible)

**5.2 — Platform Adaptation**

- Use Material Design widgets (they adapt automatically)
- Handle safe areas (notch, bottom bar) with `SafeArea`
- Handle keyboard insets with `SingleChildScrollView` or `resizeToAvoidBottomInset`

**5.3 — Accessibility**

- Ensure tap targets are at least 48x48
- Add `Semantics` labels for screen readers where meaning isn't obvious
- Ensure sufficient color contrast (use theme colors, which should already be WCAG compliant)

---

## Step 6: Visual QA [LOW]

**6.1 — Write Widget Tests**

For each new screen, write widget tests covering:
- Correct rendering of all elements
- All state variations (loading, data, empty, error)
- User interactions (tap, scroll, input)

**6.2 — Run Full Test Suite**

```bash
flutter test --coverage
flutter analyze
```

All must pass. Coverage must remain >90%.

**6.3 — Visual Comparison**

If design mockups are available:
- Run the app and compare side-by-side with the design
- Check: spacing, colors, typography, alignment, icon usage
- Note any intentional deviations (and why)

**6.4 — GATE: Present Completion Summary**

```markdown
## Design-to-Code Complete

### Screens Implemented
| Screen | States | Route |
|--------|--------|-------|
| ... | loading, data, empty, error | /product |

### Components Created
| Component | Type | Location |
|-----------|------|----------|
| ... | Shared | core/widgets/ |
| ... | Feature | features/{name}/presentation/widgets/ |

### Theme Changes
- {Any new colors, typography, spacing added}

### Tests
- Widget tests: {N} new
- All passing: yes
- Coverage: {X}%

### Design Fidelity Notes
- {Any intentional deviations from the design and why}
```

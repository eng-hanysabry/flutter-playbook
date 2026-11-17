# UI and widget lessons

Add verified lessons about screens, widgets, forms, responsive layout, themes, ScreenUtil, accessibility, keyboard/scroll behavior, rebuilds and shared components. Before adding, compare existing lessons by root cause and solution. Use `LESSON_TEMPLATE.md`.

---

## [UI-001] Never wrap a function tear-off inside a VoidCallback arrow closure without invoking it

- **Scope:** Shared (`benaaamen-m`)
- **Categories:** UI and widgets, Navigation, Event callbacks
- **Lifecycle:** User interaction, Event dispatch, Route navigation
- **Technical areas / data flow:** Widget callbacks (`VoidCallback` / `void Function()`), static/helper navigation handlers, Dart function tear-offs
- **Applies when:** Dart 3.x / Flutter 3.x whenever wiring a zero-argument method or static helper to a `VoidCallback?` (`void Function()?`) widget property
- **Symptom:** Tapping an interactive header/button widget plays the InkWell ripple animation, but silently performs no action or route transition and throws no error.
- **Reproduction:** Pass an arrow function returning a method tear-off without trailing parentheses (`onNotificationTap: () => NotificationNavigationHelper.openNotifications`) to a `VoidCallback?` property and trigger the tap handler.
- **Root cause:** In Dart, an arrow function `() => expr` typed as `void Function()` (`VoidCallback`) is allowed to evaluate any expression `expr` and discard its value. When `expr` is a function tear-off (`ClassName.staticMethod` without `()`), invoking the closure merely evaluates the function object reference and discards it rather than executing the function.
- **Rule:** Either pass a compatible zero-argument tear-off directly (`onTap: Helper.open`) or include invocation parentheses inside the closure (`onTap: () => Helper.open()`); never write `() => Helper.open`.
- **Bad pattern:**

```dart
AppHeader(
  title: 'Profile',
  // BUG: Returns the function tear-off to a void callback without calling it.
  onNotificationTap: () => NotificationNavigationHelper.openNotifications,
)
```

- **Good pattern:**

```dart
AppHeader(
  title: 'Profile',
  // Pass the tear-off directly (or call openNotifications() with parentheses).
  onNotificationTap: NotificationNavigationHelper.openNotifications,
)
```

- **Why this solution is preferred:** Passing the tear-off directly avoids allocating an unnecessary wrapper closure, eliminates the silent no-op bug, and makes the callback reference directly inspectable and testable.
- **Exceptions / caveats:** Use a wrapper closure `() => helper.open(arg)` only when passing arguments or deferring evaluation of a receiver that may change; in that case, always include the trailing invocation parentheses `(arg)`.
- **Regression scenario:**
  - **Given** a screen header wired to a single-flight navigation helper via `onNotificationTap`
  - **When** the notification callback is triggered
  - **Then** the navigation helper must execute and transition its navigation state rather than returning an uninvoked function reference
- **Automated test:** `test/features/shared/notifications/notifications_routing_and_design_test.dart` (`flutter test test/features/shared/notifications/notifications_routing_and_design_test.dart`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — `00:01 +6: All tests passed!`
- **Related lessons:** None

---

## [UI-002] Disable ambient scaffold background layers when using top App Bar gradients and unify header widgets across domain screens

- **Scope:** Shared (`benaaamen-m`)
- **Categories:** UI and widgets, AppScaffold, Responsive UI, Design tokens, Shared components
- **Lifecycle:** Initial-load, Screen build
- **Technical areas / data flow:** `AppScaffold`, `FourEllipsesLayer`, `MainAppHeader`, `PreferredSizeWidget`
- **Applies when:** Flutter 3.x / Dart 3.x with design system scaffolds supporting ambient body background layers (`FourEllipsesLayer` / background image) and pinned App Bar gradients
- **Symptom:** Unintended background image or blurred colored blobs bleed through the body of screens where the header gradient is already handled, and domain tab screens display mismatched header icon sizing, brand logo aspect ratios, or divider styles.
- **Reproduction:** Mount an `AppScaffold` with `showAppBarGradient: true` without explicitly disabling default ambient body layers (`showFourEllipses: false`), and create separate ad-hoc header widgets instead of reusing `MainAppHeader`.
- **Root cause:** `AppScaffold` defaulted `showFourEllipses` to `true`, causing blurred ellipse layers to render behind the entire body surface. Simultaneously, `ProviderHomeHeaderWidget` maintained custom dimensions (`140.w` logo width, custom icon buttons, plain divider) divergent from `MainAppHeader` (`48.w` logo width, design token divider, standardized touch targets).
- **Rule:** When an `AppScaffold` uses a top App Bar gradient, explicitly disable ambient body layers (`showFourEllipses: false, backgroundColor: context.appColors.white`), and implement domain screen headers by delegating to or rendering `MainAppHeader`.
- **Bad pattern:**

```dart
// Ad-hoc header with divergent logo sizing and scaffold body with default ambient layer
AppScaffold(
  showAppBarGradient: true,
  // showFourEllipses defaults to true, causing blurred color blobs in the body
  body: Column(
    children: [
      AdHocHeaderWidget(...), // Divergent icons, logo width 140, custom divider
      Expanded(child: ContentWidget()),
    ],
  ),
)
```

- **Good pattern:**

```dart
// Clean scaffold body with unified MainAppHeader as preferredSize appBar
AppScaffold(
  showAppBarGradient: true,
  showFourEllipses: false,
  backgroundColor: context.appColors.white,
  appBarGradientHeight: 116,
  appBar: MainAppHeader(
    title: title,
    onOpenDrawer: onOpenDrawer,
    onNotificationTap: onNotificationTap,
    hasNotificationBadge: hasUnread,
  ),
  body: ContentWidget(),
)
```

- **Why this solution is preferred:** Eliminates visual noise and GPU blur overhead in body scroll areas, establishes identical branding across all tabs, and ensures uniform touch targets and accessibility.
- **Exceptions / caveats:** Ambient layers (`showFourEllipses: true`) remain appropriate only for dedicated cards (e.g., user identity cards, wallet balance cards) or decorative marketing screens that explicitly request full-screen ambient blur.
- **Regression scenario:**
  - **Given** Provider Home and My Works screens with top App Bar gradients
  - **When** both screens render
  - **Then** the body background must contain no ambient ellipse or image layers, and both app bars must display identical icons, logo dimensions (`48.w x 36.h`), and gradient dividers
- **Automated test:** `test/features/provider/home/presentation/pages/home_page_test.dart` and `test/features/provider/portfolio/presentation/pages/portfolio_page_test.dart` (`flutter test test/features/provider/home/presentation/pages/home_page_test.dart test/features/provider/portfolio/presentation/pages/portfolio_page_test.dart`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — `00:02 +8: All tests passed!`
- **Related lessons:** `[UI-001]`

---

## [UI-003] Never filter out unavailable product variants at DTO parsing or presentation layers; display with disabled styling and guard selection

- **Scope:** Shared (`panda-m`)
- **Categories:** UI and widgets, Model serialization, Product variants, Design tokens, Disabled state
- **Lifecycle:** DTO deserialization, Widget build, User interaction
- **Technical areas / data flow:** `ProductVariantModel.fromJson`, `VariantsWidget`, `_VariantOptionRow`, `VariantController`
- **Applies when:** Flutter 3.x / Dart 3.x with product variant models having option availability flags (`is_available`)
- **Symptom:** Unavailable options (e.g. sizes or meal volumes out of stock) vanish completely from the UI instead of rendering in a disabled, unselectable state with disabled typography tokens.
- **Reproduction:** Parse an API response containing variant options with `"is_available": false` when `ProductVariantModel.fromJson` or UI widget build methods filter options via `.where((o) => o.isAvailable)`.
- **Root cause:** Pre-filtering options at the DTO deserialization level (`.where((o) => o.isAvailable)` in `fromJson`) or at UI list rendering stripped unavailable items before the presentation layer could inspect or render them in an inactive state.
- **Rule:** Retain all options during model deserialization and pass them to UI widgets. In the UI, render unavailable options with disabled text styling (`context.appColors.gray.c40`), set interaction handlers (`InkWell.onTap`, `Radio.onChanged`) to `null`, and constrain auto-default selection to available options only.
- **Bad pattern:**

```dart
// DTO parsing stripping unavailable items prematurely
factory ProductVariantModel.fromJson(Map<String, dynamic> json) {
  return ProductVariantModel(
    name: json['name'] ?? '',
    isColor: _toBool(json['is_color']),
    options: (json['options'] as List?)
        ?.map((e) => VariantOptionModel.fromJson(e))
        .where((o) => o.isAvailable) // BUG: strips unavailable options from model
        .toList() ?? [],
  );
}

// UI hiding unavailable options
final availableOptions = group.options.where((o) => o.isAvailable).toList();
```

- **Good pattern:**

```dart
// DTO parses all options, preserving isAvailable
options: (json['options'] as List?)
    ?.map((e) => VariantOptionModel.fromJson(e))
    .toList() ?? [],

// UI renders all options, styling and disabling unavailable ones
final isAvailable = option.isAvailable;
final disabledColor = context.appColors.gray.c40;

InkWell(
  onTap: isAvailable ? onTap : null,
  child: Row(
    children: [
      Radio<String>(
        value: option.id.toString(),
        groupValue: groupValue,
        onChanged: isAvailable ? (_) => onTap() : null,
      ),
      Expanded(
        child: Text(
          option.name,
          style: baseStyle?.copyWith(
            color: isAvailable ? null : disabledColor,
          ),
        ),
      ),
    ],
  ),
);
```

- **Why this solution is preferred:** Users can see all offered choices while immediately recognizing out-of-stock options through disabled styling, and state controllers prevent invalid selection.
- **Exceptions / caveats:** If a product category or business requirement strictly dictates hiding deprecated or deleted variants completely, ensure the backend suppresses them rather than client-side availability flags confusing stock availability with catalog visibility.
- **Regression scenario:**
  - **Given** a dish or product with 1 available option ("كبير") and 2 unavailable options ("وسط", "صغير")
  - **When** the variant section renders
  - **Then** all 3 options must be present in the UI, unavailable options must have `onTap: null` and `onChanged: null` with disabled color styling, and default selection must only pick "كبير"
- **Automated test:** `packages/services/resturant_home/test/restaurant_dish_variant_availability_test.dart` (`flutter test packages/services/resturant_home/test/restaurant_dish_variant_availability_test.dart`)
- **Verification:** Verified on Flutter 3.32+ / Dart 3.8+ — `00:01 +4: All tests passed!`
- **Related lessons:** None

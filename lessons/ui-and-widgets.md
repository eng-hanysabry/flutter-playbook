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

## [UI-003] Never combine tight explicit vertical padding with fixed height on buttons displaying localized text with descenders

- **Scope:** Shared (`benaaamen-m`)
- **Categories:** UI and widgets, Responsive UI, Localization, Typography
- **Lifecycle:** User interaction, Screen build, Localization
- **Technical areas / data flow:** Button widgets (`Container`, `AppButton`, `CustomText`), font metrics (`Noto Sans Arabic`), vertical alignment and clipping
- **Applies when:** Flutter 3.x / Dart 3.x with fixed-height button containers (`height: 46.h` / `48.h`) displaying non-Latin or Arabic typography with glyphs extending below baseline (`تسجيل الخروج`)
- **Symptom:** The bottom portion of Arabic letters with bottom descenders (such as `ج` in `الخروج` or `ر`) is visibly clipped off at the bottom border of the button.
- **Reproduction:** Set a fixed button container height (`height: 46.h`), add explicit vertical padding (`padding: EdgeInsets.symmetric(vertical: 12.h, horizontal: 20.w)`), and render Arabic text with rigid line height (`height: 20 / 14`). The available inner height becomes `46 - 24 = 22.h`, clipping bottom glyph descenders.
- **Root cause:** Combining a rigid container `height` with explicit `vertical` padding severely constrains the inner vertical layout space (`inner_height = height - (2 * vertical_padding)`). Because Arabic font metrics in `Noto Sans Arabic` allocate additional descent space for letters like `ج`, `ح`, `خ`, `ع`, `غ`, `ر`, `ز`, the text overflows the inner bounds and clips at the container edge.
- **Rule:** On fixed-height action buttons, omit explicit vertical padding (or use `EdgeInsets.symmetric(horizontal: ...)`) and center the text with `alignment: Alignment.center` within an adequate height (`48.h`), allowing the text widget to use the full container vertical span without descender clipping.
- **Bad pattern:**

```dart
Container(
  width: double.infinity,
  height: 46.h,
  // BUG: 12.h * 2 leaves only 22.h for 14.sp Arabic text with descenders, clipping the bottom.
  padding: EdgeInsets.symmetric(vertical: 12.h, horizontal: 20.w),
  alignment: Alignment.center,
  child: CustomText(
    'profile.logout'.tr(), // "تسجيل الخروج"
    fontSize: 14.sp,
    height: 20 / 14,
  ),
)
```

- **Good pattern:**

```dart
Container(
  width: double.infinity,
  height: 48.h,
  // Omit vertical padding; let alignment: Alignment.center use the full 48.h span.
  padding: EdgeInsets.symmetric(horizontal: 20.w),
  alignment: Alignment.center,
  child: CustomText(
    'profile.logout'.tr(),
    fontSize: 14.sp,
    fontWeight: FontWeight.w700,
  ),
)
```

- **Why this solution is preferred:** Centers text cleanly without clipping descenders or diacritics across varying localized strings and ScreenUtil scaling factors.
- **Exceptions / caveats:** When a button's height is determined dynamically by its content (no fixed `height:`), symmetric vertical padding is appropriate and necessary.
- **Regression scenario:**
  - **Given** Client Profile and Provider Profile screens with dangerous action logout buttons
  - **When** the page renders in Arabic
  - **Then** the text "تسجيل الخروج" must render with full descenders visible and zero bottom border clipping
- **Automated test:** `test/features/provider/profile/presentation/pages/profile_details_page_test.dart` (`flutter test test/features/provider/profile/presentation/pages/profile_details_page_test.dart`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — all profile tests passed cleanly.
- **Related lessons:** `[UI-002]`

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

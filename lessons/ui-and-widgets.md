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

---

## [UI-004] Render binary image API responses in WebView using in-memory Base64 HTML strings instead of direct loadRequest URLs

- **Scope:** Shared (`benaaamen-m`)
- **Categories:** UI and widgets, Web views, Media, REST/Dio
- **Lifecycle:** User interaction, Initial-load, Async fetch
- **Technical areas / data flow:** `webview_flutter` (`WebViewController`), binary response streams, Base64 data URLs, HTML rendering
- **Applies when:** Flutter 3.x / Dart 3.x using `webview_flutter` to preview images or files retrieved from authenticated REST endpoints returning binary byte streams (`Content-Type: image/jpeg`, `application/octet-stream`)
- **Symptom:** Opening a document preview dialog displays an indefinite loading spinner while the console logs raw response bytes (`147, 92, 22, 88...`), and the WebView never displays the image or invokes `onPageFinished`.
- **Reproduction:** Call `WebViewController.loadRequest(uri, headers: ...)` directly with an API endpoint that returns a binary stream (or attachment headers). Android WebView treats the response as an unhandled download rather than a renderable HTML document, hanging navigation indefinitely.
- **Root cause:** Android's internal WebView engine does not automatically render arbitrary raw binary byte streams received from HTTP GET requests without a standard HTML wrapper or native mime handling. Because navigation never completes as a web page, `onPageFinished` is never triggered and the UI loading indicator spins forever.
- **Rule:** Retrieve the binary bytes via clean architecture (BLoC / use case with authenticated client), convert the byte buffer to a Base64 data URL embedded in an HTML `<img>` template with responsive CSS and pinch-to-zoom viewport meta tags, and render it using `WebViewController.loadHtmlString(...)`.
- **Bad pattern:**

```dart
// BUG: Android WebView cannot render raw binary octet streams directly via loadRequest; hangs forever
_controller.loadRequest(
  Uri.parse('https://api.example.com/documents/69'),
  headers: {'Authorization': 'Bearer $token'},
);
```

- **Good pattern:**

```dart
// 1. Fetch bytes via clean architecture (BLoC / Use Case)
final bytes = await file.readAsBytes();
final mimeType = isPng ? 'image/png' : 'image/jpeg';
final base64String = base64Encode(bytes);

// 2. Embed into responsive HTML with pinch-to-zoom and render via loadHtmlString
final html = '<!DOCTYPE html><html><head><meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=3.0, user-scalable=yes"><style>html,body{margin:0;padding:12px;display:flex;justify-content:center;align-items:center;min-height:100vh;background-color:#f7f9fa;box-sizing:border-box;}img{max-width:100%;max-height:100%;object-fit:contain;border-radius:8px;box-shadow:0 2px 8px rgba(0,0,0,0.1);}</style></head><body><img src="data:$mimeType;base64,$base64String" alt="Preview"/></body></html>';
await _controller.loadHtmlString(html);
_isLoadingNotifier.value = false;
```

- **Why this solution is preferred:** Guarantees instantaneous, 100% reliable rendering on both Android and iOS WebView, provides pinch-to-zoom accessibility, prevents download dialogs or external app opening when the user merely wants a quick preview, and eliminates infinite loading states.
- **Exceptions / caveats:** For multi-page PDF documents on Android (where WebView does not natively render PDFs without PDF.js or Google Docs viewer), pass the file to a dedicated PDF viewer widget or fallback to `loadFile` / platform view.
- **Regression scenario:**
  - **Given** a provider document returned from an API endpoint as raw binary image bytes
  - **When** the user taps the "معاينة" (Preview) action on the document tile
  - **Then** the image must render immediately inside the preview dialog WebView with zero external app opening, no download notification, and the loading indicator dismissed promptly
- **Automated test:** `test/features/provider/profile/presentation/bloc/profile_details_bloc_test.dart` and `test/features/provider/profile/presentation/pages/profile_details_page_test.dart` (`flutter test test/features/provider/profile/`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — all 19 profile tests passed cleanly.
- **Related lessons:** `[UI-003]`

---

## [UI-005] Sort and format localized working day schedules with unclipped flex banner rows

- **Scope:** Shared (`benaaamen-m`)
- **Categories:** UI and widgets, Localization, Responsive UI, Typography
- **Lifecycle:** Initial-load, Screen build, Localization
- **Technical areas / data flow:** `ProviderWorkingScheduleCard`, day normalization, canonical week ordering (Saturday to Friday), consecutive range collapsing, flex rows
- **Applies when:** Flutter 3.x / Dart 3.x displaying localized working schedule banners containing both variable-length day lists and formatted time period strings
- **Symptom:** In summary schedule banners, hours text is truncated with ellipsis (`08:00 صباحًا — 05...`), and day lists appear unsorted or separated by plain dashes instead of canonical ranges or locale-appropriate commas (`، ` in Arabic, `, ` in English).
- **Reproduction:** Wrap both the days label and the hours label in `Flexible(child: CustomText(..., overflow: TextOverflow.ellipsis))` inside a `Row`. When the days string grows (multiple days joined with commas), it shrinks the hours label until its descenders or trailing period are clipped.
- **Root cause:** Simultaneous use of `Flexible` with `TextOverflow.ellipsis` on both sides of a horizontal `Row` causes competing flex space allocation. Without sorting days by standard calendar order (Saturday = 0 through Friday = 6), arbitrary incoming API array orders produce chaotic day displays.
- **Rule:** Order incoming days strictly by canonical week order (Saturday to Friday). If the sorted days form an unbroken consecutive sequence, collapse them into a range (`startDay - endDay`); otherwise join them with locale-specific commas (`، ` in Arabic, `, ` in English). In the summary row, allocate remaining width to days via `Expanded` while allowing the formatted time period to size to its natural width without clipping.
- **Bad pattern:**

```dart
// BUG: Arbitrary day order, plain dashes for non-continuous days, and hours truncated with ellipsis
final daysStr = rawDays.join(' — ');
Row(
  children: [
    Flexible(child: CustomText(daysStr, overflow: TextOverflow.ellipsis)),
    Flexible(child: CustomText(hoursStr, overflow: TextOverflow.ellipsis)),
  ],
)
```

- **Good pattern:**

```dart
// 1. Sort by canonical week order (Saturday to Friday)
final sortedOrders = ProviderScheduleFormatter.getSortedDayOrders(workingDays);

// 2. Collapse continuous sequences into range (Sat - Mon) or join with localized comma
final daysStr = ProviderScheduleFormatter.formatGroupedDays(sortedOrders, languageCode: locale.languageCode);

// 3. Format hours without clipping
Row(
  children: [
    Expanded(child: CustomText(daysStr, fontSize: 13.sp, fontWeight: FontWeight.w700)),
    SizedBox(width: 8.w),
    CustomText(hoursStr, fontSize: 13.sp, fontWeight: FontWeight.w700, textAlign: TextAlign.end),
  ],
)
```

- **Why this solution is preferred:** Guarantees predictable, professional RTL/LTR day order, produces readable day ranges (e.g. "السبت - الإثنين") or comma lists ("السبت، الإثنين، الأربعاء"), and ensures time periods are never truncated.
- **Exceptions / caveats:** In individual child rows where width is constrained, use `Expanded` on the day label and natural sizing on the trailing clock icon/time row.
- **Regression scenario:**
  - **Given** working schedule data with continuous and non-continuous days
  - **When** the profile schedule card renders in Arabic or English
  - **Then** consecutive days must collapse into a dash range ("السبت - الإثنين"), non-consecutive days must display localized commas ("السبت، الإثنين، الأربعاء"), days must be ordered Saturday to Friday, and hours must display full unclipped text
- **Automated test:** `test/features/provider/profile/presentation/widgets/provider_schedule_formatter_test.dart` and `test/features/provider/profile/presentation/pages/profile_details_page_test.dart` (`flutter test test/features/provider/profile/`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — all 31 tests passed cleanly.
- **Related lessons:** `[UI-003]`, `[UI-004]`

---

## [UI-006] Preserve persistent document upload slots on deletion during edit flows with register-matching validation

- **Scope:** Shared (`benaaamen-m`)
- **Categories:** UI and widgets, Forms, Profile/Settings, Validation UX, Shared components
- **Lifecycle:** Update-save, Delete, Submit, Screen build
- **Technical areas / data flow:** Document management, slot-based UI state (`ProfileEditDocumentSlot`), multipart uploads, client-side validation toasts
- **Applies when:** Flutter 3.x / Dart 3.x applications with edit flows for professional profiles or documents where mandatory verification attachments can be replaced
- **Symptom:** In profile edit forms, deleting an existing attachment removes the entire widget tile from the tree, shrinking or hiding the card and leaving the user with no UI placeholder to upload a replacement document, while allowing form submission without mandatory documents.
- **Reproduction:** Call `.removeWhere((doc) => doc.id == id)` on the documents list when the user taps delete, causing the document card to vanish, and tap save without enforcing that every required document category is satisfied.
- **Root cause:** Treating editable document attachments as an ephemeral list of existing items rather than a fixed set of required category slots (`certificate`, `health_certificate`, `commercial_front`, `commercial_back`, `activity_license`). When an existing document was deleted, the slot was erased from the UI rather than transitioning to an empty upload placeholder.
- **Rule:** Model documents in edit flows as category-bound slots (`ProfileEditDocumentSlot`) tracking original document, deletion flag, and replacement file. When an attachment is deleted, retain its position in the UI and render the empty upload placeholder card (with dashed border, category title, and file picker) matching the registration step. Block form save and alert the user with localized error toasts if any required document slot is missing an attachment.
- **Bad pattern:**
```dart
// BUG: Removing the document erases the slot from the UI and allows saving without required doc
void _onRemoveDocument(String docId) {
  _documentsNotifier.value = List.from(_documentsNotifier.value)
    ..removeWhere((doc) => doc.id == docId);
}
```
- **Good pattern:**
```dart
// Slot remains in place as replacement upload placeholder
void _onRemoveDocument(String slotId) {
  _slotsNotifier.value = _slotsNotifier.value
      .map((s) => s.id == slotId ? s.copyWith(isDeleted: true, clearNewFile: true) : s)
      .toList();
}

// Validation enforces all required slots are satisfied before submit
final missingSlot = ProfileEditDocumentHelper.findMissingDocumentSlot(_slotsNotifier.value);
if (missingSlot != null) {
  context.showErrorAlert(ProfileEditDocumentHelper.getMissingDocumentMessage(missingSlot));
  return;
}
```
- **Why this solution is preferred:** Maintains a clear, persistent visual affordance for the user to upload replacement documents without navigating away, guarantees that required verification documents are never accidentally bypassed on profile update, and mirrors the exact UX and error messages of the registration wizard.
- **Exceptions / caveats:** Optional document types (if configured by backend metadata) should allow empty states; required document types must strictly block form submission when empty.
- **Regression scenario:**
  - **Given** a provider profile edit screen with an existing uploaded certificate
  - **When** the user taps the delete icon on the certificate tile
  - **Then** the card must not disappear; it must transition to the empty upload card placeholder, and tapping save must show an error alert ("يرجى رفع الشهادة المهنية") until a replacement file is picked
- **Automated test:** `test/features/provider/profile/presentation/pages/profile_edit_page_test.dart` (`flutter test test/features/provider/profile/presentation/pages/profile_edit_page_test.dart`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — all 42 tests passed cleanly.
- **Related lessons:** `[UI-003]`, `[UI-005]`


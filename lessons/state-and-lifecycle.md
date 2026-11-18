# State and lifecycle lessons

Add verified lessons about BLoC/Cubit/Riverpod/Provider, async state, controllers, streams, subscriptions, disposal and lifecycle races. Before adding, compare existing lessons by root cause and solution. Use `LESSON_TEMPLATE.md`.

---

## [STATE-001] Separate UI optimistic counter notifiers from server-confirmed BLoC quantities and preserve explicit zero in copyWith

- **Scope:** Shared (`panda-m`)
- **Categories:** architecture/state, API/data, lists/pagination, detail views, shared components
- **Lifecycle:** update-save, delete, concurrent-race
- **Technical areas / data flow:** BLoC (`flutter_bloc`), `Equatable` state `props`, `ValueNotifier<int>` optimistic UI counters, `copyWith` nullable integer fields, additive/delta cart endpoints
- **Applies when:** Flutter 3.x / Dart 3.x with `flutter_bloc` and `Equatable` managing item/dish quantities backed by additive (`POST /carts`) and relative delta (`update-qty` / `DELETE`) REST endpoints
- **Symptom:**
  1. Rapidly tapping `+` then immediately tapping `-` while the add request is in flight causes the UI counter to jump back to the older add request's quantity when the response returns, leaving the UI counter and server cart out of sync.
  2. Decrementing/deleting an item to `0` fails to clear `item.qty` in state, causing subsequent operations to misclassify additions as decrements (`404 Resource not found` on deleted `cartItemId`) or loop infinitely in convergence loops.
  3. Adding items with variants from a bottom sheet (`targetQty == null`) overwrites the card badge with only the last added quantity instead of the accumulated total.
- **Reproduction:**
  1. Increment an item from `3` to `6`, wait for `POST /carts {qty: 3}` to start, then immediately tap `-` (`6 -> 5`) before the HTTP response returns.
  2. Delete an item (`targetQty: 0`) when `_updateLocalDishQty` passes `qty: qty > 0 ? qty : null` into `copyWith({int? qty}) => Model(qty: qty ?? this.qty)`.
- **Root cause:**
  1. Unconditionally setting `dish.quantityNotifier.value = qty` inside the BLoC's network-success handler overwrote the user's newer local tap (`5`) with the older completed request's target (`6`).
  2. Passing `null` for `qty` when `qty == 0` into `copyWith` (`qty: qty ?? this.qty`) preserved the previous non-zero `this.qty` instead of resetting `dish.qty` to `0`.
  3. Computing `final targetQty = event.targetQty ?? event.qty` on variant additions ignored the existing in-cart quantity `originalQty`.
  4. Omitting `restaurantDishes` from `RestaurantState.props` caused `Equatable` to drop state emissions when only dish quantities changed.
- **Rule:** Never overwrite a UI-owned `ValueNotifier` counter on network success (only on error rollback), pass explicit `0` (never `null`) to `copyWith` when resetting quantities, accumulate `originalQty + event.qty` when `targetQty` is omitted, and include all mutable collection fields in `Equatable.props`.
- **Bad pattern:**

```dart
// 1. Passing null when qty is 0 keeps the old this.qty in copyWith(qty: qty ?? this.qty)!
// 2. Unconditionally overwriting quantityNotifier.value on network success clobbers newer taps!
void _updateLocalDishQty(String itemId, int qty, Emitter<RestaurantState> emit) {
  final updated = state.restaurantDishes?.map((dish) {
    if (dish.id == itemId) {
      dish.quantityNotifier.value = qty; // BUG: overwrites newer in-flight user taps!
      return dish.copyWith(qty: qty > 0 ? qty : null); // BUG: null ?? this.qty keeps old qty!
    }
    return dish;
  }).toList();
  emit(state.copyWith(restaurantDishes: updated));
}
```

- **Good pattern:**

```dart
void _updateLocalDishQty(
  String itemId,
  int qty, {
  String? cartItemId,
  bool clearCartItemId = false,
  bool syncNotifier = true,
  required Emitter<RestaurantState> emit,
}) {
  final updated = state.restaurantDishes?.map((dish) {
    if (dish.id == itemId) {
      if (syncNotifier) {
        dish.quantityNotifier.value = qty; // Only true on error rollback or external sheet add
      }
      return dish.copyWith(
        qty: qty, // Pass explicit 0 so qty resets cleanly
        cartItemId: cartItemId,
        clearCartItemId: clearCartItemId,
      );
    }
    return dish;
  }).toList();
  emit(state.copyWith(restaurantDishes: updated));
}
```

- **Why this solution is preferred:** Decouples instantaneous UI responsiveness (`ValueNotifier`) from serialized server-state convergence (`dish.qty` in BLoC), prevents stale network responses from reverting user input, guarantees termination of convergence loops on delete, and keeps variant badge totals accurate.
- **Exceptions / caveats:** When an addition originates from an external modal (such as a variant quick-add bottom sheet) rather than the inline counter itself, pass `syncNotifier: true` with `targetQty = originalQty + event.qty` so the card badge reflects the newly accumulated total. In cart and catalog grids (`OrdersCartItemTile`, `ClothesProductListItem`, `ProductCardWidget`), propagate `maxValue: item.maxStock ?? product.stock` to counter widgets to disable `+` when reaching maximum stock. For recommended or catalog products requiring variants (`hasVariants == true`), intercept inline `addToCart` to navigate to the product details page instead of dispatching direct cart additions without `combination_id` (preventing 422 Unprocessable Entity errors). Omit manual `showErrorAlert` calls in page BLoC listeners when the core networking interceptor already surfaces API error alerts.
- **Regression scenario:**
  - **Given** an in-cart item or recommended product with limited stock, and a recommended multi-variant product
  - **When** the item quantity equals stock, or a user taps add on the recommended multi-variant product
  - **Then** the increment `+` button is disabled when quantity reaches stock, and tapping add on the variant product navigates to the details page rather than failing with a 422 HTTP error
- **Automated test:**
  - `packages/services/resturant_home/test/resturant_home_test.dart` (`flutter test packages/services/resturant_home/test/resturant_home_test.dart`)
  - `packages/services/cart/test/cart_quantity_test.dart` (`flutter test packages/services/cart/test/cart_quantity_test.dart`)
  - `packages/services/clothes/test/clothes_stock_and_cart_test.dart` (`flutter test packages/services/clothes/test/clothes_stock_and_cart_test.dart`)
  - `packages/services/cart/test/cart_stock_and_variant_test.dart` (`flutter test packages/services/cart/test/cart_stock_and_variant_test.dart`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — all unit and widget tests in `resturant_home_test.dart`, `cart_quantity_test.dart`, `clothes_stock_and_cart_test.dart`, and `cart_stock_and_variant_test.dart` passed
- **Related lessons:** None

---

## [STATE-002] Child Modular feature modules must resolve root-level dependencies via Modular.get instead of module-scoped injector i.get

- **Scope:** Shared (`benaaamen-m`)
- **Categories:** architecture/di, state management, module wiring, error handling
- **Lifecycle:** initialization, navigation, route-building
- **Technical areas / data flow:** Flutter Modular (`flutter_modular`), `AutoInjector`, module scoping, root services (`NetworkingService`, `AppDatabase`)
- **Applies when:** Flutter Modular 5.x / 6.x multi-module applications where global services are declared in root `AppModule` and accessed by child feature modules (`features/*/*_module.dart`)
- **Symptom:** Navigating to a child module route or instantiating a BLoC/Cubit throws `BindNotFoundException: UnregisteredInstance: <Service> unregistered` during widget build or route resolution (e.g. `UnregisteredInstance: NetworkingService unregistered`).
- **Reproduction:** In a child feature module's `binds(Injector i)` method, bind a data source using `i.get<NetworkingService>()` instead of `Modular.get<NetworkingService>()`, then navigate to the child module route.
- **Root cause:** The child module injector `i` is an isolated `AutoInjector` instance that only searches binds defined inside that specific child module and its explicitly imported modules. It cannot resolve global services registered in the root `AppModule`. In contrast, `Modular.get<T>()` traverses the root Modular container tree.
- **Rule:** Inside child feature modules, always resolve root-level singleton services (such as `NetworkingService`) via `Modular.get<T>()` rather than `i.get<T>()`, and enforce this statically with a linter rule and root-level route integration tests.
- **Bad pattern:**

```dart
class ProfileModule extends Module {
  @override
  void binds(Injector i) {
    i.addLazySingleton<EquipmentOwnerActivationRemoteDataSource>(
      () => EquipmentOwnerActivationRemoteDataSourceImpl(
        networkingService: i.get<NetworkingService>(), // CRASH: i cannot see root AppModule binds!
      ),
    );
  }
}
```

- **Good pattern:**

```dart
class ProfileModule extends Module {
  @override
  void binds(Injector i) {
    i.addLazySingleton<EquipmentOwnerActivationRemoteDataSource>(
      () => EquipmentOwnerActivationRemoteDataSourceImpl(
        networkingService: Modular.get<NetworkingService>(), // SAFE: Modular.get delegates to root AppModule container
      ),
    );
  }
}
```

- **Why this solution is preferred:** Ensures reliable dependency resolution regardless of navigation path or module boundary, prevents runtime `BindNotFoundException` crashes on screen entry, and allows both modular routing and global service sharing.
- **Exceptions / caveats:** Inside the root `AppModule` itself, using `i.get<NetworkingService>()` is valid because `NetworkingService` is registered directly within `AppModule`'s injector. In test doubles, if a test boots an isolated child module without `AppModule`, it must provide a mock/fake data source or mock module.
- **Regression scenario:**
  - **Given** an authenticated user on the profile screen
  - **When** the user taps to navigate to `/profile/equipment-owner-register`
  - **Then** `EquipmentOwnerActivationCubit` resolves `NetworkingService` without `BindNotFoundException` and `EquipmentOwnerRegisterPage` mounts cleanly.
- **Automated test:**
  - `test/features/client/profile/presentation/pages/equipment_owner_register_di_test.dart`
  - Linter rule `no-child-module-root-di` in `tool/check_rules.dart`
- **Verification:** Ran `flutter test test/features/client/profile/presentation/pages/equipment_owner_register_di_test.dart` (2 tests passed), `dart run tool/check_rules.dart` (OK, 0 violations), and `tool/verify.ps1 -Fast` (`VERIFY OK`).
- **Related lessons:** None

---

## [STATE-003] Suppressing multiple error alerts during input editing after form submission failure

- **Scope:** Shared (`benaaamen-m`)
- **Categories:** state management, error handling, ui/ux, form validation, bloc
- **Lifecycle:** form submission, input editing, state emission
- **Technical areas / data flow:** BLoC / Cubit state transition, `BlocConsumer.listenWhen`, `ActionAlert.show` overlay deduplication
- **Applies when:** Form screens or edit views (e.g. profile edit, add equipment, registration) submit an action, encounter a failure, and the user subsequently edits input fields (`onChanged`).
- **Symptom:** After an API error occurs, typing any character or changing any dropdown repeatedly triggers the error toast/alert (`showErrorAlert`) on every keystroke or change, spamming the user interface.
- **Reproduction:** Submit a form with invalid data or broken connection to trigger a failure state. Then type characters in any text field or select another dropdown item. An error toast appears on each keystroke.
- **Root cause:** 
  1. Input change events in BLoC (e.g. `UpdateNameEvent`, `SelectCategoryEvent`) emit `state.copyWith(...)` without resetting `status` away from `failure` or without setting `errorMessage: null`. Thus every new keystroke emits a new state that still has `status == failure` and `errorMessage != null`.
  2. The UI `BlocConsumer` / `BlocListener` lacks a `listenWhen` condition, causing the listener to re-execute on every state emission.
  3. `ActionAlert.show` unconditionally cleared the previous overlay and mounted a new one, causing UI flashing without deduplication.
- **Rule:** 
  1. In `ActionAlert.show`, enforce automatic deduplication so identical error messages cannot re-mount within 2 seconds.
  2. In all BLoCs handling form editing, every input change handler must transition `status` back to `loaded` / `ready` and clear `errorMessage`.
  3. In all UI listeners (`BlocConsumer` / `BlocListener`), always provide `listenWhen: (prev, curr) => prev.status != curr.status || (curr.errorMessage != null && prev.errorMessage != curr.errorMessage)`.
- **Bad pattern:**

```dart
// BLoC:
void _onUpdateName(UpdateNameEvent event, Emitter<FormState> emit) {
  emit(state.copyWith(name: event.name)); // Retains status == failure and previous errorMessage!
}

// UI:
BlocConsumer<FormBloc, FormState>(
  listener: (context, state) {
    if (state.status == FormStatus.failure) {
      context.showErrorAlert(state.errorMessage!); // Fires on EVERY keystroke!
    }
  },
  builder: (context, state) => ...
)
```

- **Good pattern:**

```dart
// BLoC:
void _onUpdateName(UpdateNameEvent event, Emitter<FormState> emit) {
  emit(state.copyWith(
    status: state.status == FormStatus.failure ? FormStatus.loaded : state.status,
    errorMessage: () => null,
    name: event.name,
  ));
}

// UI:
BlocConsumer<FormBloc, FormState>(
  listenWhen: (prev, curr) =>
      prev.status != curr.status ||
      (curr.errorMessage != null && prev.errorMessage != curr.errorMessage),
  listener: (context, state) {
    if (state.status == FormStatus.failure && state.errorMessage != null) {
      context.showErrorAlert(state.errorMessage!);
    }
  },
  builder: (context, state) => ...
)

// Design system global guard (ActionAlert):
if (type == ActionAlertType.error) {
  final now = DateTime.now();
  if (_lastErrorMessage == message &&
      _lastErrorTime != null &&
      now.difference(_lastErrorTime!) < const Duration(seconds: 2)) {
    return;
  }
  _lastErrorMessage = message;
  _lastErrorTime = now;
}
```

- **Why this solution is preferred:** Provides defense-in-depth: the global design-system layer suppresses accidental duplicate toasts, while the BLoC and UI layers maintain clean state contracts without stale failure states.
- **Automated test:**
  - `test/features/shared/notifications/push_preference_cubit_device_api_test.dart`
- **Verification:** Verified with `flutter test` and `powershell -File .\tool\verify.ps1 -Fast` (`VERIFY OK`).
- **Related lessons:** None

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
- **Exceptions / caveats:** When an addition originates from an external modal (such as a variant quick-add bottom sheet) rather than the inline counter itself, pass `syncNotifier: true` with `targetQty = originalQty + event.qty` so the card badge reflects the newly accumulated total.
- **Regression scenario:**
  - **Given** a dish in cart with `qty: 3` and an in-flight `ChangeDishQuantity(targetQty: 6)` request
  - **When** the user taps minus (`quantityNotifier.value = 5` and `ChangeDishQuantity(targetQty: 5)`) before the add response completes, or deletes the item (`targetQty: 0`)
  - **Then** `quantityNotifier.value` remains `5`, the BLoC dispatches a follow-up decrement of `1` once the add resolves, and deleting to `0` resets `dish.qty` to `0` and clears `cartItemId` in a single `DELETE` call
- **Automated test:**
  - `packages/services/resturant_home/test/resturant_home_test.dart` (`flutter test packages/services/resturant_home/test/resturant_home_test.dart`)
  - `packages/services/cart/test/cart_quantity_test.dart` (`flutter test packages/services/cart/test/cart_quantity_test.dart`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — all 5 unit and BLoC tests in `resturant_home_test.dart` and all 5 regression tests in `cart_quantity_test.dart` passed
- **Related lessons:** None

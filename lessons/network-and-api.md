# Network and API lessons

Add verified lessons about REST/GraphQL, Dio, repositories, DTO/entity mapping, pagination, search, cache, retries, token refresh and concurrent requests. Before adding, compare existing lessons by root cause and solution. Use `LESSON_TEMPLATE.md`.

---

## [NET-001] Model transaction polarity from API direction field and avoid defaulting unknown types to debit in fintech entities

- **Scope:** Shared (`panda-m`)
- **Categories:** payments, API/data, architecture/state, dashboards, detail views
- **Lifecycle:** initial-load, refresh-reload, pagination
- **Technical areas / data flow:** DTO/entity mapping, fintech models, transaction feed, polarity styling
- **Applies when:** Parsing transaction histories and wallet feeds where APIs provide transaction types and explicit credit/debit directions
- **Symptom:** Credit transactions (such as order refunds or incoming adjustments) display with negative amounts (`-`), red debit styling, and wrong action icons because the transaction type was unknown and defaulted to a debit enum (e.g. `sendMoney`).
- **Reproduction:** Call wallet/transactions endpoint returning a new transaction type (e.g. `order_refund`, `order_payment`) with `direction: "credit"`. If `TransactionType.fromString` does not map the type and defaults to `sendMoney`, the UI computes `isDebit = true` and renders `- 110` in red.
- **Root cause:** The domain model relied solely on a hardcoded `TransactionType` enum to infer financial polarity (`isCredit`/`isDebit`) and used a debit enum as the parser fallback, ignoring the explicit server-side `direction` field (`credit` / `debit`).
- **Rule:** Always capture the server-provided transaction `direction` (`credit` / `debit`) in models and entities, use `direction` as the primary decider for transaction polarity, and never silently default unknown transaction types to a business-specific debit type.
- **Bad pattern:**

```dart
// Entity relying strictly on local type enum with a debit fallback
enum TransactionType {
  sendMoney('send_money'),
  receiveMoney('receive_money');

  static TransactionType fromString(String? val) {
    // Unsafe: New backend types silently turn into sendMoney (debit)
    return TransactionType.sendMoney;
  }
}

class TransactionEntity {
  final TransactionType type;
  bool get isCredit => type == TransactionType.receiveMoney;
}
```

- **Good pattern:**

```dart
enum TransactionDirection {
  credit('credit'),
  debit('debit');

  final String value;
  const TransactionDirection(this.value);

  static TransactionDirection? fromString(String? val) {
    if (val == null) return null;
    return TransactionDirection.values.firstWhere(
      (e) => e.value.toLowerCase() == val.trim().toLowerCase(),
      orElse: () => val.toLowerCase() == 'credit' ? credit : debit,
    );
  }
}

class TransactionEntity {
  final TransactionType type;
  final TransactionDirection? direction;

  // Polarity decider favors explicit server direction
  bool get isCredit =>
      direction == TransactionDirection.credit ||
      (direction == null && type.isCredit);

  bool get isDebit =>
      direction == TransactionDirection.debit ||
      (direction == null && type.isDebit);
}
```

- **Why this solution is preferred:** Resilient to new transaction types added by the backend: even before a client update maps a new transaction type enum, the transaction displays with the correct sign (+/-) and color (green/red) based on the server-provided `direction`.
- **Exceptions / caveats:** For legacy backend endpoints that do not return a `direction` property, the entity gracefully falls back to the type's `isCredit`/`isDebit` properties.
- **Regression scenario:**
  - **Given** a wallet transaction feed containing an `order_refund` item with `direction: "credit"`
  - **When** the transaction is deserialized and rendered in `RecentTransactionsWidget` or `AllTransactionsPage`
  - **Then** the amount is formatted with a `+` prefix and green color (`0xFF00C853`), and the leading icon displays an `Icons.add` badge
- **Automated test:** `packages/services/panda_pay/test/data/order_transaction_regression_test.dart` (`flutter test packages/services/panda_pay/test/data/order_transaction_regression_test.dart`)
- **Verification:** Verified on Flutter 3.35.1 / Dart 3.9.0 — `00:00 +3: All tests passed!`
- **Related lessons:** None

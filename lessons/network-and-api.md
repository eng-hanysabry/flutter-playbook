# Network and API lessons

Add verified lessons about REST/GraphQL, Dio, repositories, DTO/entity mapping, pagination, search, cache, retries, token refresh and concurrent requests. Before adding, compare existing lessons by root cause and solution. Use `LESSON_TEMPLATE.md`.

## [NET-001] Synthetic Offline Fallbacks in Write Operations Masking API Failures and Causing Data Loss on App Restart

- **Scope:** Shared
- **Categories:** API/data, architecture/state, media, testing
- **Lifecycle:** submit, upload-cancel-retry, cache-invalidation, initial-load, offline-reconnect
- **Technical areas / data flow:** repository, offline queue/cache, Dio multipart upload, BLoC state synchronization
- **Applies when:** Any Flutter Clean Architecture repository utilizing offline-first mixins or optimistic local fallback results for mutating writes/uploads.
- **Symptom:** The user uploads or adds an entity (e.g. portfolio image, document). The UI displays a success alert and shows the newly added item in the current session. However, upon app restart, the newly added item disappears and the list is empty.
- **Reproduction:**
  1. Add an item or image while connectivity check or offline wrapper defaults to offline/uninitialized state.
  2. The repository invokes an `offlineFallback` returning a synthetic model (e.g. `temp_${timestamp}`) wrapped in `Result.success`.
  3. BLoC receives `Result.success` and adds the item to the in-memory state.
  4. Terminate and restart the application.
  5. The initial GET query fetches data from the live backend, where the item was never uploaded, resulting in empty data.
- **Root cause:** 
  1. The repository wrapped write operations in an offline fallback mechanism (`BaseOfflineRepositoryMixin.executeWrite`) that returned synthetic fake models with `Result.success` when offline detection was uninitialized or false. The network write (`POST` multipart) was never actually dispatched to the remote API.
  2. HTTP GET responses were aggressively cached without invalidation on mutation, masking remote updates.
- **Rule:** Write operations (POST, PUT, DELETE, multipart uploads) in production features must execute live remote network requests directly, never report synthetic success for unsynced writes, and immediately invalidate cached GET responses on mutation.
- **Bad pattern:**

```dart
@override
Future<Result<PortfolioItemModel>> addPortfolioImage(String imagePath) {
  return executeWrite(
    onlineAction: () => remoteDataSource.addPortfolioImage(imagePath),
    offlineFallback: () => PortfolioItemModel(
      id: 'temp_${DateTime.now().millisecondsSinceEpoch}',
      imageUrl: imagePath,
      isPendingSync: true,
    ),
  );
}
```

- **Good pattern:**

```dart
@override
Future<Result<PortfolioItemModel>> addPortfolioImage(String imagePath) async {
  try {
    final model = await remoteDataSource.addPortfolioImage(imagePath);
    return Result.success(model);
  } on ServerException catch (e) {
    return Result.failure(ServerFailure(e.message));
  } catch (e) {
    return Result.failure(ServerFailure(e.toString()));
  }
}
```

- **Why this solution is preferred:** Ensures genuine data integrity. The UI only indicates success when the backend has persisted the entity. Network caches are explicitly cleared upon mutation, ensuring subsequent GET queries reflect true server state across app restarts.
- **Exceptions / caveats:** Dedicated offline-first features with robust sync engines and persistent local databases (e.g., Hive/Isar with queue worker) may queue operations locally, but must clearly mark items as pending sync in UI rather than returning fake permanent IDs as genuine success.
- **Regression scenario:** Given an active provider adding a portfolio image, When the upload is triggered, Then the genuine multipart API request is executed, cache is cleared, and only genuine server-persisted models are returned.
- **Automated test:** `test/features/provider/portfolio_live_contract_test.dart`
- **Verification:** Ran `flutter test test/features/provider/portfolio_live_contract_test.dart` (all 6 tests passed) and full suite `flutter test test/features/provider/` (all 142 tests passed).
- **Related lessons:** None

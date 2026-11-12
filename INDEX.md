# Company Flutter Bug Playbook Index

**Automatic-use instruction:** The agent identifies the project, requested work, affected feature/screen, lifecycle and technical areas, then reads only the matching topic file(s). Every lesson may carry multiple category/lifecycle/data-flow tags, but it is stored once only.

| Work area / category | Read | Includes |
|---|---|---|
| State management, Cubit, BLoC, Riverpod, Provider, async state, controllers, streams, disposal | `lessons/state-and-lifecycle.md` | emit after close, stale async results, request overlap, controller disposal, subscriptions |
| Screens, widgets, forms, responsive layout, themes, ScreenUtil, accessibility, keyboard/scroll behavior | `lessons/ui-and-widgets.md` | overflow, text scaling, context usage, rebuilds, shared widget behavior, validation UX |
| REST/GraphQL, Dio, repositories, DTO/entity mapping, pagination, search, cache, retry, auth refresh | `lessons/network-and-api.md` | duplicate requests, pagination race, token refresh, stale cache, serialization and error mapping |
| Navigation, deep links, app lifecycle, notifications, maps/location, uploads, native Android/iOS, build/release | `lessons/navigation-and-platform.md` | route disposal, async context, permissions, plugin lifecycle, Podfile/Gradle and platform differences |

## Deduplication (mandatory)

Compare confirmed root cause, trigger/reproduction conditions, affected lifecycle, category/data-flow tags, package/platform/version applicability, unsafe pattern, preferred solution, exceptions and regression behavior—not just title/error text.

- Same root cause + materially same solution: reuse; no duplicate lesson, commit, push or PR.
- Same root cause + better verified solution: improve the existing canonical lesson once.
- Same symptom + different confirmed root cause: create a separate linked lesson.
- Same general issue + unique non-sensitive project context: retain one general lesson and add only unique project notes.
- Unclear cause or unverified fix: keep as a candidate; do not publish as an approved rule or auto-merge.
- Multiple solutions valid for different SDK/platform/architecture conditions: preserve explicit variants.

## Scope and privacy

- Shared lessons are reusable engineering principles.
- Project-specific notes describe only unique, non-sensitive API or architecture context.
- Project branches are contribution lanes, not access-control boundaries. Anything merged to `main` is visible to all repository readers. Do not store secrets, customer data, private endpoints or confidential business rules.

## Project branches

Use one persistent external branch per registered application: `project/<project-id>`. This branch belongs to `flutter-playbook`, never to the application repository.

## Suggested classification tags

Categories: forms, lists/pagination, detail views, dashboards, authentication, permissions, navigation, search, maps, realtime, payments, media, profile/settings, onboarding, multitenancy, shared components, responsive UI, background tasks, build/release, architecture/state, API/data, performance/memory, security, testing, other.

Lifecycle: initial-load, refresh-reload, pagination, search-debounce, submit, update-save, delete, navigation-dispose, background-resume, offline-reconnect, retry-timeout-cancel, concurrent-race, cache-invalidation, realtime-events, notification-deeplink, permission-change, auth-refresh, tenant-switch, upload-cancel-retry, resize-rotation, localization, build-install-release.

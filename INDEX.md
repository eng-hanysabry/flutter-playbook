# Company Flutter Bug Playbook Index

## Mandatory Pre-Investigation Knowledge Retrieval Protocol (Strict Gate)

For every Flutter engineering task, before investigating, reading, or modifying application code:
1. **Verify Worktree & Read Foundation:** Verify project registration and branch `project/<project-id>`, then read `GLOBAL_AGENT_RULES.md` and this `INDEX.md`.
2. **Search Lessons by Topic & Pattern:** Identify matching topic file(s) from the table below. Search for existing lessons related to the reported issue, including relevant concepts, architectural components, symptom keywords, implementation mechanisms, and previous bug patterns.
3. **Read Lessons Completely:** Read all identified relevant lessons completely from beginning to end. If a relevant lesson cannot be found or read, report that explicitly before proceeding.
4. **Internal Relevance Evaluation & Multi-Lesson Combination (Visible Preflight):**
   - Identify which previous lessons apply and explain their relevance before inspecting or modifying application code.
   - Combine multiple lessons when evidence supports their relevance to different facets of the problem.
   - Treat lessons as investigation guidance rather than predetermined diagnoses.
   - Emit the mandatory visible preflight block before inspecting any application source code:
     ```
     PLAYBOOK PREFLIGHT
     - Project ID: <project-id>
     - Worktree and branch: <path> (<branch>)
     - Relevant lessons found: <lesson-ids or 'None'>
     - Why each lesson is relevant: <brief explanation per lesson>
     - Investigation areas suggested by the lessons: <suggested focus areas>
     - Knowledge retrieval completed.
     ```
5. **Only Then Proceed:** Only after steps 1-4 are verified in execution order and the visible preflight is shown may application code investigation and implementation begin.

> [!IMPORTANT]
> **Integrity Requirement:** Never claim a lesson was applied if it was read after application investigation or implementation started. Never claim pre-investigation reads occurred unless the execution order in the tool trace explicitly proves it.

## Multi-Layer Investigation & Cross-Feature Consistency
- **Multi-layer root-cause analysis:** Select investigation categories based on evidence (business logic, state management, event ordering, API synchronization, widget lifecycle, shared caches, navigation, recovery). See `GLOBAL_AGENT_RULES.md` Section 3.
- **Cross-feature behavioral consistency:** Different features may use different state-management architectures, but when implementing the same business operation, they must satisfy the same expected business behavior without forcing identical architecture. See `GLOBAL_AGENT_RULES.md` Section 4.
- **Prioritized scenario coverage:** Test selection must prioritize scenarios relevant to failure and regression risk (rapid repeated actions, out-of-order responses, boundary values, state clobbering, cross-feature parity). See `GLOBAL_AGENT_RULES.md` Section 6.

| Work area / category | Read | Includes |
|---|---|---|
| State management, Cubit, BLoC, Riverpod, Provider, async state, controllers, streams, disposal | `lessons/state-and-lifecycle.md` | emit after close, stale async results, request overlap, controller disposal, subscriptions |
| Screens, widgets, forms, responsive layout, themes, ScreenUtil, accessibility, keyboard/scroll behavior | `lessons/ui-and-widgets.md` | overflow, text scaling, context usage, rebuilds, shared widget behavior, validation UX |
| REST/GraphQL, Dio, repositories, DTO/entity mapping, pagination, search, cache, retry, auth refresh | `lessons/network-and-api.md` | duplicate requests, pagination race, token refresh, stale cache, serialization and error mapping |
| Navigation, deep links, app lifecycle, notifications, maps/location, uploads, native Android/iOS, build/release | `lessons/navigation-and-platform.md` | route disposal, async context, permissions, plugin lifecycle, Podfile/Gradle and platform differences |

## Deduplication (mandatory)

Compare confirmed root cause, trigger/reproduction conditions, affected lifecycle, category/data-flow tags, package/platform/version applicability, unsafe pattern, preferred solution, exceptions and regression behavior—not just title/error text.

- Same root cause + materially same solution: reuse; no duplicate lesson, commit, or push.
- Same root cause + better verified solution: improve the existing canonical lesson once.
- Same symptom + different confirmed root cause: create a separate linked lesson.
- Same general issue + unique non-sensitive project context: retain one general lesson and add only unique project notes.
- Unclear cause or unverified fix: keep as a candidate; do not publish as an approved rule.
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

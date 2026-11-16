# Global Flutter Playbook — mandatory agent operating rules

These rules are stored in the independent `flutter-playbook` Git repository. The application only has the small always-on bootstrap rule at `.agents/rules/flutter-playbook-bootstrap.md` (or its equivalent in the tool's always-on rules mechanism). All knowledge and playbook Git operations remain external.

## 1. Automatic behavior & mandatory 6-step gated sequence

The bootstrap rule runs at the start of every user turn in an active Flutter application workspace.
For EVERY task that investigates, reviews, modifies, tests, or refactors Flutter/Dart application code—including bug fixes, feature development, UI changes, refactoring, code reviews, debugging, testing, and performance improvements—you MUST execute this mandatory 6-step pre-investigation sequence in exact order before touching or inspecting application code:

### Mandatory 6-Step Pre-Investigation Sequence (Strict Gate)
1. **Verify project identity & external worktree:**
   - Verify that the application's registered project ID matches the workspace.
   - Verify that the external Playbook worktree exists, is clean, and is on branch `project/<project-id>`.
2. **Read foundation rules:**
   - Read `GLOBAL_AGENT_RULES.md` from the external worktree.
   - Read `INDEX.md` from the external worktree.
3. **Search Playbook for related lessons:**
   - Search the Playbook topic files (`lessons/`) for existing lessons related to the reported issue, including relevant concepts, symptom keywords, implementation mechanisms, architectural patterns, and previous bug patterns.
4. **Read all relevant lessons completely:**
   - Read all identified relevant lessons completely from start to finish (do not skim or read only titles/summaries).
   - If a relevant lesson cannot be found or read, report that explicitly before proceeding.
5. **Identify, combine & explain relevance (Visible Playbook Preflight):**
   - Identify which previous lessons apply (single or combined) and explain their relevance before investigating or reading application code.
   - Combine multiple lessons when evidence supports their relevance to different aspects of the issue.
   - Distinguish confirmed causes from hypotheses, and treat lessons as guidance rather than predetermined diagnoses.
   - Emit the mandatory visible preflight block in the response before inspecting any application source code:
     ```
     PLAYBOOK PREFLIGHT
     - Project ID: <project-id>
     - Worktree and branch: <path> (<branch>)
     - Relevant lessons found: <lesson-ids or 'None'>
     - Why each lesson is relevant: <brief explanation per lesson>
     - Investigation areas suggested by the lessons: <suggested focus areas>
     - Knowledge retrieval completed.
     ```
6. **Begin application investigation & implementation:**
   - **ONLY THEN** begin application investigation, reproduction, and implementation.

### Strict Execution & Integrity Rules
- **Strict Pre-Investigation Gate:** Never read, search, or modify application source files before completing steps 1 through 5 and emitting the visible PLAYBOOK PREFLIGHT.
- **Zero Retroactive Claims:** Never claim a lesson was applied if it was read after application investigation or implementation started.
- **Evidence-Based Order Proof:** Never claim pre-investigation reads occurred unless the execution order in the tool calls and execution trace explicitly proves it. Claiming pre-investigation reads when files were read after application code was read or edited is a critical integrity violation.
- **Explicit Reporting of Missing Knowledge:** If a relevant lesson cannot be found or read, report that explicitly before proceeding.
- **Mandatory activation:**
  - The user does NOT need to mention the Playbook or paste a special execution prompt.
  - A normal engineering request is sufficient to activate this workflow.
  - Do not wait for the user to ask for investigation, testing, documentation, or Playbook updates.
  - Do not treat the Playbook as optional guidance.
- **Preflight failure:**
  - If the Playbook worktree, required instructions, or relevant lessons cannot be located or read, stop before implementation and report the exact problem.
  - Never claim the workflow was activated successfully unless the required pre-investigation reads succeeded in proper sequence.
- **Scope control:**
  - Always implement the smallest correct change that satisfies the user's request. Do not introduce unrelated refactoring or improvements.
  - For non-engineering conversations, explanations, translations, and general questions that do not involve Flutter code changes or reviews, do not execute the engineering workflow.

## 2. Project and lesson retrieval & combining multiple lessons

1. Identify the application by normalized Git `origin` remote, not its folder name.
2. Resolve the project profile/registry record and external branch `project/<project-id>`.
3. Synchronize the project branch with the latest shared `main` before reading lessons.
4. Read `INDEX.md` first. Classify work by screen/feature category, lifecycle, data flow, engineering domain, shared components, source symbols, platform, SDK and package versions.
5. Search the Playbook topic files (`lessons/`) for existing lessons related to the reported issue, including relevant concepts, symptoms, implementation mechanisms, architectural patterns, and previous bug patterns.
6. Read all identified relevant lessons completely from beginning to end (do not skim or read only titles/summaries). If a relevant lesson cannot be found or read, report that explicitly before proceeding.
7. Identify how each retrieved lesson relates to the current issue. Combine multiple lessons when evidence supports their relevance across different architectural layers, data flows, or failure symptoms.
8. Treat lessons as investigation guidance and hypotheses, not as predetermined diagnoses or complete solutions. A lesson can guide investigation without necessarily describing the ultimate root cause.
9. Distinguish confirmed causes from hypotheses. Avoid forcing unrelated lessons into the investigation.
10. Reuse existing knowledge instead of creating duplicate lessons.
11. Apply only lessons whose applicability and exceptions match the current project. Never copy another project's API/business assumptions.
12. Preserve the existing project architecture and conventions. Do not refactor unrelated code merely to match a preferred pattern.

## 3. Evidence-based root-cause investigation across layers

The investigation must consider relevant causes across multiple layers based on available evidence, rather than assuming a single point of failure or blindly executing an inflexible checklist. Select relevant areas according to the issue and available evidence:

- **Business logic and validation:** Inconsistent business constraints, missing boundary checks, divergent validation rules between forms or surfaces, unhandled edge-case domain states.
- **State management and state transitions:** Illegal transitions, race conditions between user actions and server updates, unhandled intermediate states, inconsistent state mutations.
- **Event ordering and asynchronous operations:** Concurrency races, event transformers dropping or buffering events unexpectedly, unawaited Futures, out-of-order completion.
- **API requests, responses, and synchronization:** Optimistic updates colliding with in-flight HTTP requests, stale server responses overwriting newer user changes, improper retry or error handling, pagination synchronization failures.
- **Widget lifecycle and rebuild behavior:** Subscriptions attached inside build methods, missing cleanup on dispose, unwanted rebuild loops, state lost across route push/pop.
- **Shared providers, repositories, services, and caches:** Cache invalidation races, singleton or scoped service leaks, out-of-sync multi-consumer state, shared repository state mutations.
- **Navigation and feature interactions:** State preserved or reset unintentionally during navigation, callbacks executing after widget unmount, deep linking state mismatch.
- **Loading, failure, retry, and recovery behavior:** Incomplete error recovery, retry logic re-emitting stale parameters, failure states masking valid local modifications.

### State-management-specific investigation
When state management could be involved, investigate its actual behavior using concrete evidence rather than speculation:

- **For BLoC / Cubit:**
  - Event ordering and event transformers: concurrent versus sequential transformers (`droppable`, `restartable`, `sequential`, `concurrent`).
  - Multiple events arriving while previous handlers are running.
  - State emission and equality behavior (`Equatable.props`, distinct states, state emissions dropped due to missing fields).
  - Filtering conditions: `listenWhen` and `buildWhen`.
  - `BlocListener` placement, subscriptions, and lifecycle (e.g. duplicate listeners, listeners placed inside build methods or rebuildable child widgets).
  - Multiple listeners responding to the same state change.
  - Nested or shared BLoCs and provider scopes.
  - State being overwritten by reloads or unrelated events.
  - Listeners triggering additional events or causing feedback loops.
  - Asynchronous handlers emitting stale results after subsequent user interactions.
- **For other state-management solutions (Riverpod, Provider, MobX, Signals):**
  - Investigate equivalent mechanisms (e.g. `ref.watch` vs `ref.listen`, provider scoping and overrides, `autoDispose` lifecycle, notification loops, synchronous vs asynchronous notification order) rather than applying BLoC-specific assumptions.
- **Strict Evidence Requirement:** Do not assume that a listener, event transformer, or state-management mechanism is the cause without verified evidence from tracing, reproduction, or code inspection.

## 4. Cross-feature behavioral consistency

### Core Principle
Different features may use different state-management architectures, but when they implement the same business operation, they must satisfy the same expected business behavior.

For example, a Cart screen and a "Products You May Like" carousel or catalog may employ entirely different state-management approaches (e.g. BLoC with sequential event transformers versus a dedicated Cubit, Riverpod provider, or ValueNotifier), but quantity adjustments, item additions, removals, and boundary limits must yield consistent expected outcomes across both surfaces.

### Mandatory Agent Actions
When an issue touches or shares a business operation implemented across multiple features:
1. **Identify shared business rules and expected behavior:** Determine the canonical business requirements (e.g. quantity boundaries, zero-handling, stock limits, optimistic synchronization).
2. **Inspect each relevant implementation independently:** Examine the actual code of each feature rather than assuming they share the same architecture or defects.
3. **Compare behavior rather than assuming identical architecture:** Evaluate whether each implementation satisfies the shared business expectations in its own context.
4. **Identify whether defects share a root cause or have independent causes:** Determine whether the failure stems from a shared underlying component (e.g. shared repository/API) or independent feature-level implementation defects.
5. **Fix each confirmed cause using the appropriate implementation:** Apply the fix tailored to each feature's architectural pattern without forcing a foreign pattern upon it.
6. **Verify all affected implementations against the same expected behavior:** Run regression verification across all affected features to confirm behavioral consistency.
7. **Preserve architectural autonomy:** Do not force unrelated features to use identical state management architectures. Parity of business behavior is required; homogeneity of implementation is not.

## 5. Mandatory duplicate and solution comparison

Before creating or updating a lesson, compare all relevant lessons using confirmed root cause, reproduction conditions, lifecycle, categories, data flow, symbols, packages, SDK/platform/architecture constraints, bad pattern, preferred solution, exceptions and regression behavior. Do not compare titles/error strings alone.

- Same root cause + same or materially equivalent verified solution: reuse the existing canonical lesson. No duplicate file, commit, or push.
- Same root cause + a safer/better verified solution: update the existing canonical lesson once; preserve valid caveats and compatibility variants.
- Same symptom + different confirmed root cause: create a separate lesson and cross-link it.
- Same general issue + unique non-sensitive project context: keep the general lesson once and add only the unique project note.
- Unknown root cause, unverified fix or failed test: do not publish as approved knowledge. Keep it as a clearly marked candidate on the project branch, or report why no lesson can yet be recorded.
- If two solutions both work, prefer the safer, simpler, better-tested and more compatible option. Keep both only when SDK/platform/architecture conditions justify variants.

## 6. Fix, test application code & scenario coverage

1. Reproduce the issue when practical. Separate confirmed facts from hypotheses.
2. Implement the requested fix using project conventions and applicable lessons.
3. Add/update a real regression test in the application's existing test structure.
4. **Select tests based on actual issue, relevant lessons, and confirmed root causes.**
   Prioritize scenarios relevant to the failure and its regression risk (do not blindly test every theoretical scenario):
   - **Rapid repeated user actions:** Quick taps (e.g. increment/decrement bursts), debounce/throttle stress.
   - **Requests completing out of order:** Network responses arriving in non-chronological order.
   - **Stale responses overwriting newer state:** Slower previous requests finishing after newer user changes.
   - **Duplicate events or listeners:** Multiple triggers or duplicate subscriptions firing in response to a single user interaction.
   - **Widget disposal and recreation:** Navigating back or unmounting widgets while asynchronous work or animations are pending.
   - **Navigation away and back during an operation:** Pushing a new route and returning while an operation is in-flight.
   - **Loading, success, failure, and retry:** Transitioning through all loading and error states, including recovery after failure.
   - **Zero, minimum, and maximum values:** Boundary conditions (e.g. decrementing to zero, exceeding maximum stock, empty inputs).
   - **Multiple independent items updating simultaneously:** Modifying multiple list items or entities concurrently.
   - **Initial API loading while local changes are pending:** User interacting with cached or partial data while full remote data loads.
   - **Shared state being modified by another feature:** State mutations originating from an external feature or background sync.
5. **Verify cross-feature behavioral consistency:** Where multiple features implement the same business operation, verify their behavioral consistency against the expected business outcome.
6. Report exact test paths and actual results: passed, failed, blocked, or not run. A Given/When/Then lesson is not proof that an executable test exists.
7. If a regression test cannot reasonably be automated, explain why and provide reproducible manual steps.

## 7. Capture knowledge after a resolved bug

Knowledge capture is part of completing a resolved bug. Do not ask whether the developer wants to remember it.

1. Run duplicate comparison first.
2. Reuse an equivalent canonical lesson; do not append the same lesson again.
3. Improve an existing lesson only when the new solution is materially better and verified.
4. Create one new lesson only when the root cause is genuinely new.
5. Use the relevant one of the four topic files initially. A lesson may carry multiple screen-category, lifecycle and data-flow tags but must be stored once.
6. Include: unique ID/title; shared or project-specific scope; relevant categories/lifecycle/data-flow tags; SDK/package/platform/architecture applicability; symptom; reproduction; confirmed root cause or explicit hypothesis; concise prevention rule; concrete BAD and GOOD Dart examples; exceptions; Given/When/Then scenario; actual regression test path/command/result; related lesson IDs.
7. Keep project-specific notes separate and non-sensitive. Never store secrets, credentials, private endpoints, customer PII or proprietary business rules in a repository accessible to other projects.
8. Update `INDEX.md` only when a new routing topic or meaningful coverage is introduced.

## 8. Automatic playbook Git workflow — manual PR and merge ownership

Only perform Git writes when a lesson is genuinely created or materially improved, or a necessary project registry/profile change is made.

1. Use the persistent external branch `project/<project-id>`; never create this branch in the application repository.
2. Stage only intended playbook files. Never mix application source/test files into the playbook commit.
3. Commit and push the project branch (`git push origin project/<project-id>`) automatically after deduplication, verification and privacy checks pass.
4. **The user owns the entire Pull Request creation and merge process.** The agent must **NOT**:
   - create a new Pull Request;
   - update an existing Pull Request;
   - request auto-merge;
   - merge any Pull Request;
   - push directly to `main`;
   - attempt to bypass branch protection or review requirements.
5. After the user manually merges a PR into `main`, fetch and synchronize the project branch with `origin/main` safely on subsequent turns.
6. If the lesson is duplicate and unchanged, do not create an empty commit or push.

## Playbook Git Push & Manual PR Ownership

### Authentication & Execution
- Use existing Git Credential Manager authentication for all Git operations.
- Do not require GitHub CLI (`gh`) and do not ask for interactive login during engineering tasks.
- Never extract credentials from credential stores, expose tokens, or save credentials in repositories.
- If a Git push fails, report the actual error clearly and do not claim success.

### Git Operations
Execute standard Git operations in the external project worktree:
- `git fetch origin`
- `git status`
- `git diff`
- `git add`
- `git commit`
- `git push origin project/<project-id>`

### Safety Boundaries
- Never push directly to `main`.
- Never force-push (`--force`).
- Never create, update, auto-merge, or merge Pull Requests automatically.
- Never bypass reviews or branch protections.
- Keep application source code and regression tests in the application repository.
- Keep playbook knowledge in the external playbook repository.
- Preserve unrelated changes and never create empty commits when there is no new or improved verified knowledge.

## 9. Concurrency and recovery

- Use an external worktree per project per developer. Never put it under an application repository.
- Never allow two local agents to edit the same worktree concurrently. If it is dirty/locked/in use, use another isolated clone/worktree or stop playbook writes; never overwrite changes.
- Fetch before pushing. If another developer pushed meanwhile, integrate changes safely and retry. Never force-push.
- Stop on conflicts and report them; do not auto-resolve by discarding either side.
- If playbook access is unavailable, report the blocker honestly and continue the application task where safe.

## 10. Scope and approval boundary

Automatically perform the requested application task, relevant prevention checks, regression tests and bug-lesson capture. Do not silently fix unrelated screens/modules, refactor broad architecture, upgrade dependencies or change unrelated behavior. Show exact proposed file locations, reason, benefit and risk, then ask the developer before optional related application fixes.

## 11. Final response

After code-affecting work, report concisely:
- detected project ID and verified worktree;
- pre-investigation reads and lessons searched/combined (with verified execution order);
- visible Playbook preflight execution proof;
- confirmed root cause across relevant layers (state management, API, lifecycle, business logic);
- cross-feature behavioral consistency evaluation (when shared business operations are involved);
- lessons searched and reused/created/updated/no-change decision;
- regression test paths and exact outcomes across prioritized scenarios;
- after a successful Playbook push:
  - project branch name (`project/<project-id>`);
  - commit hash;
  - whether the push was verified against the remote;
  - files changed;
  - whether a new or updated lesson was included;
  - state clearly: `"Playbook changes have been pushed to project/<project-id>. PR creation and merging are intentionally left to the user."`
- optional related fixes awaiting approval.

Never claim a lesson was applied if it was read after application investigation or implementation started. Never claim pre-investigation reads occurred unless the execution order in the tool trace explicitly proves it. Do not report PR creation, merge readiness, or merge status as completed. If the push fails, report the actual error and do not claim success.

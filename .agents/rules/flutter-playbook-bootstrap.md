---
trigger: always_on
description: "Automatically initialize and synchronize the shared Flutter Playbook for every project task."
---

# ALWAYS-ON: Shared Flutter Playbook Bootstrap

> **This rule runs automatically.** Apply it at the beginning of every user turn while a Flutter application repository is the active workspace. The developer must never mention the playbook, repeat setup instructions, or use a special prompt. The developer only describes the issue/task and, when asking for a fix, asks naturally to investigate/fix/solve it.

This small rule is the only playbook integration allowed in an application repository. The actual playbook, lessons, project notes, branches, worktrees and Git history stay in the independent `flutter-playbook` repository outside the application.

## A. Automatic bootstrap and synchronization — every user turn

1. Identify the active application Git root and `origin` remote using Git. Do not infer project identity from a folder name.
2. Check the external clone:
   - Windows: `%USERPROFILE%/dev/flutter-playbook`
   - macOS/Linux: `~/dev/flutter-playbook`
3. If missing, clone the configured company repository URL below. If it exists, verify its remote and fetch updates. Never overwrite, reclone over, reset, clean or delete an existing clone.
4. Company playbook URL (the repository owner replaces this placeholder once):

   `https://github.com/eng-hanysabry/flutter-playbook.git`

5. Match the application's normalized remote (`host/owner/repository`, treating SSH and HTTPS forms as equivalent) against the central project registry at `projects/registry/`.
6. If not registered, derive a stable project ID from the canonical remote, check for collisions, and create a registry record on the project's playbook branch. Do not guess from the local directory name. If there is no unique `origin` remote or the identity is ambiguous, ask only for the missing project identity.
7. Ensure the external worktree exists:
   - Central clone: `<home>/dev/flutter-playbook`
   - Project worktree: `<home>/dev/flutter-playbook-worktrees/<project-id>`
   - Persistent branch: `project/<project-id>`
8. Fetch remote refs. If the remote project branch exists, create/open its tracking worktree. Otherwise create the branch from `origin/main`, create the worktree, and push the initial branch. This branch belongs to the playbook Git repository, never to the application repository.
9. Verify the worktree belongs to the central repository, is on the correct branch, and is clean. Pull the project branch using fast-forward-only behavior, then merge `origin/main` into the project branch to receive lessons contributed by other projects.
10. If the clone URL, credentials, registry, worktree, lock, dirty state or merge conflicts prevent safe synchronization, state the exact blocker. Never discard changes, force-push, or pretend sync succeeded. Continue the requested application task when safe, but do not write/push playbook knowledge until the blocker is resolved.
11. After successful synchronization, read the external `GLOBAL_AGENT_RULES.md` and `INDEX.md`. Retrieve only matching lesson content when the request involves project code, design, debugging, tests, build/release or engineering decisions.

## B. How to interpret every user prompt — Mandatory 6-Step Gated Sequence

For EVERY task involving Flutter/Dart application code (bug fixes, feature development, UI changes, refactoring, code reviews, debugging, testing, and performance improvements), automatically follow the external Flutter Playbook workflow.

### Mandatory 6-Step Pre-Investigation Sequence (Strict Gate)
Before investigating, reading, or modifying application code, you MUST execute these steps in exact sequential order:
1. **Verify project ID and external worktree:** Verify that the application's registered project ID matches and that the external Playbook worktree exists, is clean, and is on branch `project/<project-id>`.
2. **Read foundation rules:** Read `GLOBAL_AGENT_RULES.md` and `INDEX.md` from the external worktree.
3. **Search Playbook for related lessons:** Search the Playbook topic files (`lessons/`) for existing lessons related to the reported issue, including relevant concepts, symptoms, implementation mechanisms, architectural components, and previous bug patterns.
4. **Read all relevant lessons completely:** Read all identified relevant lessons completely from beginning to end (do not skim or read only titles/summaries). If a relevant lesson cannot be found or read, report that explicitly before proceeding.
5. **Identify, combine & explain relevance (Visible Playbook Preflight):**
   - Identify which previous lessons apply (single or combined) and explain their relevance before inspecting or modifying application code.
   - Combine multiple lessons when evidence supports their relevance to different facets of the problem.
   - Treat lessons as guidance rather than predetermined diagnoses.
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
6. **Begin application investigation & implementation:** ONLY THEN begin application investigation, reproduction, and implementation.

### Strict Execution & Integrity Rules
- **Strict Pre-Investigation Gate:** Never read, search, or modify application source files before completing steps 1 through 5 and emitting the visible PLAYBOOK PREFLIGHT.
- **Zero Retroactive Claims:** Never claim a lesson was applied if it was read after application investigation or implementation started.
- **Evidence-Based Order Proof:** Never claim pre-investigation reads occurred unless the execution order in the tool calls and execution trace explicitly proves it. Claiming pre-investigation reads when files were read after application code was read or edited is a critical integrity violation.
- **Explicit Reporting:** If a relevant lesson cannot be found or read, report that explicitly before proceeding.
- **Evidence-Based Multi-Layer Root-Cause Analysis:** Investigate causes across relevant layers (business logic, state management, event ordering, API synchronization, widget lifecycle, shared caches, navigation, recovery) based on evidence. For state management (BLoC, Riverpod, Provider, etc.), investigate actual event transformers, listeners, equality, and emissions without guessing.
- **Cross-Feature Behavioral Consistency:** Different features implementing the same business operation (e.g. Cart vs Recommendations quantity changes) must satisfy the same expected business behavior without forcing identical architecture.
- **Prioritized Scenario Coverage:** Select tests prioritizing relevant failure scenarios (rapid repeated actions, out-of-order responses, stale responses, boundary values, cross-feature parity).
- **Deduplication:** Reuse existing lessons; do not create duplicate lessons or commits.
- **Minimal Changes:** Implement only what is strictly necessary to satisfy the user's request.
- **Approval Boundaries:** Never create, update, auto-merge, or merge Pull Requests (owned by user).
- **External Git Only:** Commit and push only justified Playbook knowledge changes to `project/<project-id>`.
- **Preflight failure:** If the Playbook worktree, required instructions, or relevant lessons cannot be located or read, stop before implementation and report the exact problem. Never claim the workflow was activated successfully unless the required pre-investigation reads succeeded.
- **Scope control:** Always implement the smallest correct change that satisfies the user's request without unrelated refactoring. For non-engineering conversations, explanations, translations, and general questions that do not involve Flutter code changes or reviews, do not execute the engineering workflow.

## C. Git and approval boundaries

- All playbook Git operations use the external playbook clone/worktree. Never add lessons, scripts, submodules, worktrees, registry files or playbook Git remotes to the application repository.
- The application fix and its executable regression tests remain in the application's existing repository and follow that repository's normal commit/push policy. Do not automatically push application code unless the developer's normal workflow explicitly authorizes it.
- A genuine playbook knowledge change is committed and pushed automatically to `project/<project-id>`. The agent must not create or update Pull Requests, request auto-merge, or merge Pull Requests; PR creation and merging into `main` are owned by the user.
- A duplicate lesson creates no duplicate file, commit, or push. No knowledge change means no playbook Git write.
- Never bypass branch protection, reviews, checks or merge queues. Never use `--admin`, force-push, reset/clean away user work, or push directly to `main`. After a verified push, state: "Playbook changes have been pushed to `project/<project-id>`. PR creation and merging are intentionally left to the user."
- Do not apply unrelated application fixes without first showing their exact locations, rationale, benefits and risks and asking for approval.

## D. No extra prompt

The developer's entire request can be as simple as:

- "The orders list repeats page 2 after pull-to-refresh. Please fix it."
- "The profile screen crashes when I navigate back while loading. Find and fix the cause."
- "Add a filter to the products screen."

The rule handles bootstrap, project detection, branch selection, synchronization, lesson retrieval, applicable tests and knowledge contribution automatically. Do not ask the developer to add workflow instructions to their prompt.

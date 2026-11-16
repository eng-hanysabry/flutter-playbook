# Project Branch, Deduplication and Automatic Workflow

## Developer experience

The developer does not run setup commands, select a branch, mention the playbook, write lessons, or push playbook branches. They open the application and describe the issue or requested feature in ordinary language. The always-on bootstrap rule handles playbook commit and push to the project branch automatically, while PR creation and merging into `main` are owned by the user.

Examples of normal requests:
- "The orders list repeats page 2 after refresh. Please fix it."
- "The profile screen crashes when I navigate back during loading. Find and fix the cause."
- "Add a status filter to the orders screen."

No special issue prompt is required.

## External paths

- Central clone: `<home>/dev/flutter-playbook`
- Project worktree: `<home>/dev/flutter-playbook-worktrees/<project-id>`
- Persistent external branch: `project/<project-id>`

These must never be located under an application Git root.

## Per-user/per-project workflow

At the beginning of every user turn in the active Flutter workspace:
1. Identify the application remote.
2. Check/clone the central repository.
3. Fetch the latest refs.
4. Identify or safely register the project.
5. Ensure its external branch/worktree.
6. Fast-forward the project branch and merge the latest `main`.
7. Execute the mandatory pre-investigation knowledge retrieval sequence before touching application code:
   a. Verify project ID and external worktree on `project/<project-id>`.
   b. Read `GLOBAL_AGENT_RULES.md` and `INDEX.md`.
   c. Search Playbook topic files (`lessons/`) for existing lessons related to the reported issue, relevant concepts, implementation mechanisms, and previous bug patterns.
   d. Read all relevant lessons completely. If none found, report explicitly before proceeding.
   e. Identify which lessons apply, combine multiple lessons where evidence supports it, explain their relevance, and emit the visible `PLAYBOOK PREFLIGHT` block before inspecting application code.
   f. Only then begin application investigation across relevant architectural layers, reproduction, and minimal implementation.

Never claim a lesson was applied if read after implementation started. Execution order must prove pre-investigation reads occurred before application code was read or edited.

For implementation/bug-fix requests, continue with prevention, implementation and test selection. For informational requests, do not mutate code or knowledge unless appropriate.

## Deduplication

Compare confirmed root cause, triggers, lifecycle, categories, data flow, symbols, package/SDK/platform versions, bad pattern, preferred solution, exceptions and regression behavior.

- Same cause + same solution: reuse; no new lesson or Git push.
- Same cause + better verified solution: update existing canonical lesson once.
- Same symptom + different confirmed cause: create a new lesson and cross-link.
- Same general issue + project-specific non-sensitive context: keep one general lesson and add only the unique context.
- Unverified cause/fix: label candidate; do not publish as authoritative.

## Regression tests, multi-layer analysis, and cross-feature consistency

Application source fixes and executable tests stay in the application repository. Investigate root causes across relevant layers (business logic, state management, event ordering, API synchronization, widget lifecycle, shared caches, navigation, recovery) using evidence. Where multiple features implement the same business operation, verify behavioral consistency without forcing identical architecture. Run relevant tests prioritizing key failure scenarios (rapid repeated actions, out-of-order responses, boundary conditions, cross-feature parity) and report exact paths/results. The playbook stores reusable scenarios and references, not copies of application test source.

## Git automation and manual PR ownership

Only for genuine playbook knowledge/registry changes:
- commit and push to the external project branch (`project/<project-id>`);
- do not automatically create, update, auto-merge, or merge Pull Requests;
- leave PR creation and merging into `main` to the user;
- once the user merges to `main`, sync the project branch with the new `main` on subsequent turns.

A duplicate lesson or no knowledge change causes no duplicate commit or push. Never push application source as part of playbook Git operations.

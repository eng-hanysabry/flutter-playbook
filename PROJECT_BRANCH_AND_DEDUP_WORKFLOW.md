# Project Branch, Deduplication and Automatic Workflow

## Developer experience

The developer does not run setup commands, select a branch, mention the playbook, write lessons, pull/push Git, or create PRs. They open the application and describe the issue or requested feature in ordinary language. The always-on bootstrap rule handles the rest.

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
7. Read index and matching lessons.

For implementation/bug-fix requests, continue with prevention, implementation and test selection. For informational requests, do not mutate code or knowledge unless appropriate.

## Deduplication

Compare confirmed root cause, triggers, lifecycle, categories, data flow, symbols, package/SDK/platform versions, bad pattern, preferred solution, exceptions and regression behavior.

- Same cause + same solution: reuse; no new lesson or Git push.
- Same cause + better verified solution: update existing canonical lesson once.
- Same symptom + different confirmed cause: create a new lesson and cross-link.
- Same general issue + project-specific non-sensitive context: keep one general lesson and add only the unique context.
- Unverified cause/fix: label candidate; do not auto-merge as authoritative.

## Regression tests

Application source fixes and executable tests stay in the application repository. Run relevant tests and report exact paths/results. The playbook stores reusable scenarios and references, not copies of application test source.

## Git automation

Only for genuine playbook knowledge/registry changes:
- commit and push to the external project branch;
- reuse/update one PR or create it if none exists;
- request automatic merge to `main` after review/check policy is satisfied;
- confirm merge from the Git host;
- sync the project branch with new `main`.

A duplicate lesson or no knowledge change causes no duplicate commit/push/PR. Never push application source as part of playbook Git operations.

# Bootstrap and Git automation — operational contract

## Trigger behavior

The `.agents/rules/flutter-playbook-bootstrap.md` rule is always-on. At the start of every user turn in an active Flutter application workspace, the agent performs project identification and a lightweight playbook availability/sync check automatically. It does not wait for a special phrase or prompt.

For code-affecting work, it retrieves relevant lessons and follows the full workflow. For informational questions, it answers normally using relevant knowledge where useful and makes no unnecessary code/lesson changes. For unrelated conversation, it does not perform project or playbook writes.

The team member only needs to describe the issue/task and ask naturally to investigate, fix or implement it.

## Bootstrap

1. Check `<home>/dev/flutter-playbook`; clone only if absent. Never overwrite an existing clone.
2. Identify the application root and normalized `origin` remote.
3. Match the remote against `projects/registry.json` (or the registry format configured for the hub).
4. If unregistered, derive a stable ID from the canonical remote; check for collisions; register it on that project's branch. Never guess from a local directory name.
5. Fetch remote refs.
6. Create/open external worktree `<home>/dev/flutter-playbook-worktrees/<project-id>` on `project/<project-id>`.
7. If branch does not exist, create it from `origin/main` and push its initial branch.
8. Verify worktree ownership, correct branch, clean status and no concurrent lock.
9. Fast-forward from `origin/project/<project-id>`, then merge `origin/main` into the project branch to receive other projects' approved lessons.
10. Execute the mandatory 6-step pre-investigation sequence before reading or modifying application code:
    a. Verify project ID and external worktree on `project/<project-id>`.
    b. Read `GLOBAL_AGENT_RULES.md` and `INDEX.md`.
    c. Search the Playbook for existing lessons related to the reported issue, including relevant concepts, symptoms, implementation mechanisms, and previous bug patterns.
    d. Read all relevant lessons completely from beginning to end. If none found, report explicitly before proceeding.
    e. Identify which lessons apply (combining multiple lessons when relevant), explain their relevance, and emit the visible `PLAYBOOK PREFLIGHT` block before inspecting or reading application code.
    f. Only then begin application code inspection, multi-layer root-cause investigation, and minimal implementation.

Never claim a lesson was applied if it was read after implementation started. Never claim pre-investigation reads occurred without execution-order proof in tool calls.

All bootstrap paths and Git operations are outside application repositories. The only application integration is the always-on rule file or its equivalent in the tool's existing rules mechanism.

## Deduplication decision

- Same confirmed root cause and materially same solution: reuse existing lesson; no duplicate or Git write.
- Same cause with a demonstrably better verified fix: update the canonical lesson.
- Same symptom but different confirmed cause: add a separate lesson and cross-link it.
- Unique project-specific context: record only non-sensitive additions and reference the shared lesson.
- Unverified cause/fix or failed test: do not publish as authoritative knowledge.

## Application fix, multi-layer investigation and regression tests

- Fix only what the developer requested.
- Investigate root causes across relevant layers (business logic, state transitions, event ordering, API synchronization, widget lifecycle, shared services/caches, navigation, recovery) using evidence, not assumptions. For state management (BLoC, Riverpod, Provider, etc.), trace actual event transformers, listeners, equality and state emission without guessing.
- Enforce cross-feature behavioral consistency: different features implementing the same business operation (e.g. cart vs recommendations quantity updates) must satisfy the same expected business behavior without forcing identical architecture.
- Add/update the executable regression test in the application's own `test/` directory and run it.
- Select matching tests by category, lifecycle, data-flow and affected dependencies, prioritizing relevant scenarios (rapid repeated actions, out-of-order responses, stale responses overwriting newer state, boundary conditions, cross-feature parity).
- Do not automatically push application source code. Its Git workflow remains the application's existing policy.

## Playbook commit, push and manual PR ownership

Only when a lesson is genuinely new or materially improved, or registry/project metadata must change:
1. Stage only external playbook files.
2. Commit and push to `project/<project-id>`.
3. Do not create a new Pull Request, update an existing Pull Request, request auto-merge, or merge any Pull Request automatically.
4. **The user owns the entire PR creation and merge process** and performs those actions manually.
5. Never bypass required checks/reviews, use `--admin`, force-push or push directly to `main`.
6. After a verified push, report the project branch name, commit hash, remote push verification, changed files, and lesson status, and state: `"Playbook changes have been pushed to project/<project-id>. PR creation and merging are intentionally left to the user."`
7. After the user merges into `main`, synchronize the project branch with `origin/main` safely on subsequent turns.

No knowledge change means no commit or push.

## Concurrency and privacy

Use separate worktrees for different project branches. If a worktree is dirty or locked, do not overwrite it. Never force-push. A branch is not a confidentiality boundary; do not publish customer PII, secrets, private endpoints or proprietary business rules.

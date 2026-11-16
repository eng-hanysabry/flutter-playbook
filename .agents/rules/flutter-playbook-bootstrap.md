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

## B. How to interpret every user prompt

- The playbook workflow is implicit and mandatory; do not ask the user to say "follow the playbook", "remember this", or use a prepared issue prompt.
- If the user reports a problem and asks to fix/solve/investigate it, perform the full diagnosis, fix, regression-test, duplicate-check and lesson-capture workflow.
- If the user requests a new feature, screen, refactor, test or build change, inspect applicable lessons before implementation and run the relevant existing regression tests. Capture a lesson only if the work reveals a genuinely reusable failure/prevention pattern.
- If the user asks an informational question about the active project, use relevant lessons to answer where helpful, but do not modify code, create tests, or write lessons unless requested or a bug is actually fixed.
- For unrelated conversation that does not concern the active project, answer normally. Do not perform application or playbook writes.
- Never require a team member to describe the playbook workflow in their prompt.

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

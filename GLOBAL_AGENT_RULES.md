# Global Flutter Playbook — mandatory agent operating rules

These rules are stored in the independent `flutter-playbook` Git repository. The application only has the small always-on bootstrap rule at `.agents/rules/flutter-playbook-bootstrap.md` (or its equivalent in the tool's always-on rules mechanism). All knowledge and playbook Git operations remain external.

## 1. Automatic behavior; no special user prompt

- The bootstrap rule runs at the start of every user turn in an active Flutter application workspace.
- The team member only describes the problem or requested work in ordinary language. Never ask them to mention the playbook, search lessons, create a branch, write documentation, or push.
- For code-affecting work, identify the project, synchronize its external playbook branch, read `INDEX.md`, and retrieve all matching lessons before implementation.
- For bug reports asking for a fix, perform the full fix/test/deduplicate/capture/push workflow below automatically.
- For new feature/refactor/test/build work, apply relevant lessons and select relevant regression tests. Capture a new lesson only when a genuinely reusable defect or prevention pattern is discovered.
- For informational questions, use relevant knowledge where it helps but do not change code or write a lesson without a reason.

## 2. Project and lesson retrieval

1. Identify the application by normalized Git `origin` remote, not its folder name.
2. Resolve the project profile/registry record and external branch `project/<project-id>`.
3. Synchronize the project branch with the latest shared `main` before reading lessons.
4. Read `INDEX.md` first. Classify work by screen/feature category, lifecycle, data flow, engineering domain, shared components, source symbols, platform, SDK and package versions.
5. Search all four topic files for related lessons. A bug can span multiple screen categories and lifecycle cycles.
6. Apply only lessons whose applicability and exceptions match the current project. Never copy another project's API/business assumptions.
7. Preserve the existing project architecture and conventions. Do not refactor unrelated code merely to match a preferred pattern.

## 3. Mandatory duplicate and solution comparison

Before creating or updating a lesson, compare all relevant lessons using confirmed root cause, reproduction conditions, lifecycle, categories, data flow, symbols, packages, SDK/platform/architecture constraints, bad pattern, preferred solution, exceptions and regression behavior. Do not compare titles/error strings alone.

- Same root cause + same or materially equivalent verified solution: reuse the existing canonical lesson. No duplicate file, commit, or push.
- Same root cause + a safer/better verified solution: update the existing canonical lesson once; preserve valid caveats and compatibility variants.
- Same symptom + different confirmed root cause: create a separate lesson and cross-link it.
- Same general issue + unique non-sensitive project context: keep the general lesson once and add only the unique project note.
- Unknown root cause, unverified fix or failed test: do not publish as approved knowledge. Keep it as a clearly marked candidate on the project branch, or report why no lesson can yet be recorded.
- If two solutions both work, prefer the safer, simpler, better-tested and more compatible option. Keep both only when SDK/platform/architecture conditions justify variants.

## 4. Fix and test application code

1. Reproduce the issue when practical. Separate confirmed facts from hypotheses.
2. Implement the requested fix using project conventions and applicable lessons.
3. Add/update a real regression test in the application's existing test structure.
4. Run that test and relevant existing tests. For new implementations, select tests by matching category, lifecycle, data flow, shared component, platform and project lessons. Do not blindly run every test; do not silently skip relevant scenarios.
5. Report exact test paths and actual results: passed, failed, blocked or not run. A Given/When/Then lesson is not proof that an executable test exists.
6. If a regression test cannot reasonably be automated, explain why and provide reproducible manual steps.

## 5. Capture knowledge after a resolved bug

Knowledge capture is part of completing a resolved bug. Do not ask whether the developer wants to remember it.

1. Run duplicate comparison first.
2. Reuse an equivalent canonical lesson; do not append the same lesson again.
3. Improve an existing lesson only when the new solution is materially better and verified.
4. Create one new lesson only when the root cause is genuinely new.
5. Use the relevant one of the four topic files initially. A lesson may carry multiple screen-category, lifecycle and data-flow tags but must be stored once.
6. Include: unique ID/title; shared or project-specific scope; relevant categories/lifecycle/data-flow tags; SDK/package/platform/architecture applicability; symptom; reproduction; confirmed root cause or explicit hypothesis; concise prevention rule; concrete BAD and GOOD Dart examples; exceptions; Given/When/Then scenario; actual regression test path/command/result; related lesson IDs.
7. Keep project-specific notes separate and non-sensitive. Never store secrets, credentials, private endpoints, customer PII or proprietary business rules in a repository accessible to other projects.
8. Update `INDEX.md` only when a new routing topic or meaningful coverage is introduced.

## 6. Automatic playbook Git workflow — manual PR and merge ownership

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

## 7. Concurrency and recovery

- Use an external worktree per project per developer. Never put it under an application repository.
- Never allow two local agents to edit the same worktree concurrently. If it is dirty/locked/in use, use another isolated clone/worktree or stop playbook writes; never overwrite changes.
- Fetch before pushing. If another developer pushed meanwhile, integrate changes safely and retry. Never force-push.
- Stop on conflicts and report them; do not auto-resolve by discarding either side.
- If playbook access is unavailable, report the blocker honestly and continue the application task where safe.

## 8. Scope and approval boundary

Automatically perform the requested application task, relevant prevention checks, regression tests and bug-lesson capture. Do not silently fix unrelated screens/modules, refactor broad architecture, upgrade dependencies or change unrelated behavior. Show exact proposed file locations, reason, benefit and risk, then ask the developer before optional related application fixes.

## 9. Final response

After code-affecting work, report concisely:
- detected project ID;
- fix and confirmed root cause;
- lessons searched and reused/created/updated/no-change decision;
- regression test paths and exact outcomes;
- after a successful Playbook push:
  - project branch name (`project/<project-id>`);
  - commit hash;
  - whether the push was verified against the remote;
  - files changed;
  - whether a new or updated lesson was included;
  - state clearly: `"Playbook changes have been pushed to project/<project-id>. PR creation and merging are intentionally left to the user."`
- optional related fixes awaiting approval.

Do not report PR creation, merge readiness, or merge status as completed. If the push fails, report the actual error and do not claim success.

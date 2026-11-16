# Flutter Team Playbook — always-on agent workflow

This is a lightweight, Git-backed knowledge workflow for preventing repeated Flutter engineering bugs across projects and team members.

## The developer experience

The team member only describes the issue/task and asks naturally to investigate, fix, solve or implement it. No special prompt, slash command, branch instructions or playbook reminder is required. The always-on rule automatically initializes/synchronizes the external playbook, identifies the project, retrieves relevant lessons, runs the applicable fix/test/knowledge workflow, and manages external playbook Git contributions.

## Repository boundaries

- One independent central Git repository: `flutter-playbook`.
- One persistent external contribution branch per application: `project/<project-id>`.
- One external Git worktree per project per developer, outside all application repositories.
- Application repositories contain only the small always-on bootstrap rule file (or its equivalent in the existing rules mechanism). No lessons, knowledge folders, worktrees, Git submodules, playbook scripts or playbook remotes are added to application Git.
- Actual application source changes and executable regression tests remain in their existing application repositories.
- Bug lessons and reusable prevention knowledge remain in the external playbook.

## Files

- `.agents/rules/flutter-playbook-bootstrap.md`: always-on application integration and bootstrap logic. Replace the placeholder repository URL and install/mark it as always-on in the coding tool.
- `GLOBAL_AGENT_RULES.md`: complete external agent workflow.
- `INDEX.md`: progressive-disclosure routing index.
- `lessons/`: four small initial topic files. A lesson can carry multiple categories/lifecycle tags but is stored once.
- `projects/registry.json`: central remote-to-project identity map.
- `projects/registry/`: one project registry record per project to reduce concurrent merge conflicts.
- `LESSON_TEMPLATE.md`: consistent bug entry structure with concrete bad/good Dart examples and regression scenarios.
- `BOOTSTRAP_AND_GIT_AUTOMATION.md`: bootstrap, push and manual PR ownership contract.
- `PROJECT_BRANCH_AND_DEDUP_WORKFLOW.md`: project branch and duplicate handling.
- `TEAM_MEMBER_USAGE.md`: examples of ordinary issue descriptions; not a required prompt.

## One-time owner setup

1. Create a central Git repository named `flutter-playbook`.
2. Replace `git@github.com:YOUR-ORG/flutter-playbook.git` in `.agents/rules/flutter-playbook-bootstrap.md` with the real central repository URL.
3. Push this package's contents to the central repository's `main` branch.
4. Protect `main` and require the reviews/checks appropriate to your team's risk. Use merge-commit strategy when manually creating and merging PRs from persistent project branches.
5. Add only `.agents/rules/flutter-playbook-bootstrap.md` to each application repository, or install its contents into the application's existing always-on `rules.md`/agent rule. This bootstrap file is the sole permitted application-repository integration. If zero tracked app files is required, install it in each developer's global user rules instead.

## Team member setup

Each developer needs:
- Git access to the central repository;
- authenticated Git credentials for pushing to `project/<project-id>` (PR creation and merging are owned by the user);
- the bootstrap rule loaded as always-on by the coding tool;
- filesystem permission for the external playbook folder.

The agent clones the central repository and creates/opens the project worktree automatically. Team members do not run manual clone/pull/branch commands as part of normal issue work.

## Normal operation

At every user turn in an active Flutter workspace, the rule performs lightweight project identification and playbook sync. On code-affecting tasks, the agent MUST complete the mandatory 6-step pre-investigation sequence (verify project/worktree, read foundation rules, search relevant lessons by concepts/patterns, read lessons completely, explain relevance internally, and only then begin application code inspection/implementation). Never claim pre-investigation reads without tool-call execution order proof, and never claim a lesson was applied if read after implementation started. On a resolved bug, it must compare existing lessons, add/update the real regression test in the application, and capture the lesson in the external playbook. Only genuine knowledge changes create commits and pushes to `project/<project-id>`. Exact duplicates cause no Git write.

The agent never creates, updates, auto-merges, or merges Pull Requests automatically, and never pushes directly to `main`. The user owns the entire PR creation and merge process.

## Safety

If repository access, identity, credentials, worktree cleanliness or branch synchronization is blocked, report it and never pretend it succeeded. Continue application work where safe. Never discard changes, force-push or merge conflicts automatically. A project branch is not a security boundary: do not store secrets, credentials, private endpoints, customer PII or proprietary business logic in a hub readable by all projects.

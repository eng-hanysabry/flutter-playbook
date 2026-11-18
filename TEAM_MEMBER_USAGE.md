# Team member usage — no special prompt required

The team member does not need a workflow prompt, slash command, checklist, or instruction to use the playbook.

Just describe the issue or requested implementation naturally. Examples:

- "The orders list loads the same page twice after pull-to-refresh. Fix it."
- "The app crashes if I leave the profile screen while the API request is running. Investigate and solve it."
- "The login screen sometimes returns to the previous user after token refresh. Find the root cause and fix it."
- "Add a search/filter field to the products list."

The always-on `.agents/rules/flutter-playbook-bootstrap.md` rule automatically handles project detection, central playbook availability, branch synchronization, the mandatory 6-step pre-investigation knowledge retrieval gate (verifying worktree, reading foundation rules, searching by concepts and bug patterns, reading full lessons, combining multiple relevant lessons, emitting the visible `PLAYBOOK PREFLIGHT`, and explaining relevance before touching code), evidence-based multi-layer root-cause investigation, cross-feature behavioral consistency, scenario-based regression-test selection, duplicate comparison, knowledge capture, and external Git push to `project/<project-id>`. Pull Request creation and merging into `main` are intentionally owned by the user.

The agent should not ask the team member to say "follow the playbook" or "remember this bug". Knowledge capture after a resolved bug is mandatory. Approval is required only for optional unrelated application fixes or when a repository policy requires human review.

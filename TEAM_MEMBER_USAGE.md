# Team member usage — no special prompt required

The team member does not need a workflow prompt, slash command, checklist, or instruction to use the playbook.

Just describe the issue or requested implementation naturally. Examples:

- "The orders list loads the same page twice after pull-to-refresh. Fix it."
- "The app crashes if I leave the profile screen while the API request is running. Investigate and solve it."
- "The login screen sometimes returns to the previous user after token refresh. Find the root cause and fix it."
- "Add a search/filter field to the products list."

The always-on `.agents/rules/flutter-playbook-bootstrap.md` rule automatically handles project detection, central playbook availability, branch synchronization, relevant lesson retrieval, regression-test selection, duplicate comparison, knowledge capture, external Git push, PR and auto-merge request.

The agent should not ask the team member to say "follow the playbook" or "remember this bug". Knowledge capture after a resolved bug is mandatory. Approval is required only for optional unrelated application fixes or when a repository policy requires human review.

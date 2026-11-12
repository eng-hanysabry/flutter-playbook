# Project registry records

The registry maps normalized application Git remotes to stable project IDs. The top-level `projects/registry.json` provides an initial/legacy summary; individual records in this directory are preferred to reduce merge conflicts when different projects are onboarded concurrently.

Create one file per project, for example `projects/registry/project-a.json`:

```json
{
  "project_id": "project-a",
  "display_name": "Project A",
  "git_remotes": [
    "github.com:company/project-a.git",
    "https://github.com/company/project-a.git"
  ],
  "playbook_branch": "project/project-a",
  "profile": {
    "stack": ["flutter", "dart"],
    "architecture": "to-be-assessed",
    "flutter_version": "unknown",
    "dart_version": "unknown"
  }
}
```

Normalize SSH/HTTPS forms to the same host/owner/repository identity. Use a stable project ID derived from the canonical remote, not the local folder name. Check collisions before creating it. Registry records contain no secrets or credentials.

# Roadmap and Limits

nGitDB V1 is intentionally narrow. The goal is a dependable structured write layer for reviewable Git-backed JSON workflows, not a general-purpose database or content platform.

## Current Limits

- JSON documents only
- resource paths must use `<collection>/<id>`
- one file per resource
- no query API
- no direct default-branch writes
- no multi-file transaction support
- no conflict-resolution UI
- no hosted service layer

## Current Implementation Notes

The current package implements local and GitHub-backed workflow primitives.

Local mode:

- reads JSON files from the configured repository root
- stages patched documents in memory
- writes commit artifacts under `.ngitdb/sessions/<branch-name>/`
- returns pull request draft metadata

GitHub mode:

- reads JSON files through the GitHub contents API
- creates or resumes a session branch from the base branch
- creates one Git commit for staged resource updates
- advances the session branch with a non-force ref update
- creates or updates a pull request for the session branch

## V1 Direction

The V1 direction remains:

- GitHub-backed JSON resources
- branch-based session isolation
- deterministic patch application
- schema validation before commit
- field ownership enforcement
- review through pull requests
- explicit typed errors

## Deferred Ideas

These are intentionally deferred until the core workflow is proven useful:

- query layer
- multi-repository support
- multi-file transactions
- conflict-resolution UI
- GitHub App packaging
- hosted control plane
- plugin system

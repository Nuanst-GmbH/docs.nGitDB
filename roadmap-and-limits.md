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

The current package implements the workflow primitives using a local repository root:

- reads JSON files from the configured repository root
- stages patched documents in memory
- writes commit artifacts under `.ngitdb/sessions/<branch-name>/`
- returns pull request draft metadata

Direct GitHub commit creation and direct GitHub pull request creation are product-direction items, not behavior provided by the current local workflow implementation.

## V1 Direction

The V1 product direction is:

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

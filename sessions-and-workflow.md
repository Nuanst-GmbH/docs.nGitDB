# Sessions and Review Workflow

nGitDB makes write operations explicit by requiring an active session. A session represents an isolated branch-backed unit of work.

## Start a Session

```ts
const session = await db.startSession("acme-gmbh");
```

Session branch names are deterministic and human-readable:

```txt
session/acme-gmbh-2026-04-21
```

Session keys are normalized by lowercasing, replacing unsafe characters with hyphens, and appending the session date.

## Session State

```ts
const session = db.getCurrentSession();
```

Session state contains:

- `key`: original session key
- `branchName`: generated branch name
- `startedAt`: ISO timestamp

## Writes Require a Session

`patch(...)`, `commit(...)`, and `createPullRequest(...)` require an active session.

Calling them outside a session throws `SessionMisuseError`.

## Commit Behavior

With `backend: { type: "github" }`, `commit(message)` validates staged documents, creates one GitHub commit containing all staged resource files, and advances the session branch with a non-force ref update.

With the default local backend, `commit(message)` validates staged documents and writes session artifacts under:

```txt
.ngitdb/sessions/<branch-name>/
```

The default branch files are left unchanged until the review branch is merged.

## Pull Requests

With `backend: { type: "github" }`, `createPullRequest(metadata)` creates a pull request for the session branch. If an open pull request already exists for the same session branch and base branch, nGitDB updates its title and body and returns that pull request.

With the local backend, `createPullRequest(metadata)` returns a pull request draft object:

```ts
{
  title: "Enrich acme-gmbh company profile",
  body: "",
  headBranch: "session/acme-gmbh-2026-04-21",
  baseBranch: "main",
  committedResources: ["companies/acme-gmbh"]
}
```

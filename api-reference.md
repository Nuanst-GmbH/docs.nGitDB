# API Reference

## `createGitDB(config)`

Creates a `GitDB` instance.

```ts
const db = createGitDB(config);
```

See [Configuration](configuration.md) for the full config shape.

## `db.read(resourcePath)`

Reads and parses the JSON document for a resource.

```ts
const document = await db.read("companies/acme-gmbh");
```

Returns:

```ts
Promise<JsonObject>
```

May throw:

- `InvalidResourcePathError`
- `ResourceDefinitionError`
- `ResourceNotFoundError`

## `db.startSession(sessionKey)`

Starts a new isolated session and resets staged documents for the `GitDB` instance. In GitHub mode this creates or resumes the session branch.

```ts
const session = await db.startSession("acme-gmbh");
```

Returns:

```ts
Promise<SessionState>
```

Example return value:

```ts
{
  key: "acme-gmbh",
  branchName: "session/acme-gmbh-2026-04-21",
  startedAt: "2026-04-21T08:15:00.000Z",
}
```

## `db.getCurrentSession()`

Returns the active session, or `undefined` if no session has been started.

```ts
const session = db.getCurrentSession();
```

## `db.patch(resourcePath, patchDocument)`

Applies a deterministic partial update to a staged resource document.

```ts
const nextDocument = await db.patch("companies/acme-gmbh", {
  "machine.summary": "Updated summary",
});
```

Returns:

```ts
Promise<JsonObject>
```

May throw:

- `SessionMisuseError`
- `InvalidResourcePathError`
- `ResourceDefinitionError`
- `ResourceNotFoundError`
- `OwnershipViolationError`

## `db.commit(message)`

Validates staged documents and commits staged changes for the current session. In GitHub mode this creates one Git commit on the session branch. In local mode this writes commit artifacts.

```ts
const commit = await db.commit("Enrich company profile for acme-gmbh");
```

Returns:

```ts
Promise<CommitResult>
```

Example return value:

```ts
{
  branchName: "session/acme-gmbh-2026-04-21",
  message: "Enrich company profile for acme-gmbh",
  committedResources: ["companies/acme-gmbh"],
  commitFilePath: "/repo/.ngitdb/sessions/session/acme-gmbh-2026-04-21/commit-0001.json",
  commitSha: "abc123",
  commitUrl: "https://api.github.com/repos/owner/repo/git/commits/abc123",
  htmlUrl: "https://github.com/owner/repo/commit/abc123",
}
```

May throw:

- `SessionMisuseError`
- `ValidationFailureError`
- `GitHubApiError`
- `GitHubConflictError`

## `db.createPullRequest(metadata)`

Creates or updates a pull request from committed session changes in GitHub mode. In local mode it returns pull request draft metadata.

```ts
const draft = await db.createPullRequest({
  title: "Enrich acme-gmbh company profile",
  body: "Updates machine-owned enrichment fields.",
});
```

Returns:

```ts
Promise<PullRequestDraft>
```

May throw:

- `SessionMisuseError`
- `GitHubApiError`

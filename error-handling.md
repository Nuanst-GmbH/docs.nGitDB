# Error Handling

nGitDB exposes explicit error classes so application code can handle safety failures without string matching.

```ts
import {
  GitDBError,
  OwnershipViolationError,
  SessionMisuseError,
  ValidationFailureError,
} from "@nuanst-one/ngitdb";
```

## Base Error

All library errors extend:

```ts
GitDBError
```

## Resource Errors

`InvalidResourcePathError` is thrown when a resource path is not in `<collection>/<id>` format.

`ResourceDefinitionError` is thrown when the collection is not configured.

`ResourceNotFoundError` is thrown when the resolved JSON file does not exist.

## Session Errors

`SessionMisuseError` is thrown when a workflow method is called at the wrong time.

Examples:

- calling `patch(...)` before `startSession(...)`
- calling `commit(...)` without staged changes
- calling `createPullRequest(...)` before a commit

## Ownership Errors

`OwnershipViolationError` is thrown when a patch targets a field that is not machine-owned.

```ts
try {
  await db.patch("companies/acme-gmbh", {
    legalName: "New Name GmbH",
  });
} catch (error) {
  if (error instanceof OwnershipViolationError) {
    console.log(error.resourcePath);
    console.log(error.fieldPath);
    console.log(error.owner);
  }
}
```

## Validation Errors

`ValidationFailureError` is thrown when a resource validator returns issues during commit.

```ts
try {
  await db.commit("Commit staged changes");
} catch (error) {
  if (error instanceof ValidationFailureError) {
    console.log(error.resourcePath);
    console.log(error.issues);
  }
}
```

## Recommended Pattern

```ts
try {
  await db.commit("Update enrichment fields");
} catch (error) {
  if (error instanceof ValidationFailureError) {
    return {
      ok: false,
      reason: "validation_failed",
      issues: error.issues,
    };
  }

  if (error instanceof GitDBError) {
    return {
      ok: false,
      reason: error.name,
      message: error.message,
    };
  }

  throw error;
}
```

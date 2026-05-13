# Patch and Ownership Model

nGitDB uses deterministic patch documents as the primary write primitive. It does not expose a public full-file replacement API.

## Patch Shape

A patch is a map of dot-separated field paths to JSON values.

```ts
await db.patch("companies/acme-gmbh", {
  "machine.summary": "Industrial supplier with operations in Berlin",
  "machine.lastEnrichedAt": "2026-04-21",
});
```

Patch entries are applied in sorted field-path order so the same base document plus the same patch produces the same result.

## Nested Fields

Dot-separated paths update nested object fields.

```ts
await db.patch("companies/acme-gmbh", {
  "machine.review.score": 92,
});
```

If intermediate objects do not exist, nGitDB creates them. If a patch tries to write beneath a scalar value, the patch fails.

## Ownership States

Writable resources may define field ownership:

```ts
ownership: {
  legalName: "human-owned",
  machine: "machine-owned",
}
```

Supported states:

- `human-owned`
- `machine-owned`

Machine writes are only allowed against machine-owned fields.

## Ownership Lookup

nGitDB checks the most specific field path first and then walks upward.

For this ownership map:

```ts
ownership: {
  legalName: "human-owned",
  machine: "machine-owned",
}
```

these patches are allowed:

```ts
{
  "machine.summary": "Updated summary",
  "machine.lastEnrichedAt": "2026-04-21"
}
```

this patch is rejected:

```ts
{
  legalName: "New Name GmbH"
}
```

## Wildcard Ownership

The ownership map can use `*` as a fallback owner.

```ts
ownership: {
  "*": "human-owned",
  machine: "machine-owned",
}
```

This makes unlisted fields human-owned by default while still allowing writes beneath `machine`.

## Validation

Validation runs during `commit(message)`, not during `patch(...)`.

```ts
validate: (document) => {
  const issues: string[] = [];

  if (typeof document.legalName !== "string") {
    issues.push("legalName must be a string");
  }

  return issues;
}
```

If validation returns issues, commit creation is blocked with `ValidationFailureError`.

# Examples

## Company Enrichment Workflow

This example updates only machine-owned enrichment fields while preserving human-owned company identity fields.

```ts
import { createGitDB } from "@nuanst-one/ngitdb";

const db = createGitDB({
  repositoryRoot: process.cwd(),
  resources: {
    companies: {
      fileName: "company.json",
      ownership: {
        legalName: "human-owned",
        machine: "machine-owned",
      },
      validate: (document) => {
        const issues: string[] = [];

        if (typeof document.legalName !== "string" || document.legalName.length === 0) {
          issues.push("legalName must be a non-empty string");
        }

        if (
          document.machine === null ||
          typeof document.machine !== "object" ||
          Array.isArray(document.machine)
        ) {
          issues.push("machine must be an object");
        }

        return issues;
      },
    },
  },
});

await db.startSession("companies/acme-gmbh");

await db.patch("companies/acme-gmbh", {
  "machine.summary": "Industrial supplier with operations in Berlin",
  "machine.lastEnrichedAt": "2026-04-21",
});

await db.commit("Enrich company profile for acme-gmbh");

const pullRequestDraft = await db.createPullRequest({
  title: "Enrich acme-gmbh company profile",
  body: "Updates machine-owned enrichment fields.",
});
```

## Rejecting Human-Owned Writes

```ts
import { OwnershipViolationError } from "@nuanst-one/ngitdb";

await db.startSession("rename-acme");

try {
  await db.patch("companies/acme-gmbh", {
    legalName: "New Name GmbH",
  });
} catch (error) {
  if (error instanceof OwnershipViolationError) {
    console.log(`Blocked ${error.fieldPath}: ${error.owner}`);
  }
}
```

## Validating Before Commit

```ts
import { ValidationFailureError } from "@nuanst-one/ngitdb";

await db.startSession("invalid-enrichment");

await db.patch("companies/acme-gmbh", {
  machine: "invalid",
});

try {
  await db.commit("Try invalid update");
} catch (error) {
  if (error instanceof ValidationFailureError) {
    console.log(error.issues);
  }
}
```

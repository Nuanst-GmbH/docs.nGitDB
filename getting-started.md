# Getting Started

This guide shows the smallest useful nGitDB setup: one JSON resource type, one repository root, field ownership, validation, a session, a patch, a commit, and a pull request.

## Install

```bash
npm install ngitdb
```

For local development from this repository:

```bash
npm install
npm run build
npm test
```

## Repository Shape

nGitDB resolves resource paths by convention. A resource path has this shape:

```txt
<collection>/<id>
```

By default, that maps to:

```txt
data/<collection>/<id>/<fileName>
```

Example:

```txt
companies/acme-gmbh -> data/companies/acme-gmbh/company.json
```

## Example JSON Document

```json
{
  "legalName": "ACME GmbH",
  "machine": {
    "summary": "Legacy summary",
    "lastEnrichedAt": "2026-04-20"
  }
}
```

## Create a Database Client

```ts
import { createGitDB } from "ngitdb";

const db = createGitDB({
  repositoryRoot: process.cwd(),
  backend: { type: "github" },
  baseBranch: "main",
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
```

In GitHub Actions, `backend: { type: "github" }` uses `GITHUB_REPOSITORY` and `GITHUB_TOKEN` unless you pass `owner`, `repo`, and `token` explicitly.

## Read a Resource

```ts
const company = await db.read("companies/acme-gmbh");
```

## Patch a Machine-Owned Field

Writes require an active session.

```ts
await db.startSession("acme-gmbh");

await db.patch("companies/acme-gmbh", {
  "machine.summary": "Industrial supplier with operations in Berlin",
  "machine.lastEnrichedAt": "2026-04-21",
});
```

## Commit and Create a Pull Request

```ts
const commit = await db.commit("Enrich company profile for acme-gmbh");

const pullRequest = await db.createPullRequest({
  title: "Enrich acme-gmbh company profile",
});
```

With the GitHub backend, this creates a commit on the session branch and creates or updates a pull request. With the default local backend, nGitDB writes session commit artifacts under `.ngitdb/sessions/<branch-name>/` and returns pull request draft metadata.

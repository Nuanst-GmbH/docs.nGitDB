# nGitDB

nGitDB is a TypeScript library for teams that keep structured JSON data in Git repositories and need safe, reviewable updates.

It gives internal tools a small workflow-aware API for reading resources, applying deterministic partial updates, validating staged changes, protecting human-owned fields, and preparing reviewable branch-based changes.

## What It Solves

Many internal tools eventually treat a Git repository as a source of truth for structured data. Raw file updates are easy to start with, but they become risky when humans and automation both edit the same documents.

nGitDB focuses on the safety layer:

- resource paths instead of raw file paths
- patch-based updates instead of full-file replacement
- field ownership rules to protect human-authored data
- schema validation before commit artifacts are created
- explicit sessions that isolate changes by branch name
- pull request drafts that make review the promotion path

## Current Package Status

The current implementation provides the V1 library surface with local and GitHub backends. Local mode writes committed session artifacts under `.ngitdb/sessions/...` and returns pull request draft metadata. GitHub mode creates or resumes session branches, creates commits with the Git database API, and creates or updates pull requests.

## Quick Example

```ts
import { createGitDB } from "ngitdb";

const db = createGitDB({
  repositoryRoot: "/path/to/repo",
  resources: {
    companies: {
      fileName: "company.json",
      ownership: {
        legalName: "human-owned",
        machine: "machine-owned",
      },
      validate: (document) => {
        const issues: string[] = [];

        if (typeof document.legalName !== "string") {
          issues.push("legalName must be a string");
        }

        return issues;
      },
    },
  },
});

await db.startSession("acme-gmbh");

await db.patch("companies/acme-gmbh", {
  "machine.summary": "Industrial supplier with operations in Berlin",
  "machine.lastEnrichedAt": "2026-04-21",
});

await db.commit("Enrich company profile for acme-gmbh");

const pullRequest = await db.createPullRequest({
  title: "Enrich acme-gmbh company profile",
});
```

## Wiki

- [Getting Started](getting-started.md)
- [Configuration](configuration.md)
- [GitHub Actions](github-actions.md)
- [Resource Model](resource-model.md)
- [Patch and Ownership Model](patch-and-ownership.md)
- [Sessions and Review Workflow](sessions-and-workflow.md)
- [API Reference](api-reference.md)
- [Error Handling](error-handling.md)
- [Examples](examples.md)
- [Prompt Recipes](prompt-recipes.md)
- [Roadmap and Limits](roadmap-and-limits.md)
- [Publishing on GitHub Pages](publishing.md)

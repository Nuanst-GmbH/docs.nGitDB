# Configuration

nGitDB is configured through `createGitDB(config)`.

```ts
import { createGitDB } from "ngitdb";

const db = createGitDB({
  repositoryRoot: "/path/to/repo",
  backend: { type: "github" },
  baseBranch: "main",
  resourceRoot: "data",
  resources: {
    companies: {
      fileName: "company.json",
      ownership: {
        legalName: "human-owned",
        machine: "machine-owned",
      },
      validate: (document) => [],
    },
  },
});
```

## `backend`

Optional backend selection.

Default local backend:

```ts
backend: { type: "local" }
```

Local mode writes session artifacts under `.ngitdb/sessions/...` and does not call GitHub.

GitHub backend:

```ts
backend: {
  type: "github",
  owner: "nuanst-gmbh",
  repo: "nGitDB",
  token: process.env.GITHUB_TOKEN
}
```

When running in GitHub Actions, `owner` and `repo` can be omitted if `GITHUB_REPOSITORY` is set, and `token` can be omitted if `GITHUB_TOKEN` is set.

GitHub Actions workflows need:

```yaml
permissions:
  contents: write
  pull-requests: write
```

Workflow steps that execute nGitDB should expose the Actions token as an environment variable:

```yaml
env:
  GITHUB_TOKEN: ${{ github.token }}
```

See [GitHub Actions](github-actions.md) for a complete minimal workflow.

## `repositoryRoot`

Absolute or process-relative path to the repository root used by the library.

```ts
repositoryRoot: "/path/to/repo"
```

## `baseBranch`

Optional base branch name used when creating pull request draft metadata.

Default:

```txt
main
```

## `resourceRoot`

Optional directory where resource collections live.

Default:

```txt
data
```

With the default resource root, `companies/acme-gmbh` resolves to:

```txt
data/companies/acme-gmbh/company.json
```

## `resources`

Resource definitions keyed by collection name.

```ts
resources: {
  companies: {
    fileName: "company.json",
    ownership: {
      legalName: "human-owned",
      machine: "machine-owned",
    },
    validate: (document) => [],
  },
}
```

Each resource definition supports:

- `fileName`: JSON file name inside each resource directory
- `ownership`: optional field ownership map
- `validate`: optional validator that returns issue strings

## `currentDate`

Optional date override used for deterministic session branch names and tests.

```ts
currentDate: new Date("2026-04-21T08:15:00.000Z")
```

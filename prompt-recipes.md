# Prompt Recipes

These prompts help consumers integrate nGitDB with AI coding assistants. They are written to steer the assistant toward nGitDB's safety model: resource paths, sessions, `db.patch(...)`, ownership rules, validators, and tests.

## Codex Plugin

nGitDB also ships these workflows as Codex skills in the bundled plugin:

- `ngitdb-starter`: start a new TypeScript project with nGitDB
- `ngitdb-integrator`: add nGitDB to an existing JSON workflow
- `ngitdb-reviewer`: review an nGitDB integration for safety gaps

The plugin source is included in the npm package under:

```txt
plugins/ngitdb
```

## Add nGitDB to a New Project

Use this when the project does not yet have an established JSON resource model.

```txt
You are setting up nGitDB in this TypeScript project.

Goal:
- Create an initial Git-backed JSON resource model.
- Configure nGitDB with resource paths in the form <collection>/<id>.
- Separate human-owned and machine-owned fields from the beginning.
- Add validators for writable resources.
- Add a small workflow that reads, starts a session, patches machine-owned fields, commits, and creates a pull request draft.
- Add tests for read, patch, ownership rejection, validation failure, and session misuse.

Please:
1. Inspect the repository structure first.
2. Propose a minimal data layout under data/<collection>/<id>/<fileName>.
3. Create the nGitDB configuration using createGitDB(...).
4. Use db.patch(...) for writes. Do not replace whole JSON files directly.
5. Keep the default branch files unchanged during the session workflow.
```

## Add nGitDB to an Existing Project

Use this when the app already stores JSON files in Git.

```txt
You are integrating nGitDB into this existing TypeScript project.

Goal:
- Preserve the existing JSON file structure where possible.
- Configure nGitDB resource definitions for the existing collections.
- Replace unsafe full-file JSON updates with db.patch(...).
- Add ownership rules so machine updates cannot overwrite human-owned fields.
- Add validators that block invalid staged documents before commit.
- Add focused tests for the safety workflow.

Please:
1. Inspect the existing JSON files and write paths.
2. Identify candidate resource paths in the form <collection>/<id>.
3. Create or update the createGitDB configuration.
4. Migrate one representative write path to startSession -> patch -> commit -> createPullRequest.
5. Add tests for read, machine-owned patch success, human-owned patch rejection, validation failure, and patch outside session.
6. Avoid unrelated refactors.
```

## Add the nGitDB GitHub Action to a Python Workflow

Use this when Python generates derived data and GitHub Actions should publish reviewable JSON changes.

```txt
Add the nGitDB GitHub Action to this repository's CI workflow.

Goal:
- Keep the existing Python data generation step.
- Make Python write an nGitDB batch JSON file with resources: [{ resourcePath, patch }].
- Add a GitHub Actions step using nuanst-gmbh/nGitDB@v0.
- Pass batch-file, session-key, commit-message, pr-title, pr-body, and resource-config.
- Grant only contents: write and pull-requests: write.
- Patch only machine-owned fields. Do not replace full JSON files.

Please:
1. Inspect the current Python workflow and JSON file layout.
2. Identify resource paths in <collection>/<id> format.
3. Add or update the Python batch writer.
4. Add the nGitDB Action step after Python generation.
5. Include ownership rules in resource-config.
6. Add tests or a fixture proving the batch contains only machine-owned patches.
```

## Generate an nGitDB Batch File from Python Output

Use this when Python already creates enriched records or merged JSON.

```txt
Create a Python batch writer for the nGitDB GitHub Action.

Output schema:
{
  "resources": [
    {
      "resourcePath": "<collection>/<id>",
      "patch": {
        "<machine-owned-field-path>": <json-value>
      }
    }
  ]
}

Rules:
- Include machine-owned generated fields only.
- Exclude human-owned identity, legal, approval, and manually curated fields.
- Use resource paths, not raw file paths.
- Fail if a record has no stable id.
- Do not write full resource documents.
```

## Model a Resource Collection

Use this to turn a JSON shape into an nGitDB resource definition.

```txt
Inspect this JSON document and design an nGitDB resource definition for it.

Please provide:
- the collection name
- the fileName
- the resource path convention
- an ownership map with human-owned and machine-owned fields
- a validator function that returns actionable issue strings
- example allowed and rejected patches

Assume machine writes should only update enrichment, analysis, or generated metadata fields.
```

## Add an AI Enrichment Workflow

Use this when automation writes derived data into existing documents.

```txt
Create an AI enrichment workflow using nGitDB.

Rules:
- AI-generated output must only be written under machine-owned fields.
- Human-owned fields must be preserved.
- Writes must happen inside an explicit session.
- Use db.patch(...), not full-file replacement.
- Commit must validate staged documents.
- Pull request draft metadata must explain what changed.

Add tests proving that human-owned fields cannot be overwritten.
```

## Review an nGitDB Integration

Use this to audit an existing integration.

```txt
Review this nGitDB integration for safety issues.

Look for:
- full-file JSON replacement instead of db.patch(...)
- GitHub Action batches containing full documents instead of resources: [{ resourcePath, patch }]
- patch calls outside sessions
- missing or overly broad ownership rules
- machine-owned fields that should be human-owned
- missing validators
- validation that returns vague errors
- workflows missing contents: write or pull-requests: write
- tests missing ownership, validation, and session misuse cases
- pull request drafts without clear title or body

Report findings with file and line references, ordered by severity.
```

## Bad Prompt

```txt
Update the company JSON file with the new AI summary.
```

This can push an assistant toward direct file replacement.

## Better Prompt

```txt
Generate an nGitDB Action batch file that patches only the machine-owned summary field for companies/acme-gmbh.
Use resources: [{ resourcePath, patch }], do not overwrite human-owned fields, and do not replace the whole JSON file.
```

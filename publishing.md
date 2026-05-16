# Publishing

## npm Package

The npm package is published as `@nuanst-one/ngitdb`. The build emits declarations and an obfuscated/minified JavaScript runtime:

```bash
npm run build
npm run build:action
npm pack --dry-run
```

`npm run build`:

- cleans `dist`
- emits `.d.ts` files from TypeScript
- bundles `src/index.ts` to `dist/index.js`
- minifies and obfuscates the runtime bundle
- imports the built package and runs a local nGitDB smoke test

`npm run build:action` bundles and minifies the GitHub Action entrypoint at `dist/action.cjs`, then verifies it can be loaded by Node.

Publish through `.github/workflows/npm-publish.yml`. The workflow requires repository secret `NPM_TOKEN` and publishes the public package from this private repository without npm provenance.

Publishing is release-driven and tag-versioned. Create a semver tag with a leading `v`, then publish a GitHub release from it:

```bash
git tag v0.1.1
git push origin v0.1.1
gh release create v0.1.1 --title "v0.1.1" --notes "Release v0.1.1"
```

The workflow sets `package.json` and `package-lock.json` to the tag version at publish time with `npm version --no-git-tag-version`. Regular PRs do not need package version bumps unless the committed baseline version should change for documentation or local development.

Npm versions are immutable. If `@nuanst-one/ngitdb@<tag-version>` already exists, the workflow fails before building.

Npm provenance requires the GitHub source repository to be public. If this repository becomes public later, restore `id-token: write` in the workflow permissions and publish with `npm publish --provenance --access public`.

Obfuscation raises the cost of casual inspection. It does not make reverse engineering impossible.

## GitHub Pages

This documentation set is ready to publish from `docs/wiki` as the public site root.

## GitHub Pages Settings

GitHub Pages branch-source publishing supports only `/` or `/docs` as the source folder. It does not support `/docs/wiki` directly.

To publish only `docs/wiki`, configure Pages to use GitHub Actions. This repository includes `.github/workflows/pages.yml`, which builds `docs/wiki` with Jekyll and deploys the generated site.

In the GitHub repository:

1. Open `Settings`.
2. Open `Pages`.
3. Under `Build and deployment`, set `Source` to `GitHub Actions`.
4. Push the `.github/workflows/pages.yml` workflow.

The workflow builds from `docs/wiki`, so GitHub Pages will use `docs/wiki/index.md` as the site homepage and `docs/wiki/_config.yml` for the site title, description, Markdown settings, and theme.

## Site Structure

The public site entry point is:

```txt
docs/wiki/index.md
```

Wiki-style pages live next to it:

```txt
docs/wiki/getting-started.md
docs/wiki/configuration.md
docs/wiki/resource-model.md
docs/wiki/patch-and-ownership.md
docs/wiki/sessions-and-workflow.md
docs/wiki/api-reference.md
docs/wiki/error-handling.md
docs/wiki/examples.md
docs/wiki/roadmap-and-limits.md
```

## Local Preview

GitHub renders Markdown automatically after Pages is enabled. A full local Jekyll preview is optional.

For a quick content check, read `docs/wiki/index.md` and follow its relative links. For a closer GitHub Pages preview, install Jekyll locally and serve the `docs/wiki` directory.

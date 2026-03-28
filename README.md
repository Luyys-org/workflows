# Reusable GitHub Workflows

## Docker image builder
- Workflow: [.github/workflows/docker-image-builder.yml](.github/workflows/docker-image-builder.yml)
- Builds or reuses a language-specific container image in GHCR and returns the image reference plus registry URL.

### Inputs
- `language`: Runtime to package (python, node, java).
- `version`: Runtime version tag (for example 3.12, 22, 21-jdk).

### Outputs
- `image-name`: Fully qualified image reference (registry/repo:tag).
- `repository-url`: GHCR package page for the image.
- `image-exists`: `true` when the image was already present.
- `pushed`: `true` when this run built and pushed the image.

### How to use (reusable call)
```yaml
jobs:
	build-runtime:
		uses: <owner>/<repo>/.github/workflows/docker-image-builder.yml@<ref>
		with:
			language: python
			version: "3.12"
```
Consume outputs with `needs.build-runtime.outputs.image-name` and `needs.build-runtime.outputs.repository-url`.

### How to run manually
Trigger the `Build and Publish Docker Image` workflow in the Actions tab and provide `language` and `version` inputs.

## NPM static bundle builder
- Workflow: [.github/workflows/npm-build.yml](.github/workflows/npm-build.yml)
- Builds npm assets into deployable static files from `./dist` and uploads them as an artifact.

### Inputs
- `node-version`: Node.js version to use.
- `application-name`: Application name used to build the artifact name.

The workflow runs `npm install`, then `npm build`, validates that `./dist` exists, and uploads the directory.

### Outputs
- `build-directory`: Resolved path to generated static files.
- `artifact-name`: Artifact name that contains static build files.
- `artifact-id`: Uploaded artifact identifier.
- `artifact-url`: URL to the uploaded artifact.

Artifact name format:
- `<application-name>-node-<node-version>_<run_id>`

### How to use (reusable call)
```yaml
jobs:
	build-static:
		uses: <owner>/<repo>/.github/workflows/npm-build.yml@<ref>
		with:
			node-version: "20"
			application-name: "frontend"
```
Consume outputs with `needs.build-static.outputs.build-directory`, `needs.build-static.outputs.artifact-name`, and optionally `needs.build-static.outputs.artifact-url`.

## GitHub Pages deploy from npm build artifact
- Workflow: [.github/workflows/deploy-page.yml](.github/workflows/deploy-page.yml)
- Deploys the static artifact produced by `npm-build.yml` to the current repository GitHub Pages site. Must be called within the same workflow run as `npm-build.yml`.

### Inputs
- `artifact-name`: Name of the artifact to deploy from `npm-build.yml` (required).

### How to use (reusable call)
```yaml
jobs:
	build-static:
		uses: <owner>/<repo>/.github/workflows/npm-build.yml@<ref>
		with:
			node-version: "20"
			application-name: "frontend"
	
	deploy-pages:
		needs: build-static
		uses: <owner>/<repo>/.github/workflows/deploy-page.yml@<ref>
		with:
			artifact-name: ${{ needs.build-static.outputs.artifact-name }}
```

### Notes
- It must be called in the same workflow run as `npm-build.yml` to access the artifact.
- Repository Pages must be enabled and configured for GitHub Actions as the source.

## NPM package version bump
- Workflow: [.github/workflows/npm-version-bump.yml](.github/workflows/npm-version-bump.yml)
- Bumps the version field in `package.json` (and `package-lock.json` when present) according to the requested semver update type, commits and pushes the change, then creates a GitHub Release tagged `v<new-version>`.

### Inputs
- `update-type`: Semver increment to apply — `major` (breaking change), `minor` (new feature), or `patch` (bug fix). Required.

### Outputs
- `new-version`: The semver version string after bumping (e.g. `1.3.0`).
- `previous-version`: The semver version string before bumping (e.g. `1.2.0`).

### How to use (reusable call)
```yaml
jobs:
	bump:
		uses: <owner>/<repo>/.github/workflows/npm-version-bump.yml@<ref>
		with:
			update-type: minor
```
Consume outputs with `needs.bump.outputs.new-version` and `needs.bump.outputs.previous-version`.

### Notes
- The workflow commits the updated file(s) as `github-actions[bot]`, pushes directly to the branch it was called on, and creates a GitHub Release tagged `v<new-version>` pointing at that commit.
- `contents: write` permission is required and is declared inside the workflow, so no additional repository settings are needed when using `workflow_call`.
- `update-type` is a free-form string validated at runtime; invalid values fail the job immediately before any file changes are made.


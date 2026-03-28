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
- Deploys the static artifact produced by `npm-build.yml` to the current repository GitHub Pages site.

### Triggers
- Manual only: `workflow_dispatch` with optional overrides.

### Manual inputs
- `build-run-id`: Optional `npm-build.yml` run ID to deploy. If omitted, the latest successful `npm-build.yml` run is used.
- `artifact-name`: Optional artifact name to deploy from the selected run. If omitted, the newest non-expired artifact in that run is used.

### Notes
- The workflow resolves the artifact from the selected npm build run, downloads it, then publishes it using `actions/deploy-pages`.
- Repository Pages must be enabled and configured for GitHub Actions as the source.


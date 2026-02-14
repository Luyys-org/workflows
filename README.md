# Reusable GitHub Workflows

## Docker image builder
- Workflow: [.github/workflows/language-image-builder.yml](.github/workflows/language-image-builder.yml)
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
		uses: <owner>/<repo>/.github/workflows/language-image-builder.yml@<ref>
		with:
			language: python
			version: "3.12"
```
Consume outputs with `needs.build-runtime.outputs.image-name` and `needs.build-runtime.outputs.repository-url`.

### How to run manually
Trigger the `Build and Publish Docker Image` workflow in the Actions tab and provide `language` and `version` inputs.


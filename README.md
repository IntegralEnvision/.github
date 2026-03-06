# Organization Shared GitHub Actions

This repository contains reusable GitHub Actions workflows that can be shared across all repositories within the organization.

## Overview

This `.github` repository serves as a centralized location for:
- **Reusable Workflows**: Common CI/CD patterns that can be used across multiple repositories
- **Organization Profile**: Public-facing organization information and branding

## Available Workflows

### Docker Build CI/CD (`docker-build-cicd.yml`)

A reusable workflow for building and publishing Docker images to any registry supported by Docker login credentials. This workflow handles:
- Docker registry authentication
- Docker image building with multi-platform support
- Pushing the image to the configured registry
- Automatic tagging with the short commit SHA and `latest`

#### Usage

To use this workflow in your repository, create a workflow file (e.g., `.github/workflows/deploy.yml`) with the following content:

```yaml
name: Build and Publish Docker Image

on:
  push:
    branches:
      - main

permissions:
  contents: read
  packages: write

jobs:
  docker:
    uses: IntegralEnvision/.github/.github/workflows/docker-build-cicd.yml@main
    with:
      image_name: "your-app/your-service"
      registry: "ghcr.io"
      dockerfile_path: "Dockerfile"  # Optional, defaults to "Dockerfile"
      platforms: "linux/amd64"       # Optional, defaults to "linux/amd64"
      build_args: |
        BUILD_ARG_1=value1
    secrets:
      REGISTRY_USERNAME: ${{ github.actor }}
      REGISTRY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

#### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `image_name` | Docker image name | `"shinyapps/app-flood"` |
| `registry` | Container registry hostname | `"ghcr.io"` |

#### Optional Inputs

| Input | Description | Default | Example |
|-------|-------------|---------|---------|
| `dockerfile_path` | Path to Dockerfile | `"Dockerfile"` | `"docker/Dockerfile"` |
| `platforms` | Target platforms for build | `"linux/amd64"` | `"linux/amd64,linux/arm64"` |
| `build_args` | Multiline Docker build args | `""` | `"FOO=bar"` |

#### Required Secrets

You must configure the following secrets in your repository or organization:

| Secret | Description |
|--------|-------------|
| `REGISTRY_USERNAME` | Username for the target registry |
| `REGISTRY_PASSWORD` | Password or token for the target registry |

#### Outputs

The reusable workflow exposes these outputs:

| Output | Description |
|--------|-------------|
| `image_tag` | Short SHA tag, for example `"abc1234"` |
| `image` | Full SHA-tagged image reference |
| `latest_image` | Full `latest` image reference |

#### Generated Tags

The workflow automatically generates the following tags for your Docker image:
- `{COMMIT_SHA:7}`
- `latest`

Example: `abc1234` and `latest`

## Registry Authentication

To use the Docker build workflow, configure credentials that can push to your target registry.

### Example: GitHub Container Registry

For `ghcr.io`, you can typically use:
- `REGISTRY_USERNAME`: `${{ github.actor }}`
- `REGISTRY_PASSWORD`: `${{ secrets.GITHUB_TOKEN }}`

### Configure Repository Secrets

Add the following secrets to your repository (Settings → Secrets and variables → Actions):

- `REGISTRY_USERNAME`: Registry username
- `REGISTRY_PASSWORD`: Registry password or token

## Contributing

When adding new reusable workflows:

1. Create the workflow file in `.github/workflows/`
2. Use `workflow_call` trigger for reusable workflows
3. Define clear inputs and secrets
4. Update this README with usage instructions
5. Test the workflow in a sample repository before merging

## Best Practices

- **Version your workflows**: Use specific tags or branches when calling reusable workflows
- **Minimize secrets**: Only require secrets that are absolutely necessary
- **Provide defaults**: Set sensible defaults for optional inputs
- **Document thoroughly**: Include clear examples and descriptions
- **Test changes**: Always test workflow changes in a development environment first

## Repository Structure

```
.github/
├── workflows/
│   └── docker-build-cicd.yml           # Generic Docker build/publish workflow
├── examples/
│   ├── dma-k8s-docker-build-cicd.yaml  # Legacy example
│   └── docker-build-cicd.yml           # Generic workflow example
├── profile/
│   ├── README.md                       # Organization profile
│   └── logo.png                        # Organization logo
└── README.md                           # This file
```

## Support

For questions or issues with these workflows, please:
1. Check the workflow logs for specific error messages
2. Verify all required secrets are configured
3. Ensure your registry credentials have permission to push images
4. Open an issue in this repository for workflow-related problems

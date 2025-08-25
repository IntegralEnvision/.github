# Organization Shared GitHub Actions

This repository contains reusable GitHub Actions workflows that can be shared across all repositories within the organization.

## Overview

This `.github` repository serves as a centralized location for:
- **Reusable Workflows**: Common CI/CD patterns that can be used across multiple repositories
- **Organization Profile**: Public-facing organization information and branding

## Available Workflows

### Docker Image Deployment (`dma-k8s-docker-build-cicd.yaml`)

A reusable workflow for building and deploying Docker images to Azure Container Registry (ACR). This workflow handles:
- Azure authentication
- Docker image building with multi-platform support
- Pushing to Azure Container Registry
- Automatic tagging with commit SHA and timestamps

#### Usage

To use this workflow in your repository, create a workflow file (e.g., `.github/workflows/deploy.yml`) with the following content:

```yaml
name: Deploy Docker Image

on:
  push:
    tags:
      - '**'  # Trigger on any tag push

jobs:
  deploy:
    uses: IntegralEnvision/.github/.github/workflows/dma-k8s-docker-build-cicd.yaml@main
    with:
      image_name: "your-app/your-service"
      registry: "integraldma.azurecr.io"
      dockerfile_path: "Dockerfile"  # Optional, defaults to "Dockerfile"
      platforms: "linux/amd64"      # Optional, defaults to "linux/amd64"
      tag_prefix: "prod"             # Optional, defaults to "dev"
    secrets:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
```

#### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `image_name` | Docker image name | `"shinyapps/app-flood"` |
| `registry` | Container registry URL | `"integraldma.azurecr.io"` |

#### Optional Inputs

| Input | Description | Default | Example |
|-------|-------------|---------|---------|
| `dockerfile_path` | Path to Dockerfile | `"Dockerfile"` | `"docker/Dockerfile"` |
| `platforms` | Target platforms for build | `"linux/amd64"` | `"linux/amd64,linux/arm64"` |
| `tag_prefix` | Prefix for generated tags | `"dev"` | `"prod"` |

#### Required Secrets

You must configure the following secrets in your repository or organization:

| Secret | Description |
|--------|-------------|
| `AZURE_CLIENT_ID` | Azure Service Principal Client ID |
| `AZURE_CLIENT_SECRET` | Azure Service Principal Client Secret |
| `AZURE_SUBSCRIPTION_ID` | Azure Subscription ID |
| `AZURE_TENANT_ID` | Azure Tenant ID |

#### Generated Tags

The workflow automatically generates the following tags for your Docker image:
- Standard metadata tags (based on git refs)
- Custom tag: `{COMMIT_SHA:7}-{tag_prefix}-{timestamp}`

Example: `abc1234-prod-1692975123`

## Setting Up Azure Authentication

To use the Docker deployment workflow, you need to create an Azure Service Principal and configure the required secrets.

### 1. Create Azure Service Principal

```bash
az ad sp create-for-rbac \
  --name "github-actions-sp" \
  --role contributor \
  --scopes /subscriptions/{subscription-id} \
  --sdk-auth
```

### 2. Configure Repository Secrets

Add the following secrets to your repository (Settings → Secrets and variables → Actions):

- `AZURE_CLIENT_ID`: From the Service Principal output
- `AZURE_CLIENT_SECRET`: From the Service Principal output  
- `AZURE_SUBSCRIPTION_ID`: Your Azure subscription ID
- `AZURE_TENANT_ID`: From the Service Principal output

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
│   └── dma-k8s-docker-build-cicd.yaml  # Docker deployment workflow
├── profile/
│   ├── README.md                        # Organization profile
│   └── logo.png                         # Organization logo
└── README.md                            # This file
```

## Support

For questions or issues with these workflows, please:
1. Check the workflow logs for specific error messages
2. Verify all required secrets are configured
3. Ensure your Azure Service Principal has the necessary permissions
4. Open an issue in this repository for workflow-related problems

# GitHub Workflows

This directory contains simplified CI/CD workflows for building and pushing container images to GitHub Container Registry (GHCR).

## Workflows

### build-dev.yaml
- **Trigger**: Pushes to `dev` branch or manual dispatch
- **Purpose**: Builds and pushes development images
- **Tags**: `dev-latest`, `dev-<sha>`

### build-test.yaml
- **Trigger**: Pushes to `test` branch or manual dispatch
- **Purpose**: Builds and pushes test images
- **Tags**: `test-latest`, `test-<sha>`

### build-prod.yaml
- **Trigger**: Pushes to `prod` branch or manual dispatch
- **Purpose**: Builds and pushes production images
- **Tags**: `prod-latest`, `prod-<sha>`, `latest`

## Services Built

All workflows build the following microservices:
- adservice
- cartservice
- checkoutservice
- currencyservice
- emailservice
- frontend
- loadgenerator
- paymentservice
- productcatalogservice
- recommendationservice
- shippingservice
- shoppingassistantservice

## Image Registry

Images are pushed to:
```
ghcr.io/<github-username>/<repo-name>/<service-name>:<tag>
```

## Prerequisites

- Repository must be public or you need to configure appropriate GHCR permissions
- No additional secrets required - workflows use the built-in `GITHUB_TOKEN`

## Manual Workflow Dispatch

All workflows can be manually triggered from the Actions tab in GitHub:
1. Go to Actions
2. Select the workflow
3. Click "Run workflow"
4. Select the branch
5. Click "Run workflow"

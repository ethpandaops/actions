# Docker Build and Push Action

A composite GitHub Action for building and optionally pushing Docker images to a container registry with multi-platform support.

## Usage

### Basic build only
```yaml
steps:
  - name: Build Docker image
    uses: ethpandaops/actions/docker-build-push@main
    with:
      image_name: my-app
```

### Build and push to registry
```yaml
steps:
  - name: Build and push Docker image
    uses: ethpandaops/actions/docker-build-push@main
    with:
      registry: ghcr.io
      registry_username: ${{ github.actor }}
      registry_password: ${{ secrets.GITHUB_TOKEN }}
      image_name: my-org/my-app
      push: true
```

### Custom platforms and Dockerfile
```yaml
steps:
  - name: Build multi-platform image
    uses: ethpandaops/actions/docker-build-push@main
    with:
      registry: docker.io
      registry_username: ${{ secrets.DOCKERHUB_USERNAME }}
      registry_password: ${{ secrets.DOCKERHUB_TOKEN }}
      image_name: my-username/my-app
      platforms: linux/amd64,linux/arm64,linux/arm/v7
      file: docker/Dockerfile.prod
      context: ./backend
      push: true
```

## Inputs

| Name                  | Description                           | Required | Default                      |
|-----------------------|---------------------------------------|----------|------------------------------|
| `registry`            | The registry to push the image to    | No       | `''`                         |
| `registry_username`   | The username to use to push the image| No       | `''`                         |
| `registry_password`   | The password to use to push the image| No       | `''`                         |
| `image_name`          | The name of the image to push        | No       | `''`                         |
| `platforms`           | The platforms to build the image for | No       | `linux/amd64,linux/arm64`   |
| `push`                | Whether to push the image            | No       | `false`                      |
| `file`                | The Dockerfile to use for building   | No       | `Dockerfile`                 |
| `context`             | The Docker build context            | No       | `.`                          |

## What it does

This action:
1. **Sets up QEMU** for cross-platform builds
2. **Sets up Docker Buildx** for advanced build features
3. **Logs into the registry** (only if push is enabled and credentials are provided)
4. **Extracts metadata** (tags, labels) from Git context using semantic versioning patterns:
   - `type=semver,pattern={{version}}` - for semantic version tags
   - `type=pep440,pattern={{version}}` - for Python PEP 440 version tags
   - `type=ref,event=branch` - for branch-based tags
5. **Builds the Docker image** with the specified platforms and optionally pushes it

## Supported Tag Patterns

The action automatically generates tags based on your Git context:
- **Semantic version tags**: `v1.2.3` → `registry/image:1.2.3`
- **PEP 440 version tags**: `1.2.3a1` → `registry/image:1.2.3a1`
- **Branch names**: `main` → `registry/image:main`, `feature/auth` → `registry/image:feature-auth`

## Notes

- When `push` is `false` or not set, the image will only be built locally
- Registry authentication is only performed when `push` is enabled
- The action supports multi-platform builds by default (`linux/amd64,linux/arm64`)
- All Docker actions use pinned SHA versions for security
- The build context defaults to the repository root (`.`)

## Common Registries

| Registry                | Example                              |
|------------------------|--------------------------------------|
| GitHub Container Registry | `ghcr.io`                        |
| Docker Hub             | `docker.io` or leave empty           |
| Amazon ECR             | `123456789012.dkr.ecr.region.amazonaws.com` |
| Google Container Registry | `gcr.io`                         |
| Azure Container Registry | `myregistry.azurecr.io`           |

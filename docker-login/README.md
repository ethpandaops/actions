# Docker Login Action

A composite GitHub Action for logging into Docker Hub with authentication credentials and checking rate limits.

## Usage

```yaml
steps:
  - name: Log in to Docker Hub
    uses: ethpandaops/actions/docker-login@main
    with:
      username: ${{ secrets.DOCKERHUB_USERNAME }}
      password: ${{ secrets.DOCKERHUB_TOKEN }}
```

## Inputs

| Name       | Description                   | Required |
|------------|-------------------------------|----------|
| `username` | Docker Hub username           | Yes      |
| `password` | Docker Hub password or token  | Yes      |

## What it does

This action:
1. Validates whether Docker Hub credentials have been provided
2. Logs into Docker Hub using the provided credentials
3. Installs `regctl` for working with container registries
4. Checks and displays Docker Hub rate limits for the authenticated user

## Notes

- If no credentials are provided, the action will skip the Docker Hub login step
- The action will always install `regctl` and check rate limits regardless of authentication status

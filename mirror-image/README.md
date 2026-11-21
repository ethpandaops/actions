# mirror-image

Mirrors images this job pushed to `ghcr.io/ethpandaops/<image>` to Docker Hub.

```yaml
permissions:
  contents: read
  packages: write
  id-token: write
  attestations: write

steps:
  # ... build and push ghcr.io/ethpandaops/my-image:v1.2.3 and :latest
  - uses: ethpandaops/actions/mirror-image@master
    with:
      image: my-image
      tags: v1.2.3 latest
      mirror-url: ${{ vars.IMAGE_MIRROR_URL }}
```

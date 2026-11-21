# mirror-image

Attests images this job pushed to `ghcr.io/ethpandaops/<image>` and mirrors
them to Docker Hub.

```yaml
permissions:
  contents: read
  packages: write
  id-token: write
  attestations: write

steps:
  # ... build and push ghcr.io/ethpandaops/my-image:v1.2.3 and :latest
  - id: build
    uses: docker/build-push-action@...
  - uses: ethpandaops/actions/mirror-image@master
    with:
      image: my-image
      tags: >-
        v1.2.3@${{ steps.build.outputs.digest }}
        latest@${{ steps.build.outputs.digest }}
      mirror-url: ${{ vars.IMAGE_MIRROR_URL }}
```

## What gets attested

The attestation records this job's repository, ref, workflow and runner as
the source of every digest it covers, and the mirror trusts exactly that.
So attest only what this job built:

- Give each tag as `tag@digest`, with the digest from your build step. The
  action fails if the tag no longer points there. A bare tag attests
  whatever it points to when the action runs, which may be something another
  job or person pushed in the meantime.
- By default the action also attests every per-platform manifest behind each
  tag. That is right when this job built all platforms.

### Split builds

If per-platform images are built in other jobs (for example on self-hosted
runners) and this job only merges them into an index, set
`attest-platforms: "false"`. The action then attests only the index. Each
platform job must attest its own digest, so its attestation records the
runner that built it:

```yaml
  # in each platform job, after pushing by digest
  - uses: actions/attest-build-provenance@...
    with:
      subject-name: ghcr.io/ethpandaops/my-image
      subject-digest: ${{ steps.build.outputs.digest }}
```

Otherwise this job would attest the self-hosted output as built on a
GitHub-hosted runner. The mirror refuses self-hosted attestations unless the
image's allowlist entry allows them.

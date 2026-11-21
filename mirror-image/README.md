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
  - id: meta
    uses: docker/metadata-action@...
    with:
      images: ghcr.io/ethpandaops/my-image
  - id: build
    uses: docker/build-push-action@...
    with:
      push: true
      tags: ${{ steps.meta.outputs.tags }}
  - uses: ethpandaops/actions/mirror-image@master
    with:
      tags: ${{ steps.meta.outputs.tags }}
      digest: ${{ steps.build.outputs.digest }}
      mirror-url: ${{ secrets.IMAGE_MIRROR_URL }}
```

`tags` takes plain tags (`v1.2.3 latest`), `tag@digest` pairs, or full
`ghcr.io/ethpandaops/<image>:<tag>` references. `image` defaults to the image
in those references, or else the repository name.

With goreleaser, point its `image_templates` at `ghcr.io/ethpandaops/...`
and pass its artifact list instead of tags. The action mirrors every image and
manifest goreleaser pushed, by digest:

```yaml
  - uses: ethpandaops/actions/mirror-image@master
    with:
      goreleaser-artifacts: dist/artifacts.json
      mirror-url: ${{ secrets.IMAGE_MIRROR_URL }}
```

If goreleaser pushes more than one image, run the step once per image with
`image` set.

`mirror-url` comes from the `IMAGE_MIRROR_URL` organization secret, so it's
masked in logs. It has to be passed in: composite actions can't read secrets
or variables themselves.

If the mirror refuses, the step fails and says what to do next, for example
asking the platform team to add the repository to the mirror's allowlist.

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

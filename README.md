# modpagespeed-images

Build, sign, and publish the public **ModPageSpeed 2.0** container images for
[We-Amp B.V.](https://we-amp.com) This repository holds no product source — it
assembles the runtime images from released binaries and publishes them, multi-arch
and signed, to the GitHub Container Registry.

## Images

- `ghcr.io/we-amp/pagespeed-combined` — the worker and nginx in one container (start here)
- `ghcr.io/we-amp/pagespeed-nginx` — nginx with the `ngx_pagespeed` module
- `ghcr.io/we-amp/pagespeed-worker` — the optimization worker

All are `linux/amd64` + `linux/arm64`, keyless-cosign-signed, and ship an SPDX SBOM and
SLSA build provenance. `:latest` is published only on the combined image; the worker and
nginx images use immutable version tags.

## Quick start

```bash
docker run --rm -p 80:80 \
  -e BACKEND_HOST=your-origin -e BACKEND_PORT=8081 -e ACCEPT_EULA=Y \
  ghcr.io/we-amp/pagespeed-combined:latest
```

See the [Docker install guide](https://modpagespeed.com/docs/installation-docker/) for the
production (separate worker + nginx) setup, Helm, and configuration.

Agent install recipes:

- Docker (all three images): https://modpagespeed.com/recipes/docker.md
- Helm (the nginx and worker images): https://modpagespeed.com/recipes/helm.md

## Verifying images

The images are signed only by `publish-images.yml` on `main`. Check that exact
identity: a pattern on the repository URL would also accept a signature from
any other workflow or branch of this repository.

```bash
cosign verify \
  --certificate-identity https://github.com/We-Amp/modpagespeed-images/.github/workflows/publish-images.yml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  ghcr.io/we-amp/pagespeed-combined:latest

gh attestation verify oci://ghcr.io/we-amp/pagespeed-combined:latest \
  --repo We-Amp/modpagespeed-images
```

## How the images are built

The publish workflow is started by the release run of the public
[pagespeed-optimizer](https://github.com/We-Amp/pagespeed-optimizer) repository,
which sends its run id, the release tag and the tag's commit. Before anything is
built the workflow checks those against the GitHub API: the run must belong to that
repository, come from its release workflow, match the tag and commit, and have built
the Linux image inputs successfully. The inputs are then downloaded from that run,
their digests are verified, and the images are built from the source at that commit.
The revision label and the creation date come from that commit, and the release run
is recorded in a label. Publishing needs a tag-triggered run and an approval.

## License

Using these images accepts the [Terms of Service](https://modpagespeed.com/terms/). Since
2.1 the images need no license key and add no license warning header.

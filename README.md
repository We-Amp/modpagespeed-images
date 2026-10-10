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

```bash
cosign verify \
  --certificate-identity-regexp '^https://github.com/We-Amp/modpagespeed-images/' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  ghcr.io/we-amp/pagespeed-combined:latest

gh attestation verify oci://ghcr.io/we-amp/pagespeed-combined:latest \
  --repo We-Amp/modpagespeed-images
```

## License

Using these images accepts the [Terms of Service](https://modpagespeed.com/terms/). Since
2.1 the images need no license key and add no license warning header.

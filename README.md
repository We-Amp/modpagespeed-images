# modpagespeed-images

Container image assembly + signing for **ModPageSpeed 2.0** (We-Amp B.V.) — see ADR-090.

This repo holds **no product source**. It assembles, signs, and publishes the public
runtime container images from per-arch release binaries built in the product repo
(`oschaaf/modpagespeed-2`). Keeping the publish step here is deliberate: the keyless
cosign identity then resolves to `github.com/We-Amp/…`, so the image namespace and the
signer identity are both `we-amp/` without transferring the product repo (ADR-090 D1/D6).

## What it publishes

- `ghcr.io/we-amp/pagespeed-nginx:<version>` — front nginx + ngx_pagespeed
- `ghcr.io/we-amp/pagespeed-worker:<version>` — the `factory_worker` optimizer

Both are multi-arch (`linux/amd64` + `linux/arm64`), keyless-cosign-signed, with an SPDX
SBOM and SLSA build provenance.

## How it works

1. The product repo's `release.yml` builds the per-arch release binaries, uploads them to
   `gs://weamp-release-artifacts` (ADR-034), and fires a `repository_dispatch` here on each
   `v*` tag.
2. This repo's workflow downloads the matching binaries, **COPY-assembles** the runtime
   images (no in-image recompile → cheap dual-arch, no OOM), merges the multi-arch
   manifest, keyless-cosign-signs each digest, attaches the SBOM + provenance, runs the
   publish smoke gate (incl. the licensed / `mps1`-stays-dark check), and pushes to
   `ghcr.io/we-amp/*`.

Because the signing workflow runs here, the keyless cosign identity is this repo:

```
cosign verify \
  --certificate-identity-regexp '^https://github.com/We-Amp/modpagespeed-images/' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  ghcr.io/we-amp/pagespeed-nginx:<version>
```

## Secrets

| Secret | Purpose |
|--------|---------|
| `GCS_ARTIFACTS_KEY` | read access to `gs://weamp-release-artifacts` (the `ci-artifacts` SA) |
| `PAGESPEED_LICENSE_KEY` | an ADR-033 prod-signed CI token, for the licensed smoke gate |

The built-in `GITHUB_TOKEN` (org-owned repo) handles `ghcr.io/we-amp/*` push + keyless
signing — no PAT needed.

> Licensing is BYOL (ADR-012/018). These images run unlicensed in **community mode**
> (soft enforcement, ADR-083); a license removes the warning and unlocks support +
> premium features.

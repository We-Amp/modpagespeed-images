# modpagespeed-images
Container image assembly + keyless signing for ModPageSpeed 2.0 (ADR-090): pulls per-arch release binaries from gs://weamp-release-artifacts, COPY-assembles, cosign-signs (keyless), and pushes ghcr.io/we-amp/pagespeed-{nginx,worker}.

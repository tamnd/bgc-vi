# Upstream sync

This translation is based on the English original:

- **Repo:** https://github.com/beejjorgensen/bgc
- **Commit:** `f6edbbbdb6cfd5f262a51261cc19cf87ceabc052`
- **Version:** v0.10.5 (April 18, 2026)

Update this file when syncing from a newer upstream commit. The release
workflow reads it to embed provenance in the release notes.

## Release tags

We tag translation releases as:

    v<UPSTREAM_VERSION>-vi.<N>

For example, `v0.10.5-vi.1` is the first Vietnamese release based on
upstream v0.10.5. Subsequent translation fixes against the same upstream
version bump `N` (`v0.10.5-vi.2`, `v0.10.5-vi.3`, ...). When upstream bumps
their version, we restart `N` at 1 (`v0.10.6-vi.1`).

Use `scripts/release.sh` to create and push a tag; the
`.github/workflows/release.yml` workflow then builds and publishes a
GitHub Release with all artifacts.

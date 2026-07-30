# Customization

## Bump versions

- Update `ARG III_VERSION` in `/home/runner/work/agentmemory-mcp/agentmemory-mcp/Dockerfile.iii-engine`, then push to `main` to rebuild `iii-engine`.
- Run the `build-agentmemory` workflow with a new `version` input to publish a newer `@agentmemory/agentmemory` package.

## Disable internet features

- The agentmemory image already pre-caches `Xenova/all-MiniLM-L6-v2` during the build.
- Runtime model downloads are disabled with `TRANSFORMERS_OFFLINE=1` and `HF_HUB_OFFLINE=1`.
- Keep the staging-to-Artifactory promotion step in place so the EC2 deployment never pulls directly from GHCR.

## Upstream alignment

If this split-image deployment needs first-party maintenance, request an upstream Dockerfile or deployment example from `rohitg00` so future agentmemory releases can be adopted with less local drift.

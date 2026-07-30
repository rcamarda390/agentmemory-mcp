# agentmemory-mcp

This repository builds the two container images needed for an air-gapped
agentmemory deployment:

- `ghcr.io/rcamarda390/iii-engine:0.11.2`
- `ghcr.io/rcamarda390/agentmemory:0.9.26`

## Supply-chain flow

1. GitHub Actions builds and pushes the images to GHCR.
2. The staging environment pulls them from GHCR for vetting.
3. Approved images are promoted into Artifactory.
4. The air-gapped EC2 environment pulls only from Artifactory.

`docker-compose.yml` defaults to GHCR and can be pointed at Artifactory by
setting `ARTIFACTORY_REGISTRY`, for example:

```bash
ARTIFACTORY_REGISTRY=artifactory.example.gov/containers docker compose up -d
```

## Images and workflows

### iii-engine

- Dockerfile: `/home/runner/work/agentmemory-mcp/agentmemory-mcp/Dockerfile.iii-engine`
- Source: iii `v0.11.2` release binary downloaded from GitHub releases
- Base image: distroless
- Runtime role: hosts the MCP/REST surface on `3111`, stream worker on `3112`,
  and WebSocket worker bridge on `49134`
- Workflow: `.github/workflows/build-iii.yml`
- Trigger: push to `main` when the iii Dockerfile/config changes, or manual dispatch

### agentmemory

- Dockerfile: `/home/runner/work/agentmemory-mcp/agentmemory-mcp/Dockerfile.agentmemory`
- Source: `@agentmemory/agentmemory` npm package
- Build stages: `npm-install` -> `model-cache` -> `production`
- Runtime role: registers the agentmemory worker against `iii-engine`, serves the
  viewer on `3113`, pre-caches `Xenova/all-MiniLM-L6-v2`, forces offline
  HuggingFace runtime settings, drops privileges with `gosu`, and generates a
  persistent HMAC secret on first boot
- Workflow: `.github/workflows/build-agentmemory.yml`
- Trigger: manual dispatch with a package version input; publishes both the
  requested tag and `latest`

## Local validation

```bash
docker build -f Dockerfile.iii-engine .
docker build -f Dockerfile.agentmemory .
docker compose config
```

## Deployment notes

- `iii-engine` persists engine state in the `iii-data` volume.
- `agentmemory` persists the generated HMAC secret in the `agentmemory-data`
  volume.
- The split deployment exposes `/agentmemory/livez` and `/agentmemory/health`
  through `iii-engine:3111` after the agentmemory worker connects.
- Runtime outbound model downloads are disabled with `TRANSFORMERS_OFFLINE=1`
  and `HF_HUB_OFFLINE=1`.
- No tarball artifacts are produced; the deliverable is the pushed container
  image set in GHCR.

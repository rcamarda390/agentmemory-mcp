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
- Workflow: `.github/workflows/build-iii.yml`
- Trigger: push to `main` when the iii Dockerfile/config changes, or manual dispatch

### agentmemory

- Dockerfile: `/home/runner/work/agentmemory-mcp/agentmemory-mcp/Dockerfile.agentmemory`
- Source: `@agentmemory/agentmemory` npm package
- Build stages: `npm-install` -> `model-cache` -> `production`
- Runtime: pre-cached `Xenova/all-MiniLM-L6-v2`, offline HuggingFace settings,
  `gosu` privilege drop, first-boot HMAC secret generation
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
- Runtime outbound model downloads are disabled with `TRANSFORMERS_OFFLINE=1`
  and `HF_HUB_OFFLINE=1`.
- No tarball artifacts are produced; the deliverable is the pushed container
  image set in GHCR.

# Installation - Local Environment {#sec-installation-local}

## Overview

Hyperledger Identus maintains component code across separate repositories. The Cloud Agent repository contains the server-side agent used in this chapter. A controller application sends HTTP requests to the Cloud Agent REST API. The Cloud Agent manages DIDs, DIDComm V2 messages, credential issuance, credential verification, credential storage, and tenant configuration. The official Cloud Agent README describes it as a server-side W3C/Aries cloud agent. Holder wallet applications use the Edge Agent SDKs.

The local setup runs the Cloud Agent in a development configuration. Docker starts the Cloud Agent with its supporting services, including PostgreSQL, HashiCorp Vault, APISIX, and a PRISM Node. The official Quick Start describes this setup as single-tenant, with API key authentication disabled and an in-memory ledger for published DID storage.

This chapter starts two local Cloud Agent instances:

- `issuer` on port `8000`, used to create issuer DIDs, schemas, and credential offers.
- `verifier` on port `9000`, used later for proof request flows.

For the REST API chapter, you can run only the issuer instance. The verifier becomes useful when the book reaches connections, issuance, and verification flows.

## Version Selection

Use the platform release notes before changing component versions. The current platform release page lists Identus Platform `v2.16` as the latest platform release and includes Cloud Agent `2.1.0`, Mediator `1.2.0`, NeoPRISM `0.6.2`, and SDK-TS `7.0.0`. The Quick Start pins the local issuer/verifier tutorial to Cloud Agent `2.0.0` and PRISM Node `2.6.0`.

This chapter follows the Quick Start version pair. Treat `AGENT_VERSION` and `PRISM_NODE_VERSION` as a tested pair. If you change one value, check the component release notes and rerun the health checks at the end of this chapter.

## Prerequisites

Install Git, Docker, and Docker Compose before running the local Cloud Agent stack. Docker Desktop includes the Docker CLI and Compose plugin on macOS and Windows. Linux users can use Docker Engine with the Compose plugin.

```bash
git --version
docker --version
docker compose version
```

Each command should print a version. If one command fails, install or repair that tool before starting the Cloud Agent.

On Windows, use WSL 2 and run the Linux commands inside the WSL distribution. The Microsoft WSL guide shows how to install a distribution, list installed distributions, and confirm whether a distribution uses WSL 1 or WSL 2.

## Clone the Cloud Agent Repository

Clone the current Cloud Agent repository and keep the local directory name used by the official Quick Start:

```bash
git clone https://github.com/hyperledger-identus/cloud-agent identus-cloud-agent
cd identus-cloud-agent
```

Older Identus material may use `https://github.com/hyperledger/identus-cloud-agent`. Identus Platform `v2.15` records the repository move to `hyperledger-identus/cloud-agent`.

## Configure Local Agent Instances

The `infrastructure/local` directory contains the scripts and environment files for local runs. The local README states that the scripts pull remote images, `.env` controls image versions, `run.sh` starts named instances, and `stop.sh` stops named instances.

Create the issuer configuration:

```bash
cat > ./infrastructure/local/.env-issuer <<'EOF'
API_KEY_ENABLED=false
AGENT_VERSION=2.0.0
PRISM_NODE_VERSION=2.6.0
PORT=8000
NETWORK=identus
VAULT_DEV_ROOT_TOKEN_ID=root
PG_PORT=5432
EOF
```

Create the verifier configuration:

```bash
cat > ./infrastructure/local/.env-verifier <<'EOF'
API_KEY_ENABLED=false
AGENT_VERSION=2.0.0
PRISM_NODE_VERSION=2.6.0
PORT=9000
NETWORK=identus
VAULT_DEV_ROOT_TOKEN_ID=root
PG_PORT=5433
EOF
```

::: {.callout-warning}
`API_KEY_ENABLED=false` disables API key authentication. Use this setting for local development only.
:::

The `PORT` values expose the two REST APIs on different host ports. The `PG_PORT` values prevent the two PostgreSQL containers from binding to the same host port. The `NETWORK=identus` value is retained from the official Quick Start configuration; the current local Compose file creates the runtime network from the named Compose project.

Current Hyperledger Identus releases publish the Cloud Agent image on Docker Hub as `docker.io/hyperledgeridentus/identus-cloud-agent`. The older local README mentions `GITHUB_TOKEN` and `ghcr.io`; that note predates the Docker registry move recorded in Identus Platform `v2.15`.

## Start the Issuer Cloud Agent

Run the issuer from the repository root. Use the command for your operating system.

macOS:

```bash
./infrastructure/local/run.sh -n issuer -b -w -e ./infrastructure/local/.env-issuer -p 8000 -d "$(ipconfig getifaddr $(route get default | grep interface | awk '{print $2}'))"
```

Linux:

```bash
./infrastructure/local/run.sh -n issuer -b -w -e ./infrastructure/local/.env-issuer -p 8000 -d "$(ip addr show $(ip route show default | awk '/default/ {print $5}') | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)"
```

The first run downloads container images and creates Docker volumes. Wait for Docker to report healthy containers, then check the issuer health endpoint:

```bash
curl http://localhost:8000/cloud-agent/_system/health
```

The response should include the configured Cloud Agent version:

```json
{"version":"2.0.0"}
```

The Cloud Agent serves the issuer REST API at `http://localhost:8000/cloud-agent/`. Swagger UI is available at `http://localhost:8000/apidocs/`, and the OpenAPI document is available at `http://localhost:8000/docs/cloud-agent/api/docs.yaml`.

## Start the Verifier Cloud Agent

Start the verifier when you need a second Cloud Agent actor for proof request and verification flows.

macOS:

```bash
./infrastructure/local/run.sh -n verifier -b -w -e ./infrastructure/local/.env-verifier -p 9000 -d "$(ipconfig getifaddr $(route get default | grep interface | awk '{print $2}'))"
```

Linux:

```bash
./infrastructure/local/run.sh -n verifier -b -w -e ./infrastructure/local/.env-verifier -p 9000 -d "$(ip addr show $(ip route show default | awk '/default/ {print $5}') | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)"
```

Check the verifier health endpoint:

```bash
curl http://localhost:9000/cloud-agent/_system/health
```

Expected response:

```json
{"version":"2.0.0"}
```

The Cloud Agent serves the verifier REST API at `http://localhost:9000/cloud-agent/`. Swagger UI is available at `http://localhost:9000/apidocs/`, and the OpenAPI document is available at `http://localhost:9000/docs/cloud-agent/api/docs.yaml`.

## Stop Local Instances

Stop each named instance with the same `-n` value used at startup:

```bash
./infrastructure/local/stop.sh -n issuer
./infrastructure/local/stop.sh -n verifier
```

To remove the local Docker volumes for an instance, add `-d`:

```bash
./infrastructure/local/stop.sh -n issuer -d
./infrastructure/local/stop.sh -n verifier -d
```

Use `-d` when you want a clean local state. Omit it when you want to keep local wallets, DIDs, schemas, credential records, and other development data across restarts.

## Local SSI Model

Each Cloud Agent instance acts as one SSI actor in this local setup. The issuer controls issuer DIDs and signs credentials. The verifier creates proof requests and verifies presentations. A holder wallet enters the flow later through an Edge Agent SDK or wallet application.

The local PRISM Node gives the Cloud Agent a development VDR target for `did:prism` operations. The in-memory ledger keeps the setup fast and disposable. Production deployments use different configuration for persistence, tenant isolation, API authentication, secrets, and Cardano network access.

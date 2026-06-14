# Installation - Production Environment {#sec-installation-production}

## Scope

Section 2 starts two local Cloud Agent instances with a development PRISM Node. This chapter keeps the same learning path and moves it toward a production-style `preprod` deployment. The goal is to show the service boundaries a developer must understand before a real issuer or verifier service depends on Identus.

The examples in this chapter are preprod scaffolding for operator learning. They show how to copy the local Docker configuration, pin current component versions, expose public Cloud Agent URLs, protect service ports, attach PRISM Node to Cardano services, and separate tutorial shortcuts from production requirements.

Run this chapter on Cardano `preprod` first. Move to `mainnet` after the deployment passes the health, wallet, DB Sync, DID publication, and backup checks at the end of the chapter.

## Version Selection

Use Identus platform release notes as the first compatibility source. At the time this chapter was checked on June 14, 2026, the latest Identus Platform release was `v2.16`. That release lists Cloud Agent `2.1.0`, Mediator `1.2.0`, NeoPRISM `0.6.2`, and SDK-TS `7.0.0`.

The Cloud Agent component repository has a newer component release, `v2.2.0`. That release adds VDR driver and NeoPRISM backend work. Treat it as a component release that requires compatibility testing against the platform pin. The Cloud Agent README warns that untested Cloud Agent and PRISM Node image pairs can fail compatibility checks.

This chapter uses these pins:

```bash
AGENT_VERSION=2.1.0
PRISM_NODE_VERSION=2.6.1
CARDANO_WALLET_TAG=v2026-05-11
CARDANO_DB_SYNC_VERSION=13.7.1.0
```

`AGENT_VERSION` follows the latest platform release. `PRISM_NODE_VERSION` follows the latest PRISM Node release checked for this chapter, `v2.6.1`. If you change either Identus image, rerun the full check list at the end of the chapter.

The Cardano Wallet release `v2026-05-11` is paired by its release notes with `cardano-node` `11.0.1`. The latest Cardano DB Sync release checked for this chapter was `13.7.1.0`.

## Deployment Model

A production Identus deployment has several actors and services:

- The controller application calls the Cloud Agent REST API and receives webhooks.
- The Cloud Agent manages tenant state, wallets, DIDs, DIDComm V2 messages, credential issuance, credential verification, credential storage, and status list publication.
- PostgreSQL stores Cloud Agent state. PRISM Node and Cardano DB Sync need their own PostgreSQL databases.
- Vault stores wallet seeds and secret material when `SECRET_STORAGE_BACKEND=vault`.
- The DID node backend publishes and resolves `did:prism` operations. Current Cloud Agent configuration can use PRISM Node, NeoPRISM, or other VDR drivers depending on the selected driver flags.
- PRISM Node acts as a level-2 node over Cardano. With `NODE_LEDGER=cardano`, it uses Cardano Wallet to submit transactions and Cardano DB Sync to read blocks.
- Cardano Wallet connects to a Cardano node through the node socket and exposes an HTTP API to PRISM Node.
- Cardano DB Sync follows the same Cardano node and indexes blocks into PostgreSQL for PRISM Node.

For a `did:prism` publication path:

1. The issuer controller asks the Cloud Agent to create, update, or deactivate a `did:prism` DID.
2. The Cloud Agent uses the wallet seed and stored derivation path to reconstruct key material for the DID operation.
3. The Cloud Agent signs the operation and sends it to the configured DID node backend.
4. PRISM Node receives the signed operation, schedules it, uses Cardano Wallet to submit a Cardano transaction, and reads indexed blocks through Cardano DB Sync.
5. The Cloud Agent or resolver reads the DID state after the operation reaches the configured confirmation depth.

Credential verification is a different decision path. Verifier software performs technical checks such as proof verification, DID resolution, key use, credential status, and presentation binding. The relying party then applies policy checks such as accepted issuer DIDs, schemas, credential types, trust registry entries, or business rules.

## Hardware and Data Requirements

Use live workload and Cardano service requirements for production sizing. The Cloud Agent itself is modest compared with the Cardano services. Production sizing depends on request rate, number of tenants, webhook volume, credential issuance volume, verification volume, database latency, and secret storage latency.

Cardano DB Sync is the largest resource consumer in this stack. The official DB Sync system requirements page listed these mainnet requirements when checked for this chapter:

- Linux host.
- At least 64 GB RAM.
- At least 4 CPU cores.
- SSD storage with 60k IOPS or better.
- At least 700 GB disk.

The same page listed current mainnet storage examples of about 203 GB for the Cardano node database, about 10 GB for DB Sync ledger state, and about 438 GB for the DB Sync PostgreSQL database. It listed mainnet memory examples of about 24 GB for `cardano-node` and about 21 GB RSS for `cardano-db-sync`.

`preprod` is much smaller. The DB Sync page listed about 12 GB for the Cardano node database, about 2 GB for DB Sync ledger state, about 16 GB for the DB Sync PostgreSQL database, about 5.5 GB RAM for `cardano-node`, and about 3.5 GB RSS for `cardano-db-sync`.

Run DB Sync and its PostgreSQL database close to each other. The DB Sync documentation recommends colocating them to reduce database traffic latency during synchronization. If you split services across machines, use a private network with low latency and monitor database IOPS.

## Prepare the Base Identus Configuration

Clone the current Cloud Agent repository and copy the local infrastructure directory into a `preprod` directory:

```bash
git clone https://github.com/hyperledger-identus/cloud-agent identus-cloud-agent
cd identus-cloud-agent

cp -R infrastructure/local infrastructure/preprod
cp infrastructure/shared/docker-compose.yml infrastructure/shared/docker-compose-preprod.yml
```

Edit `infrastructure/preprod/run.sh` so the preprod script uses the copied Compose file:

```diff
 PORT=${PORT} NETWORK=${NETWORK} DOCKERHOST=${DOCKERHOST} docker compose \
   -p ${NAME} \
-  -f ${SCRIPT_DIR}/../shared/docker-compose.yml \
+  -f ${SCRIPT_DIR}/../shared/docker-compose-preprod.yml \
   --env-file ${ENV_FILE} ${DEBUG} up ${BACKGROUND} ${WAIT}
```

If your checkout still references the old Cloud Agent registry, update the image name in `infrastructure/shared/docker-compose-preprod.yml`:

```diff
 cloud-agent:
-  image: ghcr.io/hyperledger/identus-cloud-agent:${AGENT_VERSION}
+  image: docker.io/hyperledgeridentus/identus-cloud-agent:${AGENT_VERSION:-latest}
```

Keep PostgreSQL off the host network. Add a private administration path when operators need direct database access:

```diff
 db:
-  ports:
-    - "127.0.0.1:${PG_PORT:-5432}:5432"
+  # Keep PostgreSQL inside the Docker network for this tutorial.
+  # Add a host binding for a private administration path.
+  # ports:
+  #   - "127.0.0.1:${PG_PORT:-5432}:5432"
```

Make the Cloud Agent environment explicit. The current Cloud Agent configuration supports `NODE_BACKEND`, PRISM Node connection settings, NeoPRISM connection settings, and VDR driver flags:

```diff
 cloud-agent:
   environment:
     POLLUX_STATUS_LIST_REGISTRY_PUBLIC_URL: ${POLLUX_STATUS_LIST_REGISTRY_PUBLIC_URL}
     DIDCOMM_SERVICE_URL: ${DIDCOMM_SERVICE_URL}
     REST_SERVICE_URL: ${REST_SERVICE_URL}
     PRISM_NODE_HOST: ${PRISM_NODE_HOST:-prism-node}
     PRISM_NODE_PORT: ${PRISM_NODE_PORT:-50053}
+    PRISM_NODE_USE_PLAIN_TEXT: ${PRISM_NODE_USE_PLAIN_TEXT:-true}
+    NODE_BACKEND: ${NODE_BACKEND:-prism-node}
+    NEOPRISM_BASE_URL: ${NEOPRISM_BASE_URL}
     VAULT_ADDR: ${VAULT_ADDR:-http://vault-server:8200}
     VAULT_TOKEN: ${VAULT_TOKEN:-root}
-    SECRET_STORAGE_BACKEND: postgres
-    DEV_MODE: true
+    SECRET_STORAGE_BACKEND: ${SECRET_STORAGE_BACKEND:-postgres}
+    DEV_MODE: ${DEV_MODE:-true}
+    VDR_PRISM_NODE_DRIVER_ENABLED: ${VDR_PRISM_NODE_DRIVER_ENABLED:-false}
+    VDR_NEOPRISM_DRIVER_ENABLED: ${VDR_NEOPRISM_DRIVER_ENABLED:-false}
     ADMIN_TOKEN:
     API_KEY_SALT:
     API_KEY_ENABLED:
     API_KEY_AUTHENTICATE_AS_DEFAULT_USER:
     API_KEY_AUTO_PROVISIONING:
```

Use PRISM Node for this chapter:

```bash
NODE_BACKEND=prism-node
PRISM_NODE_HOST=prism-node
PRISM_NODE_PORT=50053
PRISM_NODE_USE_PLAIN_TEXT=true
```

New deployments should evaluate NeoPRISM before mainnet. The current Cloud Agent VDR documentation marks the NeoPRISM driver as recommended for production and the PRISM Node driver as a legacy production option. This chapter keeps PRISM Node so the tutorial can show the Cardano Wallet and DB Sync boundary in detail.

## Create the Preprod Environment File

Create `infrastructure/preprod/.env-preprod`:

```bash
cat > ./infrastructure/preprod/.env-preprod <<EOF
### Identus image pins
AGENT_VERSION=2.1.0
PRISM_NODE_VERSION=2.6.1

### Docker project and public URLs
PORT=8000
NETWORK=identus-preprod
DOCKERHOST=agent.example.test
REST_SERVICE_URL=https://agent.example.test/cloud-agent
DIDCOMM_SERVICE_URL=https://agent.example.test/didcomm
POLLUX_STATUS_LIST_REGISTRY_PUBLIC_URL=https://agent.example.test/cloud-agent

### Cloud Agent access control
API_KEY_ENABLED=true
API_KEY_AUTO_PROVISIONING=false
API_KEY_AUTHENTICATE_AS_DEFAULT_USER=false
ADMIN_TOKEN=replace-with-admin-token
API_KEY_SALT=replace-with-long-random-salt
DEFAULT_WALLET_ENABLED=false

### Secret storage
SECRET_STORAGE_BACKEND=postgres
VAULT_ADDR=http://vault-server:8200
VAULT_USE_SEMANTIC_PATH=true
VAULT_DEV_ROOT_TOKEN_ID=replace-for-lab
VAULT_TOKEN=replace-for-lab

### Databases
AGENT_DB_USER=postgres
AGENT_DB_PASSWORD=replace-with-agent-db-password
PGADMIN_DEFAULT_PASSWORD=replace-with-pgadmin-password

### DID node backend
NODE_BACKEND=prism-node
PRISM_NODE_HOST=prism-node
PRISM_NODE_PORT=50053
PRISM_NODE_USE_PLAIN_TEXT=true
VDR_PRISM_NODE_DRIVER_ENABLED=false
VDR_NEOPRISM_DRIVER_ENABLED=false

### PRISM Node Cardano mode
NODE_LEDGER=cardano
NODE_CARDANO_NETWORK=testnet
NODE_CARDANO_WALLET_ID=replace-after-wallet-create
NODE_CARDANO_WALLET_PASSPHRASE=replace-after-wallet-create
NODE_CARDANO_PAYMENT_ADDRESS=replace-after-wallet-create
NODE_CARDANO_WALLET_API_HOST=cardano-wallet
NODE_CARDANO_WALLET_API_PORT=8090
NODE_CARDANO_DB_SYNC_HOST=cardano-db-sync-postgres:5432
NODE_CARDANO_DB_SYNC_DATABASE=cexplorer
NODE_CARDANO_DB_SYNC_USERNAME=postgres
NODE_CARDANO_DB_SYNC_PASSWORD=replace-with-db-sync-password

### Cardano services
CARDANO_NETWORK=preprod
CARDANO_WALLET_TAG=v2026-05-11
CARDANO_DB_SYNC_VERSION=13.7.1.0
NODE_DB=$PWD/cardano/node-db
WALLET_DB=$PWD/cardano/wallet-db
NODE_CONFIGS=$PWD/cardano/configs
NODE_SOCKET_DIR=$PWD/cardano/ipc
NODE_SOCKET_NAME=node.socket
WALLET_PORT=127.0.0.1:8090

### Keycloak, disabled until the Keycloak section is completed
KEYCLOAK_ENABLED=false
KEYCLOAK_URL=https://iam.example.test
KEYCLOAK_REALM=identus
KEYCLOAK_CLIENT_ID=cloud-agent
KEYCLOAK_CLIENT_SECRET=replace-when-keycloak-enabled
KEYCLOAK_UMA_AUTO_UPGRADE_RPT=false
EOF
```

`SECRET_STORAGE_BACKEND=postgres` keeps the lab small. The Cloud Agent secret storage documentation states that PostgreSQL storage stores secrets in plaintext and is unsuitable for production. Before issuer traffic reaches this stack, set `SECRET_STORAGE_BACKEND=vault` and configure Vault with a production server, token policy, or AppRole.

## Production-Style Compose Example

The `infrastructure/shared/docker-compose-preprod.yml` example collects the earlier edits into one file. It preserves the full configuration view from the original tutorial and keeps the production boundaries visible.

This file is still a preprod lab scaffold. Replace the development Vault server, default database credentials, APISIX HTTP listener, and DB Sync service definition before mainnet. Keep the file under source control so developers can compare the local stack with the production-style stack.

```yaml
---
version: "3.8"

services:
  db:
    image: postgres:13
    environment:
      POSTGRES_MULTIPLE_DATABASES: "pollux,connect,agent,node_db"
      POSTGRES_USER: ${AGENT_DB_USER:-postgres}
      POSTGRES_PASSWORD: ${AGENT_DB_PASSWORD}
    volumes:
      - pg_data_db:/var/lib/postgresql/data
      - ./postgres/init-script.sh:/docker-entrypoint-initdb.d/init-script.sh
      - ./postgres/max_conns.sql:/docker-entrypoint-initdb.d/max_conns.sql
    # Keep the database inside the Docker network.
    # ports:
    #   - "127.0.0.1:${PG_PORT:-5432}:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "${AGENT_DB_USER:-postgres}", "-d", "agent"]
      interval: 10s
      timeout: 5s
      retries: 5

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-pgadmin4@pgadmin.org}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD}
      PGADMIN_CONFIG_SERVER_MODE: "False"
    volumes:
      - pgadmin:/var/lib/pgadmin
    ports:
      - "127.0.0.1:${PGADMIN_PORT:-5050}:80"
    depends_on:
      db:
        condition: service_healthy
    profiles:
      - debug

  vault-server:
    image: hashicorp/vault:1.15.6
    environment:
      VAULT_ADDR: "http://0.0.0.0:8200"
      VAULT_DEV_ROOT_TOKEN_ID: ${VAULT_DEV_ROOT_TOKEN_ID}
    command: server -dev -dev-root-token-id=${VAULT_DEV_ROOT_TOKEN_ID}
    cap_add:
      - IPC_LOCK
    healthcheck:
      test: ["CMD", "vault", "status"]
      interval: 10s
      timeout: 5s
      retries: 5

  prism-node:
    image: ghcr.io/input-output-hk/prism-node:${PRISM_NODE_VERSION}
    environment:
      NODE_PSQL_HOST: db:5432
      NODE_PSQL_DATABASE: node_db
      NODE_PSQL_USERNAME: ${AGENT_DB_USER:-postgres}
      NODE_PSQL_PASSWORD: ${AGENT_DB_PASSWORD}
      NODE_LEDGER: ${NODE_LEDGER:-cardano}
      NODE_CARDANO_NETWORK: ${NODE_CARDANO_NETWORK:-testnet}
      NODE_CARDANO_WALLET_ID: ${NODE_CARDANO_WALLET_ID}
      NODE_CARDANO_WALLET_PASSPHRASE: ${NODE_CARDANO_WALLET_PASSPHRASE}
      NODE_CARDANO_PAYMENT_ADDRESS: ${NODE_CARDANO_PAYMENT_ADDRESS}
      NODE_CARDANO_WALLET_API_HOST: ${NODE_CARDANO_WALLET_API_HOST:-cardano-wallet}
      NODE_CARDANO_WALLET_API_PORT: ${NODE_CARDANO_WALLET_API_PORT:-8090}
      NODE_CARDANO_DB_SYNC_HOST: ${NODE_CARDANO_DB_SYNC_HOST}
      NODE_CARDANO_DB_SYNC_DATABASE: ${NODE_CARDANO_DB_SYNC_DATABASE:-cexplorer}
      NODE_CARDANO_DB_SYNC_USERNAME: ${NODE_CARDANO_DB_SYNC_USERNAME}
      NODE_CARDANO_DB_SYNC_PASSWORD: ${NODE_CARDANO_DB_SYNC_PASSWORD}
    depends_on:
      db:
        condition: service_healthy
      cardano-wallet:
        condition: service_started
      cardano-db-sync-postgres:
        condition: service_started

  cloud-agent:
    image: docker.io/hyperledgeridentus/identus-cloud-agent:${AGENT_VERSION:-latest}
    environment:
      POLLUX_DB_HOST: db
      POLLUX_DB_PORT: 5432
      POLLUX_DB_NAME: pollux
      POLLUX_DB_USER: ${AGENT_DB_USER:-postgres}
      POLLUX_DB_PASSWORD: ${AGENT_DB_PASSWORD}
      CONNECT_DB_HOST: db
      CONNECT_DB_PORT: 5432
      CONNECT_DB_NAME: connect
      CONNECT_DB_USER: ${AGENT_DB_USER:-postgres}
      CONNECT_DB_PASSWORD: ${AGENT_DB_PASSWORD}
      AGENT_DB_HOST: db
      AGENT_DB_PORT: 5432
      AGENT_DB_NAME: agent
      AGENT_DB_USER: ${AGENT_DB_USER:-postgres}
      AGENT_DB_PASSWORD: ${AGENT_DB_PASSWORD}
      POLLUX_STATUS_LIST_REGISTRY_PUBLIC_URL: ${POLLUX_STATUS_LIST_REGISTRY_PUBLIC_URL}
      DIDCOMM_SERVICE_URL: ${DIDCOMM_SERVICE_URL}
      REST_SERVICE_URL: ${REST_SERVICE_URL}
      PRISM_NODE_HOST: ${PRISM_NODE_HOST:-prism-node}
      PRISM_NODE_PORT: ${PRISM_NODE_PORT:-50053}
      PRISM_NODE_USE_PLAIN_TEXT: ${PRISM_NODE_USE_PLAIN_TEXT:-true}
      NODE_BACKEND: ${NODE_BACKEND:-prism-node}
      NEOPRISM_BASE_URL: ${NEOPRISM_BASE_URL}
      VAULT_ADDR: ${VAULT_ADDR:-http://vault-server:8200}
      VAULT_TOKEN: ${VAULT_TOKEN}
      VAULT_APPROLE_ROLE_ID: ${VAULT_APPROLE_ROLE_ID}
      VAULT_APPROLE_SECRET_ID: ${VAULT_APPROLE_SECRET_ID}
      VAULT_USE_SEMANTIC_PATH: ${VAULT_USE_SEMANTIC_PATH:-true}
      SECRET_STORAGE_BACKEND: ${SECRET_STORAGE_BACKEND:-postgres}
      DEV_MODE: ${DEV_MODE:-false}
      DEFAULT_WALLET_ENABLED: ${DEFAULT_WALLET_ENABLED:-false}
      DEFAULT_WALLET_SEED: ${DEFAULT_WALLET_SEED}
      DEFAULT_WALLET_WEBHOOK_URL: ${DEFAULT_WALLET_WEBHOOK_URL}
      DEFAULT_WALLET_WEBHOOK_API_KEY: ${DEFAULT_WALLET_WEBHOOK_API_KEY}
      DEFAULT_WALLET_AUTH_API_KEY: ${DEFAULT_WALLET_AUTH_API_KEY}
      GLOBAL_WEBHOOK_URL: ${GLOBAL_WEBHOOK_URL}
      GLOBAL_WEBHOOK_API_KEY: ${GLOBAL_WEBHOOK_API_KEY}
      WEBHOOK_PARALLELISM: ${WEBHOOK_PARALLELISM}
      ADMIN_TOKEN: ${ADMIN_TOKEN}
      API_KEY_SALT: ${API_KEY_SALT}
      API_KEY_ENABLED: ${API_KEY_ENABLED:-true}
      API_KEY_AUTHENTICATE_AS_DEFAULT_USER: ${API_KEY_AUTHENTICATE_AS_DEFAULT_USER:-false}
      API_KEY_AUTO_PROVISIONING: ${API_KEY_AUTO_PROVISIONING:-false}
      KEYCLOAK_ENABLED: ${KEYCLOAK_ENABLED:-false}
      KEYCLOAK_URL: ${KEYCLOAK_URL}
      KEYCLOAK_REALM: ${KEYCLOAK_REALM}
      KEYCLOAK_CLIENT_ID: ${KEYCLOAK_CLIENT_ID}
      KEYCLOAK_CLIENT_SECRET: ${KEYCLOAK_CLIENT_SECRET}
      KEYCLOAK_UMA_AUTO_UPGRADE_RPT: ${KEYCLOAK_UMA_AUTO_UPGRADE_RPT:-false}
      VDR_PRISM_NODE_DRIVER_ENABLED: ${VDR_PRISM_NODE_DRIVER_ENABLED:-false}
      VDR_NEOPRISM_DRIVER_ENABLED: ${VDR_NEOPRISM_DRIVER_ENABLED:-false}
    depends_on:
      db:
        condition: service_healthy
      prism-node:
        condition: service_started
      vault-server:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://cloud-agent:8085/_system/health"]
      interval: 30s
      timeout: 10s
      retries: 5
    extra_hosts:
      - "host.docker.internal:host-gateway"

  swagger-ui:
    image: swaggerapi/swagger-ui:v5.1.0
    environment:
      - 'URLS=[
        { name: "Cloud Agent", url: "/docs/cloud-agent/api/docs.yaml" }
        ]'

  apisix:
    image: apache/apisix:2.15.0-alpine
    volumes:
      - ./apisix/conf/apisix.yaml:/usr/local/apisix/conf/apisix.yaml:ro
      - ./apisix/conf/config.yaml:/usr/local/apisix/conf/config.yaml:ro
    ports:
      - "${PORT}:9080/tcp"
    depends_on:
      - cloud-agent
      - swagger-ui

  cardano-node:
    image: cardanofoundation/cardano-wallet:${CARDANO_WALLET_TAG}
    environment:
      CARDANO_NODE_SOCKET_PATH: /ipc/${NODE_SOCKET_NAME}
    volumes:
      - ${NODE_DB}:/data
      - ${NODE_SOCKET_DIR}:/ipc
      - ${NODE_CONFIGS}:/configs
    entrypoint: []
    command: >
      cardano-node run
        --topology /configs/cardano/${CARDANO_NETWORK}/topology.json
        --database-path /data
        --socket-path /ipc/${NODE_SOCKET_NAME}
        --config /configs/cardano/${CARDANO_NETWORK}/config.json
        +RTS -N -A16m -qg -qb -RTS
    restart: on-failure

  cardano-wallet:
    image: cardanofoundation/cardano-wallet:${CARDANO_WALLET_TAG}
    volumes:
      - ${WALLET_DB}:/wallet-db
      - ${NODE_SOCKET_DIR}:/ipc
      - ${NODE_CONFIGS}:/configs
    ports:
      - "${WALLET_PORT}:8090"
    entrypoint: []
    command: >
      cardano-wallet serve
        --node-socket /ipc/${NODE_SOCKET_NAME}
        --database /wallet-db
        --listen-address 0.0.0.0
        --testnet /configs/cardano/${CARDANO_NETWORK}/byron-genesis.json
    depends_on:
      - cardano-node
    restart: on-failure

  cardano-db-sync-postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: ${NODE_CARDANO_DB_SYNC_USERNAME:-postgres}
      POSTGRES_PASSWORD: ${NODE_CARDANO_DB_SYNC_PASSWORD}
      POSTGRES_DB: ${NODE_CARDANO_DB_SYNC_DATABASE:-cexplorer}
    volumes:
      - cardano_db_sync_postgres:/var/lib/postgresql/data

  cardano-db-sync:
    image: ghcr.io/intersectmbo/cardano-db-sync:${CARDANO_DB_SYNC_VERSION}
    depends_on:
      - cardano-node
      - cardano-db-sync-postgres
    volumes:
      - ${NODE_SOCKET_DIR}:/ipc
      - ${NODE_CONFIGS}:/configs
    environment:
      CARDANO_NODE_SOCKET_PATH: /ipc/${NODE_SOCKET_NAME}
      NETWORK: ${CARDANO_NETWORK}
      POSTGRES_HOST: cardano-db-sync-postgres
      POSTGRES_PORT: 5432
      POSTGRES_DB: ${NODE_CARDANO_DB_SYNC_DATABASE:-cexplorer}
      POSTGRES_USER: ${NODE_CARDANO_DB_SYNC_USERNAME:-postgres}
      POSTGRES_PASSWORD: ${NODE_CARDANO_DB_SYNC_PASSWORD}

volumes:
  pg_data_db:
  pgadmin:
  cardano_db_sync_postgres:
```

## Production Configuration and Security

Treat the preprod file as a working reference for placeholders and service boundaries. Commit the sample so developers can see every moving part. Load real values from your deployment system, CI secret store, Vault, or a sealed secret mechanism.

A production Cloud Agent configuration should make these decisions explicit:

- Image pins for Cloud Agent, PRISM Node or NeoPRISM, Cardano Wallet, Cardano node, and DB Sync.
- Public URLs for `REST_SERVICE_URL`, `DIDCOMM_SERVICE_URL`, and `POLLUX_STATUS_LIST_REGISTRY_PUBLIC_URL`.
- Tenant model: default wallet disabled for multi-tenant deployments, API keys or Keycloak enabled, and a long random `ADMIN_TOKEN`.
- Secret backend: `SECRET_STORAGE_BACKEND=vault` for issuer and verifier deployments that keep wallet seeds.
- DID backend: PRISM Node for this tutorial path, or NeoPRISM after you validate the newer production driver in your target version.
- Cardano network wiring: `NODE_CARDANO_NETWORK=testnet` or `mainnet` for PRISM Node, and `CARDANO_NETWORK=preprod` or `mainnet` for Cardano node, Cardano Wallet, and DB Sync configuration.

The public URL values are part of protocol behavior. Holder wallets and mediator services use the DIDComm service endpoint to deliver DIDComm V2 messages. Verifier software or controller applications use the REST URL for API calls and status list access. If a load balancer rewrites the path or host, update the Cloud Agent URLs to match the externally reachable address.

Set `DEV_MODE=false` for deployments beyond the local tutorial. Rotate `ADMIN_TOKEN`, API keys, database passwords, Vault credentials, Keycloak client secrets, and wallet passphrases through the same process used for other production credentials. Store real wallet seeds, Cardano mnemonics, and Keycloak admin credentials in the production secret system.

## Docker and PostgreSQL Hardening

The Compose file keeps the original tutorial shape so a developer can inspect all services. Production operators should reduce the Docker surface area.

Keep PostgreSQL ports off public interfaces. The Cloud Agent database, PRISM Node database, and DB Sync database can share a managed PostgreSQL cluster. Each service should use a separate database, separate credentials, and a backup policy that matches its recovery needs. DB Sync stores chain index data that can be rebuilt or restored from snapshots. Cloud Agent and PRISM Node databases hold application state that should be backed up and restore-tested.

Replace default `postgres` passwords before the first non-local run. The tutorial uses `POSTGRES_MULTIPLE_DATABASES` for convenience; a production deployment should create databases and users through migrations, infrastructure code, or managed database provisioning.

Docker Compose secrets mount files under `/run/secrets/<name>` inside a container. Images such as PostgreSQL support the `_FILE` convention for some secret values. Use that pattern where the image supports it:

```yaml
secrets:
  agent_db_password:
    file: ./config/secrets/agent_db_password

services:
  db:
    image: postgres:13
    secrets:
      - agent_db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/agent_db_password
```

The Cloud Agent reads its database, API, Vault, and Keycloak values from environment variables. Inject those values through the orchestrator or deployment platform. Compose secrets for Cloud Agent variables require an entrypoint wrapper that reads secret files and exports the expected variable names.

Set log rotation for long-running containers. Cardano node, Cardano Wallet, and DB Sync produce continuous logs during synchronization. The example later in this chapter sets `json-file` size limits for the Cardano services; carry the same policy to Cloud Agent, PRISM Node, APISIX, Vault, and PostgreSQL when Compose is used beyond a lab.

## SSL and Reverse Proxy

Expose the Cloud Agent through HTTPS. The example APISIX service listens on HTTP port `9080` so the lab remains simple. For production, terminate TLS at APISIX, a load balancer, or another reverse proxy, then forward to the internal Cloud Agent and DIDComm routes.

Use a public route plan before creating DIDs or issuing credentials:

```text
https://agent.example.test/cloud-agent  -> Cloud Agent REST API
https://agent.example.test/didcomm      -> Cloud Agent DIDComm endpoint
https://agent.example.test/apidocs      -> Swagger UI, private or disabled
https://iam.example.test/realms/...     -> Keycloak OIDC and UMA endpoints
```

If APISIX terminates TLS, configure TLS 1.2 and TLS 1.3 at the gateway level or on each SNI-specific SSL resource:

```yaml
apisix:
  ssl:
    ssl_protocols: TLSv1.2 TLSv1.3
```

Bind the APISIX admin API to a private network. If your deployment exposes Swagger UI, protect it with network controls or authentication. The OpenAPI route is useful for developer onboarding and gives attackers a complete API map.

Keycloak needs a separate proxy configuration. Keycloak production docs require HTTPS for sensitive authentication traffic and recommend a reverse proxy or load balancer.

Proxy the Keycloak application port, usually `8443` or `8080` when HTTP is enabled behind the proxy. Keep the management port `9000` internal.

Expose the public OIDC and static paths needed by clients, such as `/realms/`, `/resources/`, and `/.well-known/`. Keep `/admin/`, `/metrics`, and `/health` on an internal administration path. Configure `proxy-headers` and trusted proxy addresses so Keycloak accepts forwarded host and scheme values from trusted proxies.

## Key Management with HashiCorp Vault

The Cloud Agent creates, stores, and uses wallet seed material for DID operations. A tenant wallet seed lets the agent derive DID keys again after restart or redeploy. Losing it can make existing DIDs unusable for future updates, deactivation, or issuance flows that depend on the same keys.

Set the production backend to Vault:

```bash
SECRET_STORAGE_BACKEND=vault
VAULT_ADDR=https://vault.example.internal:8200
VAULT_USE_SEMANTIC_PATH=true
```

Reserve `VAULT_TOKEN` for a lab or break-glass workflow. For automated deployment, prefer AppRole:

```bash
VAULT_APPROLE_ROLE_ID=replace-with-role-id
VAULT_APPROLE_SECRET_ID=replace-with-secret-id
```

The Cloud Agent expects permissions under `/secret/*` for the tutorial mount and policy design. A minimal policy for that path is:

```hcl
path "secret/*" {
  capabilities = ["create", "read", "update", "patch", "delete", "list"]
}
```

Vault stores Cloud Agent secrets under wallet-scoped paths such as `/secret/<wallet-id>/seed`, peer DID key paths, and generic secret paths. Back up Vault storage, test restore, and document the unseal or recovery-key procedure. A development Vault server started with `server -dev` is lab storage. HashiCorp’s production hardening guidance calls out TLS, unprivileged runtime users, memory locking or encrypted swap decisions, storage separation, and restricted network paths.

## Multi-Tenant Operation

The Cloud Agent tenant model separates administrator actions from tenant actions. The administrator manages wallets, entities, API keys, and optional Keycloak permissions. The tenant uses the Cloud Agent for SSI actions inside the wallet assigned to that tenant.

Use these Cloud Agent variables for basic multi-tenancy:

```bash
ADMIN_TOKEN=replace-with-admin-token
API_KEY_ENABLED=true
API_KEY_AUTO_PROVISIONING=false
API_KEY_AUTHENTICATE_AS_DEFAULT_USER=false
DEFAULT_WALLET_ENABLED=false
```

With those settings, the Cloud Agent starts with an empty tenant wallet set. The administrator creates a wallet, creates an entity tied to that wallet, and registers an API key for that entity:

```bash
curl -X POST "https://agent.example.test/cloud-agent/wallets" \
  -H "x-admin-api-key: $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "issuer-tenant-a"
  }'
```

```bash
curl -X POST "https://agent.example.test/cloud-agent/iam/entities" \
  -H "x-admin-api-key: $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "tenant-a",
    "walletId": "replace-with-wallet-id"
  }'
```

```bash
curl -X POST "https://agent.example.test/cloud-agent/iam/apikey-authentication" \
  -H "x-admin-api-key: $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "entityId": "replace-with-entity-id",
    "apiKey": "replace-with-tenant-api-key"
  }'
```

The tenant uses the assigned API key in the `apikey` header:

```bash
curl "https://agent.example.test/cloud-agent/did-registrar/dids" \
  -H "apikey: replace-with-tenant-api-key" \
  -H "Accept: application/json"
```

The response is scoped to the tenant wallet. A tenant that issues credentials from one wallet gains access to that tenant's DIDs, credentials, connections, and verification records.

## Keycloak External IAM

Use Keycloak when tenants should authenticate through OIDC and receive wallet permissions through UMA. This replaces static Cloud Agent API keys with Keycloak-issued tokens for tenant access. In this model, the Cloud Agent still owns wallet resources. Keycloak owns user authentication and authorization tokens.

Configure Keycloak with a dedicated client for the Cloud Agent and enable the authorization services required for UMA. Then switch the Cloud Agent to Keycloak:

```bash
KEYCLOAK_ENABLED=true
KEYCLOAK_URL=https://iam.example.test
KEYCLOAK_REALM=identus
KEYCLOAK_CLIENT_ID=cloud-agent
KEYCLOAK_CLIENT_SECRET=replace-with-client-secret
KEYCLOAK_UMA_AUTO_UPGRADE_RPT=false
```

In the Keycloak flow, the administrator creates a wallet in the Cloud Agent, registers a user in Keycloak, and grants that Keycloak subject permission to the wallet:

```bash
curl -X POST "https://agent.example.test/cloud-agent/wallets/replace-with-wallet-id/uma-permissions" \
  -H "x-admin-api-key: $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "replace-with-keycloak-user-id"
  }'
```

The tenant obtains an OIDC access token from Keycloak, requests an UMA requesting party token with `grant_type=urn:ietf:params:oauth:grant-type:uma-ticket`, and calls the Cloud Agent with:

```bash
Authorization: Bearer replace-with-rpt
```

If `KEYCLOAK_UMA_AUTO_UPGRADE_RPT=true`, the Cloud Agent can accept the tenant's access token and perform the RPT upgrade path described in the Cloud Agent external IAM guide. Keep `false` until the controller application owns the token exchange and error handling.

## Cardano Wallet Operations

PRISM Node in Cardano mode has three external dependencies: its own PostgreSQL database, a Cardano Wallet backend, and Cardano DB Sync. The local Identus Compose file already creates the Node database through the shared PostgreSQL service. Add Cardano Wallet and a Cardano node to `infrastructure/shared/docker-compose-preprod.yml`.

Use separate variables for the PRISM Node network selector and the Cardano service configuration directory:

```bash
NODE_CARDANO_NETWORK=testnet
CARDANO_NETWORK=preprod
```

PRISM Node accepts `testnet` and `mainnet` for `NODE_CARDANO_NETWORK`. Cardano Wallet, Cardano node, and DB Sync use `preprod`, `preview`, or `mainnet` configuration names. This tutorial uses Cardano `preprod`, which is still a PRISM Node `testnet` deployment.

```yaml
  cardano-node:
    image: cardanofoundation/cardano-wallet:${CARDANO_WALLET_TAG}
    environment:
      CARDANO_NODE_SOCKET_PATH: /ipc/${NODE_SOCKET_NAME}
    volumes:
      - ${NODE_DB}:/data
      - ${NODE_SOCKET_DIR}:/ipc
      - ${NODE_CONFIGS}:/configs
    entrypoint: []
    command: >
      cardano-node run
        --topology /configs/cardano/${CARDANO_NETWORK}/topology.json
        --database-path /data
        --socket-path /ipc/${NODE_SOCKET_NAME}
        --config /configs/cardano/${CARDANO_NETWORK}/config.json
        +RTS -N -A16m -qg -qb -RTS
    restart: on-failure
    logging:
      driver: "json-file"
      options:
        compress: "true"
        max-file: "10"
        max-size: "50m"

  cardano-wallet:
    image: cardanofoundation/cardano-wallet:${CARDANO_WALLET_TAG}
    volumes:
      - ${WALLET_DB}:/wallet-db
      - ${NODE_SOCKET_DIR}:/ipc
      - ${NODE_CONFIGS}:/configs
    ports:
      - "${WALLET_PORT}:8090"
    entrypoint: []
    command: >
      cardano-wallet serve
        --node-socket /ipc/${NODE_SOCKET_NAME}
        --database /wallet-db
        --listen-address 0.0.0.0
        --testnet /configs/cardano/${CARDANO_NETWORK}/byron-genesis.json
    depends_on:
      - cardano-node
    restart: on-failure
    logging:
      driver: "json-file"
      options:
        compress: "true"
        max-file: "10"
        max-size: "50m"
```

For `mainnet`, set both production network variables and change the wallet command:

```bash
NODE_CARDANO_NETWORK=mainnet
CARDANO_NETWORK=mainnet
```

Change:

```bash
--testnet /configs/cardano/${CARDANO_NETWORK}/byron-genesis.json
```

to:

```bash
--mainnet
```

Bind the Cardano Wallet API to `127.0.0.1`, a private administration network, or an internal service network.

Create or restore the wallet used by PRISM Node after Cardano Wallet reports a healthy network state:

```bash
curl http://127.0.0.1:8090/v2/network/information
```

Generate the mnemonic through your custody process or through the Cardano Wallet CLI on a secured operator machine:

```bash
cardano-wallet mnemonic generate --size 24
```

Restore the wallet through the local Cardano Wallet API. Replace every `word-*` placeholder before running the command:

```bash
curl -X POST "http://127.0.0.1:8090/v2/wallets" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "prism-node-preprod",
    "mnemonic_sentence": [
      "word-01", "word-02", "word-03", "word-04", "word-05", "word-06",
      "word-07", "word-08", "word-09", "word-10", "word-11", "word-12",
      "word-13", "word-14", "word-15", "word-16", "word-17", "word-18",
      "word-19", "word-20", "word-21", "word-22", "word-23", "word-24"
    ],
    "passphrase": "replace-with-wallet-passphrase",
    "address_pool_gap": 20
  }'
```

Record the returned wallet `id` as `NODE_CARDANO_WALLET_ID`. Store the wallet mnemonic and spending passphrase in the production secret system. Store the mnemonic away from `.env-preprod`, CI logs, shell history, and the repository.

Ask Cardano Wallet for an unused address and fund it:

```bash
curl "http://127.0.0.1:8090/v2/wallets/${NODE_CARDANO_WALLET_ID}/addresses?state=unused"
```

Set the first unused address as `NODE_CARDANO_PAYMENT_ADDRESS` after it is funded. On `preprod`, use the Cardano faucet. On `mainnet`, transfer real ADA from the treasury process assigned to the issuer or verifier operator. PRISM Node needs enough ADA for transaction fees and the 1 ADA output described by the PRISM Node deployment guide.

Add the documented PRISM Node Cardano variables to the existing `prism-node` service:

```yaml
  prism-node:
    image: ghcr.io/input-output-hk/prism-node:${PRISM_NODE_VERSION}
    environment:
      NODE_PSQL_HOST: db:5432
      NODE_PSQL_DATABASE: node_db
      NODE_PSQL_USERNAME: ${AGENT_DB_USER:-postgres}
      NODE_PSQL_PASSWORD: ${AGENT_DB_PASSWORD}
      NODE_LEDGER: ${NODE_LEDGER:-cardano}
      NODE_CARDANO_NETWORK: ${NODE_CARDANO_NETWORK:-testnet}
      NODE_CARDANO_WALLET_ID: ${NODE_CARDANO_WALLET_ID}
      NODE_CARDANO_WALLET_PASSPHRASE: ${NODE_CARDANO_WALLET_PASSPHRASE}
      NODE_CARDANO_PAYMENT_ADDRESS: ${NODE_CARDANO_PAYMENT_ADDRESS}
      NODE_CARDANO_WALLET_API_HOST: ${NODE_CARDANO_WALLET_API_HOST:-cardano-wallet}
      NODE_CARDANO_WALLET_API_PORT: ${NODE_CARDANO_WALLET_API_PORT:-8090}
      NODE_CARDANO_DB_SYNC_HOST: ${NODE_CARDANO_DB_SYNC_HOST}
      NODE_CARDANO_DB_SYNC_DATABASE: ${NODE_CARDANO_DB_SYNC_DATABASE:-cexplorer}
      NODE_CARDANO_DB_SYNC_USERNAME: ${NODE_CARDANO_DB_SYNC_USERNAME}
      NODE_CARDANO_DB_SYNC_PASSWORD: ${NODE_CARDANO_DB_SYNC_PASSWORD}
```

The PRISM Node deployment guide states that `NODE_CARDANO_NETWORK` sets PRISM Node's network selector. Wallet and DB Sync use their own network configuration. Cardano node, Cardano Wallet, DB Sync, and PRISM Node must all point to the same Cardano network. A mixed setup can pass process startup and still fail to publish or resolve DID operations. In this tutorial, `NODE_CARDANO_NETWORK=testnet` plus `CARDANO_NETWORK=preprod` is consistent: PRISM Node runs in testnet mode, and the Cardano services use the pre-production testnet files.

## Cardano DB Sync Operations

DB Sync requires its own PostgreSQL database and a connection to the same Cardano node. Use the current DB Sync release package, its release notes, or your infrastructure module for the exact service definition. The service boundary must satisfy these PRISM Node variables:

```bash
NODE_CARDANO_DB_SYNC_HOST=cardano-db-sync-postgres:5432
NODE_CARDANO_DB_SYNC_DATABASE=cexplorer
NODE_CARDANO_DB_SYNC_USERNAME=postgres
NODE_CARDANO_DB_SYNC_PASSWORD=replace-with-db-sync-password
```

A Compose-based lab normally contains:

```yaml
  cardano-db-sync-postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${NODE_CARDANO_DB_SYNC_PASSWORD}
      POSTGRES_DB: ${NODE_CARDANO_DB_SYNC_DATABASE:-cexplorer}
    volumes:
      - cardano_db_sync_postgres:/var/lib/postgresql/data

  cardano-db-sync:
    image: ghcr.io/intersectmbo/cardano-db-sync:${CARDANO_DB_SYNC_VERSION}
    depends_on:
      - cardano-node
      - cardano-db-sync-postgres
    volumes:
      - ${NODE_SOCKET_DIR}:/ipc
      - ${NODE_CONFIGS}:/configs
    environment:
      CARDANO_NODE_SOCKET_PATH: /ipc/${NODE_SOCKET_NAME}
      NETWORK: ${CARDANO_NETWORK}
      POSTGRES_HOST: cardano-db-sync-postgres
      POSTGRES_PORT: 5432
      POSTGRES_DB: ${NODE_CARDANO_DB_SYNC_DATABASE:-cexplorer}
      POSTGRES_USER: ${NODE_CARDANO_DB_SYNC_USERNAME:-postgres}
      POSTGRES_PASSWORD: ${NODE_CARDANO_DB_SYNC_PASSWORD}
```

DB Sync command flags and snapshot restore steps change across releases. Pin the DB Sync image, read its release notes, and validate synchronization on `preprod` before switching to `mainnet`. For `mainnet`, restore from a trusted snapshot or plan for a long initial sync.

Add the volume used by the DB Sync database:

```yaml
volumes:
  cardano_db_sync_postgres:
```

Run DB Sync against the same node socket used by Cardano Wallet. DB Sync publishes synchronization progress in logs and in the `cexplorer` database. Before testing `did:prism` publication, confirm DB Sync is close to chain tip, then create, update, and resolve a DID through the Cloud Agent.

For DB Sync upgrades, read the release notes before changing the image tag. The 13.7.1.0 release notes state that upgrading from 13.7.0.x runs an `epoch` table migration and that compatible 13.7 and 13.6 mainnet snapshots are published. Treat snapshot compatibility as release-specific.

## Preprod and Mainnet Network Selection

Use `preprod` for the first production-style deployment. Cardano documentation describes pre-production as the mature test network that resembles mainnet for release testing. It uses test ADA, so issuer and verifier teams can test DID publication, credential issuance, credential revocation, and verification before moving real funds.

For preprod:

```bash
NODE_CARDANO_NETWORK=testnet
CARDANO_NETWORK=preprod
```

For mainnet:

```bash
NODE_CARDANO_NETWORK=mainnet
CARDANO_NETWORK=mainnet
```

Mainnet changes the risk profile. The operator must fund the PRISM Node wallet with real ADA, protect the Cardano Wallet mnemonic and passphrase, monitor balance, and approve transaction fee spending. Run at least one complete issuer flow on preprod before mainnet: create an issuer `did:prism`, issue a credential to a holder wallet, revoke or suspend it if your use case uses status lists, present it to verifier software, and confirm the verifier separates cryptographic checks from issuer and trust-policy checks.

## Copy Cardano Network Configuration

The Cardano Wallet repository stores network configuration under `configs/cardano`. Copy those files into the project working directory used by the Compose volume:

```bash
mkdir -p ./cardano
git clone --depth 1 https://github.com/cardano-foundation/cardano-wallet ./cardano/cardano-wallet
cp -R ./cardano/cardano-wallet/configs ./cardano/configs
```

Run the refresh script from the `configs/cardano` directory. The script enters each network directory and runs that directory's `download.sh`:

```bash
(
  cd ./cardano/configs/cardano
  ./refresh.sh
)
```

Confirm the `preprod` directory contains the network files required by `cardano-node` and `cardano-wallet`:

```bash
ls ./cardano/configs/cardano/preprod
```

Expected files include:

```text
alonzo-genesis.json
byron-genesis.json
config.json
conway-genesis.json
download.sh
shelley-genesis.json
topology.json
```

Create the data directories used by the Compose file:

```bash
mkdir -p ./cardano/node-db ./cardano/wallet-db ./cardano/ipc
```

## Start Preprod

Validate the Compose configuration before starting services:

```bash
docker compose \
  -p identus-preprod \
  -f ./infrastructure/shared/docker-compose-preprod.yml \
  --env-file ./infrastructure/preprod/.env-preprod \
  config
```

Start the stack:

```bash
./infrastructure/preprod/run.sh \
  -n preprod \
  -b \
  -w \
  -e ./infrastructure/preprod/.env-preprod \
  -p 8000 \
  -d agent.example.test
```

Check the Cloud Agent health endpoint through the same public base URL configured in `REST_SERVICE_URL`:

```bash
curl https://agent.example.test/cloud-agent/_system/health
```

For a lab before TLS termination, use the APISIX host port:

```bash
curl http://localhost:8000/cloud-agent/_system/health
```

Check Cardano Wallet network status:

```bash
curl http://127.0.0.1:8090/v2/network/information
```

Create or restore the Cardano wallet used by PRISM Node, fund it on `preprod`, then update these values in `.env-preprod`:

```bash
NODE_CARDANO_WALLET_ID=replace-after-wallet-create
NODE_CARDANO_WALLET_PASSPHRASE=replace-after-wallet-create
NODE_CARDANO_PAYMENT_ADDRESS=replace-after-wallet-create
```

Restart PRISM Node after changing those values.

## Production Checks

Use these checks before an issuer, verifier, holder wallet, or controller application depends on this deployment:

- Cloud Agent health returns the expected version and the public `REST_SERVICE_URL` uses HTTPS.
- `DIDCOMM_SERVICE_URL` is reachable from holder wallets and mediator services.
- API key authentication is enabled. `ADMIN_TOKEN` and `API_KEY_SALT` are long random values stored in the production secret system.
- `DEFAULT_WALLET_ENABLED=false` for multi-tenant deployments.
- `SECRET_STORAGE_BACKEND=vault` for production. Vault runs with production configuration, and wallet seed backup has an operator procedure.
- PostgreSQL ports stay on private networks. Cloud Agent, PRISM Node, and DB Sync databases have backups and restore tests.
- PRISM Node uses `NODE_CARDANO_NETWORK=testnet` for preprod and `mainnet` for mainnet. Cardano node, Cardano Wallet, and DB Sync use the matching `CARDANO_NETWORK` value, `preprod` or `mainnet`.
- Cardano Wallet has enough ADA for PRISM Node transactions. On `preprod`, use a faucet. On `mainnet`, monitor balance and transaction fees.
- DB Sync is close to chain tip before PRISM Node publication tests.
- A `did:prism` create operation reaches confirmed state, then a resolver can resolve the short-form DID.
- Verifier software separates technical credential checks from issuer, schema, trust registry, and business policy decisions.

Keep the example Docker files under source control for developer learning and reproducible lab work. For mainnet, move secrets, persistent storage, networking, TLS, monitoring, backups, and rollout policy into the deployment system your team uses for production operations.

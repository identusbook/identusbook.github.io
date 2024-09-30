---
title: "Installation - Production Environment"
---
## Overview

A production environment setup requires connecting Hyperledger Identus to the Cardano blockchain as the Verifiable Data Registry (VDR). This is achieved through the `prism-node` component, which abstracts the VDR operations for publishing, resolving, updating, and deactivating Decentralized Identifiers (DIDs).

According to the official documentation:

>The PRISM Node generates a transaction with information about the DID operation and verifies and validates the DID operation before publishing it to the blockchain. Once the transaction gets confirmed on the blockchain, the PRISM Node updates its internal state to reflect the changes.

While our local setup instructs `prism-node` to use a local database as the VDR for testing and development, it lacks the benefits of Cardano's secure and decentralized blockchain for publishing DIDs on-chain. To reach the blockchain, our `prism-node` needs to connect to two components:

1. A full node Cardano wallet to submit transactions to the blockchain
2. Cardano-DB-Sync to read the blockchain through a normalized database interface

When the cloud agent needs to publish, update, or deactivate DIDs, it requests `prism-node` to create and validate a transaction, which is then passed to the `cardano-wallet` for submission to the blockchain. Simultaneously, `prism-node` connects to `cardano-db-sync` to read new blocks, filter DID Prisms published on-chain, and notify the cloud agent when the DID operation reaches a certain number of confirmations.

Production and pre-production installations are similar, with the main difference being that production points to `mainnet` and pre-production to `testnet` for both `cardano-wallet` and `cardano-db-sync`. This is achieved by changing environmental variables passed to the Docker containers.

For a production setup, we recommend additional security measures, including changing default passwords, deactivating unnecessary services, and setting up managed database and secret storage providers with regular backups. While these aspects are beyond the scope of this book, we will provide corresponding notes with our recommendations when appropriate.

## Hardware recommendations

The Hyperledger Identus cloud agent alone doesn't require too much hardware, any instance with 2GB-4GB ram will run it for testing purposes. Of course as you scale in usage you will naturally want more ram available to handle higher concurrent loads.

The Cardano wallet and DB sync components on the other hand are going to need a lot more resources due to the `mainnet` requirements, according to the official documentation, to run a full Cardano node you need:

- 200GB of disk space (for the history of blocks)
- 24GB of RAM (for the current UTxO set)

Of course, on a production environment you may want to run your agent, wallet and db-sync on different machines connected through a VPN or SSH tunnel in order to isolate them and improve security, e.g., your Cardano wallet may be only connecting to a Cardano node, but not exposed to the Internet. You may also reuse and share your Cardano node instance for your wallet and db-sync components, reducing your hardware requirements.

There are many ways to setup your infrastructure and at least in the beginning, the Cardano node is the most resource demanding of all components.

## Configuration

In order to setup `preprod` we recommend copying the config files from the Identus cloud agent and making your own modifications. This is because we will need to modify the docker compose file and that is currently shared among every other type of install such as `local`, `dev` and `multi`. So, to avoid making modifications that will conflict with those defaults, we recommend copying and merging the config into it's own file.

### Preparing base config files

1. Copy `infrastructure/local` config into your `preprod` destination, e.g. standing in the `cloud-agent` root directory:

```bash
cp -rf infrastructure/local infrastructure/preprod
```

2. Copy `infrastructure/shared/docker-compose.yml` for `preprod`:

```bash
cp infrastructure/shared/docker-compose.yml infrastructure/shared/docker-compose-preprod.yml
```

3. Modify `infrastructure/preprod/run.sh` to point to the new docker compose, the `diff` between the files should look like this:

```diff
--- local/run.sh	2024-06-18 13:08:47
+++ preprod/run.sh	2024-09-16 17:03:56
@@ -125,5 +125,5 @@

 PORT=${PORT} NETWORK=${NETWORK} DOCKERHOST=${DOCKERHOST} docker compose \
 	-p ${NAME} \
-	-f ${SCRIPT_DIR}/../shared/docker-compose.yml \
+	-f ${SCRIPT_DIR}/../shared/docker-compose-preprod.yml \
 	--env-file ${ENV_FILE} ${DEBUG} up ${BACKGROUND} ${WAIT}
```

4. Modify `docker-compse-prepod.yml` to disable `postgres` port mapping and to be able to set `DEV_MODE` trough an environment variable:

```diff
--- docker-compose.yml	2024-06-18 13:08:47
+++ docker-compose-preprod.yml	2024-09-16 17:41:36
@@ -15,8 +15,8 @@
       - pg_data_db:/var/lib/postgresql/data
       - ./postgres/init-script.sh:/docker-entrypoint-initdb.d/init-script.sh
       - ./postgres/max_conns.sql:/docker-entrypoint-initdb.d/max_conns.sql
-    ports:
-      - "127.0.0.1:${PG_PORT:-5432}:5432"
+    #ports:
+    #  - "127.0.0.1:${PG_PORT:-5432}:5432"
     healthcheck:
       test: ["CMD", "pg_isready", "-U", "postgres", "-d", "agent"]
       interval: 10s
@@ -96,7 +96,7 @@
       VAULT_ADDR: ${VAULT_ADDR:-http://vault-server:8200}
       VAULT_TOKEN: ${VAULT_DEV_ROOT_TOKEN_ID:-root}
       SECRET_STORAGE_BACKEND: postgres
-      DEV_MODE: true
+      DEV_MODE: ${DEV_MODE:-true}
       DEFAULT_WALLET_ENABLED:
       DEFAULT_WALLET_SEED:
       DEFAULT_WALLET_WEBHOOK_URL:
```

5. Modify `.env` file and add environmental variables for Cardano, please note that `NETWORK` variable conflicts with the prism node network, so we are renaming it to `NODE_CARDANO_NETWORK`, your `.env`  should look like this:

```bash
### IDENTUS
AGENT_VERSION=1.33.0
PRISM_NODE_VERSION=2.2.1
VAULT_DEV_ROOT_TOKEN_ID=root

### CARDANO
NODE_CARDANO_NETWORK=preprod
NODE_DB=$PWD/cardano/node-db
WALLET_DB=$PWD/cardano/wallet-db
NODE_CONFIGS=$PWD/cardano/configs
NODE_SOCKET_NAME=node.socket
NODE_SOCKET_DIR=$PWD/cardano/ipc
NODE_TAG=9.1.1
WALLET_TAG=2024.9.3
WALLET_PORT=8090
WALLET_UI_PORT=8091
```
6. Update your `docker-composer-preprod.yml` to add the Cardano wallet service.

```yaml
---
version: "3.8"

services:
  ##########################
  # Database
  ##########################
  db:
    image: postgres:13
    environment:
      POSTGRES_MULTIPLE_DATABASES: "pollux,connect,agent,node_db"
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - pg_data_db:/var/lib/postgresql/data
      - ./postgres/init-script.sh:/docker-entrypoint-initdb.d/init-script.sh
      - ./postgres/max_conns.sql:/docker-entrypoint-initdb.d/max_conns.sql
    #ports:
    #  - "127.0.0.1:${PG_PORT:-5432}:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres", "-d", "agent"]
      interval: 10s
      timeout: 5s
      retries: 5

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-pgadmin4@pgadmin.org}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-admin}
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

  ##########################
  # Services
  ##########################

  prism-node:
    image: ghcr.io/input-output-hk/prism-node:${PRISM_NODE_VERSION}
    environment:
      NODE_PSQL_HOST: db:5432
      NODE_REFRESH_AND_SUBMIT_PERIOD:
      NODE_MOVE_SCHEDULED_TO_PENDING_PERIOD:
      NODE_WALLET_MAX_TPS:
    depends_on:
      db:
        condition: service_healthy

  vault-server:
    image: hashicorp/vault:latest
    #    ports:
    #      - "8200:8200"
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

  cloud-agent:
    image: ghcr.io/hyperledger/identus-cloud-agent:${AGENT_VERSION}
    environment:
      POLLUX_DB_HOST: db
      POLLUX_DB_PORT: 5432
      POLLUX_DB_NAME: pollux
      POLLUX_DB_USER: postgres
      POLLUX_DB_PASSWORD: postgres
      CONNECT_DB_HOST: db
      CONNECT_DB_PORT: 5432
      CONNECT_DB_NAME: connect
      CONNECT_DB_USER: postgres
      CONNECT_DB_PASSWORD: postgres
      AGENT_DB_HOST: db
      AGENT_DB_PORT: 5432
      AGENT_DB_NAME: agent
      AGENT_DB_USER: postgres
      AGENT_DB_PASSWORD: postgres
      POLLUX_STATUS_LIST_REGISTRY_PUBLIC_URL: http://${DOCKERHOST}:${PORT}/cloud-agent
      DIDCOMM_SERVICE_URL: http://${DOCKERHOST}:${PORT}/didcomm
      REST_SERVICE_URL: http://${DOCKERHOST}:${PORT}/cloud-agent
      PRISM_NODE_HOST: prism-node
      PRISM_NODE_PORT: 50053
      VAULT_ADDR: ${VAULT_ADDR:-http://vault-server:8200}
      VAULT_TOKEN: ${VAULT_DEV_ROOT_TOKEN_ID:-root}
      SECRET_STORAGE_BACKEND: postgres
      DEV_MODE: ${DEV_MODE:-true}
      DEFAULT_WALLET_ENABLED:
      DEFAULT_WALLET_SEED:
      DEFAULT_WALLET_WEBHOOK_URL:
      DEFAULT_WALLET_WEBHOOK_API_KEY:
      DEFAULT_WALLET_AUTH_API_KEY:
      GLOBAL_WEBHOOK_URL:
      GLOBAL_WEBHOOK_API_KEY:
      WEBHOOK_PARALLELISM:
      ADMIN_TOKEN:
      API_KEY_SALT:
      API_KEY_ENABLED:
      API_KEY_AUTHENTICATE_AS_DEFAULT_USER:
      API_KEY_AUTO_PROVISIONING:
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

  ##########################
  # Cardano
  ##########################

  cardano-node:
    image: cardanofoundation/cardano-wallet:${WALLET_TAG}
    environment:
      CARDANO_NODE_SOCKET_PATH: /ipc/${NODE_SOCKET_NAME}
    volumes:
      - ${NODE_DB}:/data
      - ${NODE_SOCKET_DIR}:/ipc
      - ${NODE_CONFIGS}:/configs
    restart: on-failure
    #user: ${USER_ID}:${GROUP_ID}
    logging:
      driver: "json-file"
      options:
        compress: "true"
        max-file: "10"
        max-size: "50m"
    entrypoint: []
    command: >
      cardano-node run --topology /configs/cardano/${NODE_CARDANO_NETWORK}/topology.json
        --database-path /data
        --socket-path /ipc/node.socket
        --config /configs/cardano/${NODE_CARDANO_NETWORK}/config.json
        +RTS -N -A16m -qg -qb -RTS

  cardano-wallet:
    image: cardanofoundation/cardano-wallet:${WALLET_TAG}
    volumes:
      - ${WALLET_DB}:/wallet-db
      - ${NODE_SOCKET_DIR}:/ipc
      - ${NODE_CONFIGS}:/configs
    ports:
      - 127.0.0.1:${WALLET_PORT}:8090
      - 127.0.0.1:${WALLET_UI_PORT}:8091
    environment:
      NETWORK: ${NODE_CARDANO_NETWORK}
    entrypoint: []
    command: >
      cardano-wallet serve
        --node-socket /ipc/${NODE_SOCKET_NAME}
        --database /wallet-db
        --listen-address 0.0.0.0
        --testnet /configs/cardano/${NODE_CARDANO_NETWORK}/byron-genesis.json

    #user: ${USER_ID}:${GROUP_ID}
    restart: on-failure
    logging:
      driver: "json-file"
      options:
        compress: "true"
        max-file: "10"
        max-size: "50m"
  icarus:
    image: piotrstachyra/icarus:v2023-04-14
    ports:
      - 127.0.0.1:4444:4444
    network_mode: "host"
    restart: on-failure

volumes:
  pg_data_db:
  pgadmin:
  node-ipc:
# Temporary commit network setting due to e2e CI bug
# to be enabled later after debugging
#networks:
#  default:
#    name: ${NETWORK}
```

### Cardano wallet

In order to publish DIDs into the VDR, we need to setup our Cardano wallet, the following steps will incorporate the Cardano wallet docker config into our Identus docker config. We will merge whats on [https://github.com/cardano-foundation/cardano-wallet/tree/master/run/preprod/docker]() into our cloud agent setup.

1. Create `snapshot.sh`

```bash
#! /usr/bin/env -S nix shell 'nixpkgs#curl' 'nixpkgs#lz4' 'nixpkgs#gnutar' --command bash
# shellcheck shell=bash

set -euo pipefail

# shellcheck disable=SC1091
source .env

# Define a local db if NODE_DB is not set
if [[ -z "${NODE_DB-}" ]]; then
    LOCAL_NODE_DB=./databases/node-db
    mkdir -p $LOCAL_NODE_DB
    NODE_DB=$LOCAL_NODE_DB
fi

# Clean the db directory
rm -rf "${NODE_DB:?}"/*

echo "Network: $NETWORK"

case "$NETWORK" in
    preprod)
        SNAPSHOT_NAME=$(curl -s https://downloads.csnapshots.io/testnet/testnet-db-snapshot.json| jq -r .[].file_name )
        echo "Snapshot name: $SNAPSHOT_NAME"
        SNAPSHOT_URL="https://downloads.csnapshots.io/testnet/$SNAPSHOT_NAME"
        ;;
    mainnet)
        SNAPSHOT_NAME=$(curl -s https://downloads.csnapshots.io/mainnet/mainnet-db-snapshot.json| jq -r .[].file_name )
        echo "Snapshot name: $SNAPSHOT_NAME"
        SNAPSHOT_URL="https://downloads.csnapshots.io/mainnet/$SNAPSHOT_NAME"
        ;;
    *)
        echo "Error: Invalid network $NETWORK"
        exit 1
        ;;
esac

echo "Downloading the snapshot..."

if [ -n "${LINK_TEST:-}" ]; then
    echo "Link test enabled"
    echo "Snapshot URL: $SNAPSHOT_URL"
    curl -f -LI "$SNAPSHOT_URL" > /dev/null
    curl -r 0-1000000 -SL "$SNAPSHOT_URL" > /dev/null
    exit 0
fi

curl -SL "$SNAPSHOT_URL" | lz4 -c -d - | tar -x -C "$NODE_DB"

mv -f "$NODE_DB"/db/* "$NODE_DB"/
rm -rf "$NODE_DB"/db

echo "Snapshot downloaded and extracted to $NODE_DB"
```

2. Run `snapshot.sh` in order to sync the required ledger snapshot (you will need `curl`, `lz4` and `tar` installed in your system):

```bash
bash snapshot.sh
Network: preprod
Snapshot name: testnet-db-70689600.tar.lz4
Downloading the snapshot...
...
Snapshot downloaded and extracted to ./cardano/node-db
```

3. Copy Cardano wallet [configs](https://github.com/cardano-foundation/cardano-wallet/tree/master/configs) into `./cardano/configs`
4. Run `./refresh.sh` to download the configs.
5. Confirm configs directory for `preprod` look like this:

```bash
ls cardano/configs/cardano/preprod
alonzo-genesis.json  byron-genesis.json   config.json          conway-genesis.json  download.sh          shelley-genesis.json topology.json
```



In this chapter we will discuss how to prepare a production envioronment for an Identus application.

We will discuss:

- Hardware recommendations
- Production configuration and security
- Lock down Docker / Postgres (default password hole, etc)
- SSL
- Multi Tenancy
- Keycloak
- Testnet / Preprod env
- Connecting to Mainnet
- Set up Cardano Wallet
- Key Management with HashiCorp
- Connecting to Cardano
- Running dbSync


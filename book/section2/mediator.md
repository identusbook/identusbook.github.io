# Mediator {#sec-mediator}

## Overview

The Hyperledger Identus Mediator is a DIDComm V2 routing service. It gives mobile wallets, browser wallets, and other agents with changing connectivity a stable endpoint where other agents can send encrypted DIDComm messages.

In a mediated DIDComm flow, the holder wallet first establishes mediation with the mediator. The mediator returns routing information. The holder wallet then advertises that routing information in later DIDComm service entries and connection records. An issuer or verifier sending a DIDComm message to that holder encrypts the message for the holder, wraps it in a DIDComm `forward` message for the mediator, and sends the wrapped message to the mediator endpoint. The mediator can decrypt the outer routing envelope, but it cannot read the holder's encrypted message body.

The mediator stores pending holder messages in MongoDB. The holder wallet later uses DIDComm Message Pickup to check queue status, request delivery, and confirm received messages. This model lets an issuer or verifier send messages when the holder wallet is offline, without exposing the holder's device address to every connection.

The mediator is not a Verifiable Data Registry and does not publish DIDs. It handles DIDComm transport and storage. PRISM Node handles ledger-backed `did:prism` operations. Cloud Agent and Edge Agent SDKs use the mediator when their DIDComm counterparty needs a stable relay endpoint.

## Protocol surface

The current Mediator repository documents support for these DIDComm protocols:

- `https://didcomm.org/routing/2.0`
- `https://didcomm.org/coordinate-mediation/2.0`
- `https://didcomm.org/messagepickup/3.0`
- `https://didcomm.org/trust-ping/2.0`
- `https://didcomm.org/report-problem/2.0`
- `https://didcomm.org/basicmessage/2.0`

For setup, the important protocols are Coordinate Mediation, Routing, and Message Pickup. Coordinate Mediation lets a recipient wallet request mediation and register recipient keys. Routing defines the `forward` message used to send traffic through the mediator. Message Pickup lets the recipient wallet retrieve queued messages.

## Version selection

Check both release streams before pinning a mediator image. The current Identus Platform `v2.16` release lists Mediator `1.2.0`. The standalone Mediator repository lists `v1.2.1` as the latest stable Mediator release.

This chapter uses `MEDIATOR_VERSION=1.2.1` for the standalone mediator setup. If you are reproducing the full Identus Platform `v2.16` component set, use `MEDIATOR_VERSION=1.2.0` instead.

## Prerequisites

Install Git, Docker, and Docker Compose before running the mediator stack.

```bash
git --version
docker --version
docker compose version
```

Each command should print a version. If one command fails, install or repair that tool before starting the mediator.

## Clone the mediator repository

Clone the current repository:

```bash
git clone https://github.com/hyperledger-identus/mediator identus-mediator
cd identus-mediator
```

Older Identus material may use `https://github.com/hyperledger/identus-mediator`. Current Identus repositories live under the `hyperledger-identus` GitHub organization.

## Local configuration files

The official repository already includes `docker-compose.yml` and `initdb.js`. You do not need to create either file when you run from the cloned repository. Read both files before changing the mediator deployment. Those files define the image, database, identity keys, public DIDComm endpoints, and health check.

The current local Compose file has this shape:

```yaml
services:
  mongo:
    image: mongo:6.0
    ports:
      - "27017:27017"
    command: ["--auth"]
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin
      - MONGO_INITDB_DATABASE=mediator
    volumes:
      - ./initdb.js:/docker-entrypoint-initdb.d/initdb.js

  identus-mediator:
    image: docker.io/hyperledgeridentus/identus-mediator:${MEDIATOR_VERSION:-latest}
    ports:
      - "8080:8080"
    environment:
      - KEY_AGREEMENT_D=Z6D8LduZgZ6LnrOHPrMTS6uU2u5Btsrk1SGs4fn8M7c
      - KEY_AGREEMENT_X=Sr4SkIskjN_VdKTn0zkjYbhGTWArdUNE4j_DmUpnQGw
      - KEY_AUTHENTICATION_D=INXCnxFEl0atLIIQYruHzGd5sUivMRyQOzu87qVerug
      - KEY_AUTHENTICATION_X=MBjnXZxkMcoQVVL21hahWAw43RuAG-i64ipbeKKqwoA
      - SERVICE_ENDPOINTS=${SERVICE_ENDPOINTS:-http://localhost:8080;ws://localhost:8080/ws}
      - MONGODB_USER=admin
      - MONGODB_PASSWORD=admin
      - MONGODB_PROTOCOL=mongodb
      - MONGODB_HOST=mongo
      - MONGODB_PORT=27017
      - MONGODB_DB_NAME=mediator
    depends_on:
      - mongo
    healthcheck:
      test: curl --fail http://localhost:8080/health || exit 1
      interval: 30s
      timeout: 8s
      retries: 3
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

The `mongo` service stores mediation accounts and queued messages. The local file exposes MongoDB on host port `27017` for debugging. Wallets and Cloud Agents do not need that host port. Production deployments should keep MongoDB private to the deployment network.

The `identus-mediator` service runs the mediator image from Docker Hub. `MEDIATOR_VERSION` selects the image tag. The `ports` mapping exposes the mediator HTTP and WebSocket service on host port `8080`. The mediator uses `SERVICE_ENDPOINTS` when it builds its Peer DID and out-of-band invitation, so this value must name an endpoint that the holder wallet can reach.

The demo key values in the local Compose file create a stable mediator identity for local development. Do not reuse those keys for a shared or production mediator. Generate a new X25519 key pair for key agreement and a new Ed25519 key pair for authentication, then provide the four JWK fields through deployment configuration.

The Mongo initialization file creates the database user, collections, indexes, and TTL rule:

```js
db.createUser({
    user: "admin",
    pwd: "admin",
    roles: [
        { role: "readWrite", db: "mediator" }
    ]
});

const database = 'mediator';
const collectionDidAccount = 'user.account';
const collectionMessages = 'messages';
const collectionMessagesSend = 'messages.outbound';

use(database);

db.createCollection(collectionDidAccount);
db.createCollection(collectionMessages);
db.createCollection(collectionMessagesSend);

db.getCollection(collectionDidAccount).createIndex({ 'did': 1 }, { unique: true });
db.getCollection(collectionDidAccount).createIndex({ 'alias': 1 }, { unique: true, partialFilterExpression: { "alias.0": { $exists: true } } });
db.getCollection(collectionDidAccount).createIndex({ "messagesRef.hash": 1, "messagesRef.recipient": 1 });

const expireAfterSeconds = 7 * 24 * 60 * 60;
db.getCollection(collectionMessages).createIndex(
    { ts: 1 },
    {
        name: "message-ttl-index",
        partialFilterExpression: { "message_type" : "Mediator" },
        expireAfterSeconds: expireAfterSeconds
    }
)
```

`user.account` stores mediation account records keyed by DID. `messages` stores mediator protocol messages and queued user payload references. `messages.outbound` stores outbound message records. The TTL index matches records whose `message_type` is `Mediator`; queued user messages stay until the recipient retrieves them and confirms receipt.

A local `.env` file can pin the mediator image and public endpoints:

```bash
MEDIATOR_VERSION=1.2.1
SERVICE_ENDPOINTS=http://localhost:8080;ws://localhost:8080/ws
```

Use a LAN IP address or public domain in `SERVICE_ENDPOINTS` when a mobile wallet must connect from outside the host machine. Keep the URL scheme and port aligned with the endpoint you expose through Docker, a reverse proxy, or a load balancer.

## Start the mediator

The repository includes the local `docker-compose.yml`. It starts two services:

- `mongo`, the MongoDB database used for accounts and queued messages.
- `identus-mediator`, the JVM mediator service exposed on port `8080`.

The current Compose file maps host port `8080` to the mediator container. Check that no other local service uses that port before startup:

```bash
docker ps --format '{{.Names}} {{.Ports}}' | grep 8080
```

If the command prints another container using `8080`, stop that service or adapt the Compose port mapping before starting the mediator. Keep the exposed host port and `SERVICE_ENDPOINTS` in agreement, since wallets read `SERVICE_ENDPOINTS` from the mediator's DIDComm invitation.

Run the command for your operating system from the repository root.

macOS:

```bash
MEDIATOR_VERSION=1.2.1 SERVICE_ENDPOINTS="http://$(ipconfig getifaddr $(route get default | grep interface | awk '{print $2}')):8080;ws://$(ipconfig getifaddr $(route get default | grep interface | awk '{print $2}')):8080/ws" docker compose up -d
```

Linux:

```bash
MEDIATOR_VERSION=1.2.1 SERVICE_ENDPOINTS="http://$(ip addr show $(ip route show default | awk '/default/ {print $5}') | grep 'inet ' | awk '{print $2}' | cut -d/ -f1):8080;ws://$(ip addr show $(ip route show default | awk '/default/ {print $5}') | grep 'inet ' | awk '{print $2}' | cut -d/ -f1):8080/ws" docker compose up -d
```

The `SERVICE_ENDPOINTS` value becomes part of the mediator's Peer DID service entries and out-of-band invitation. Use an address that the wallet can reach. A browser wallet running on the same host can use `localhost`; a mobile wallet on the same network needs the host machine's LAN IP address or a public domain. For production deployments, use HTTPS and WSS endpoints.

## Check the running mediator

Check the containers:

```bash
docker compose ps
```

The mediator service defines a Docker health check against `/health`. After startup, Docker should report `identus-mediator` as healthy.

Check the health endpoint:

```bash
curl -i http://localhost:8080/health
```

Expected result:

```text
HTTP/1.1 200 OK
```

Check the running mediator version:

```bash
curl http://localhost:8080/version
```

Expected result:

```text
1.2.1
```

Fetch the mediator Peer DID:

```bash
curl http://localhost:8080/did
```

The response should start with `did:peer:`. Holder wallet software uses this DID as the mediator DID when it starts mediation.

Fetch the mediation invitation:

```bash
curl http://localhost:8080/invitation
```

The response is a DIDComm out-of-band invitation. Its `from` field is the mediator Peer DID. Its body should include `goal_code` set to `request-mediate` and `accept` containing `didcomm/v2`.

To get the URL-encoded invitation used for QR-code flows, call:

```bash
curl http://localhost:8080/invitationOOB
```

The response should contain an `_oob=` parameter.

## Configuration notes

The local Compose file includes demo identity keys. Do not use them outside local development. A production mediator needs its own X25519 key agreement key and Ed25519 authentication key. The Mediator repository includes a key generation guide for `KEY_AGREEMENT_D`, `KEY_AGREEMENT_X`, `KEY_AUTHENTICATION_D`, and `KEY_AUTHENTICATION_X` values in JWK-compatible form.

MongoDB can be configured with individual variables:

```text
MONGODB_PROTOCOL
MONGODB_HOST
MONGODB_PORT
MONGODB_USER
MONGODB_PASSWORD
MONGODB_DB_NAME
```

Mediator `1.2.0` added `MONGODB_CONNECTION_STRING`. If present, this full connection string takes precedence over the individual MongoDB variables.

The mediator stores two message categories. `Mediator` messages cover mediation setup, keylist operations, pickup requests, and other mediator protocol traffic. These records can expire through the TTL index configured in `initdb.js`. `User` messages are encrypted payloads waiting for holder pickup. The mediator keeps them until the recipient retrieves them and confirms receipt through Message Pickup.

## Stop the mediator

Stop the stack without deleting the database volume:

```bash
docker compose down
```

Remove the containers and local MongoDB data:

```bash
docker compose down -v
```

Use `-v` when you want a clean local mediator state. Omit it when you want to keep registered mediation accounts and queued development messages across restarts.

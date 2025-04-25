# Mediator {#sec-mediators}

## Overview

The `mediator` is a service that enables private and secure message routing between peers. Its primary function is to relay encrypted messages between agents using [DIDComm V2](../section3/didcomm.md) set of communication protocols. Since each participant wallet is Self-Sovereign, mediators help route messages to their intended recipient without any details of each peer besides the agreed public keys over the initial connection protocol, upon which they both create a `did:peer`.  Encrypted messages can be sent to peers even if they are not online by relaying on the mediator to manage a queue for the offline party until they are able to connect and retrieve them.

Think of a mediator like a set of Post Office mailboxes. Each mailbox has a mailing address (*a `DID`*).  Each mailbox can hold an stack of incoming mail [Encrypted DIDComm V2 messages](../section3/didcomm.md) for its owner.  Each mailbox also keeps a list of cryptographic [Connections](../section3/connections.md) with whom its owner can send or receive mail with.  In this way, Identus mediators facilitate all types of important transactions, including routing both messages, Verifiable Credential issuance, Credential Exchange or any custom message for your own use.

The key insight for this piece of infrastructure is that each agent can define their own `mediator` to be reachable at, so whenever they establish a `did:peer` with another agent, each peer defines their own reachable `DIDCOMM` endpoint, each one will tell the other their `mediator` as the way to send messages addressed to them.

### Why use a Mediator?

Why do we need a `mediator` in the first place?  If participants are Self-Sovereign, then why not connect an edge wallet directly to another edge wallet?  

SSI participants are connected through various types of devices and network conditions. [The Cloud Agent](../section2/installation-local.md), is expected to be online and available all the time, as it's usually hosted in a cloud infrastructure somewhere. However users who keep their wallets on mobile devices, or laptop computers are only online sporadically and can't be expected to have a reliable connection to the Internet at all times.  This is where mediators earn their keep. If `Alice` were to send a message to `Bob`, who happens to be offline at the moment, the message could not be delivered, plus `Alice` would also have to know sensitive data about how to contact `Bob`. If `Bob` changes devices, they would have to tell `Alice`, and all other peer connections. If `Alice` instead, sends the message through the `mediator`, it can be held securely and retrieved the next time `Bob` connects to the Internet. Mediators provide reliability between [Connections](../section3/connections.md), without compromising privacy or security.  

## Set up a Mediator

As usual, we use Docker to setup our infrastructure, `mediator` is no different and here is an example `docker-compose.yml`, `mongo-initdb.js` and `.env` files, you should put all 3 files in a folder.

`docker-compose.yml`
```yml
services:
  ################
  ### MEDIATOR ###
  ################

  mediator-mongo:
    image: mongo:6.0
    command: [ "--auth" ]
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin
      - MONGO_INITDB_DATABASE=mediator
    volumes:
      - ./mongo-initdb.js:/docker-entrypoint-initdb.d/initdb.js

  mediator:
    image: docker.io/identus/identus-mediator:${MEDIATOR_VERSION:latest}
    ports:
      - "${MEDIATOR_PORT:-8080}:8080"
    environment:
      # Creates the identity:
      # These keys are for demo purpose only for production deployments generate keys
      # Please follow the README file in the Mediator repository for guidelines on How to generate JWK format keys
      # KEY_AGREEMENT KEY_AUTHENTICATION are using format JOSE(JWK) OKP type base64urlsafe encoded keys
      - KEY_AGREEMENT_D=${KEY_AGREEMENT_D:-Z6D8LduZgZ6LnrOHPrMTS6uU2u5Btsrk1SGs4fn8M7c}
      - KEY_AGREEMENT_X=${KEY_AGREEMENT_X:-Sr4SkIskjN_VdKTn0zkjYbhGTWArdUNE4j_DmUpnQGw}
      - KEY_AUTHENTICATION_D=${KEY_AUTHENTICATION_D:-INXCnxFEl0atLIIQYruHzGd5sUivMRyQOzu87qVerug}
      - KEY_AUTHENTICATION_X=${KEY_AUTHENTICATION_X:-MBjnXZxkMcoQVVL21hahWAw43RuAG-i64ipbeKKqwoA}
      - SERVICE_ENDPOINTS=${SERVICE_ENDPOINTS:-http://identus-mediator:8080;ws://identus-mediator:8080/ws}
      - MONGODB_USER=admin
      - MONGODB_PASSWORD=admin
      - MONGODB_PROTOCOL=mongodb
      - MONGODB_HOST=mediator-mongo
      - MONGODB_PORT=27017
      - MONGODB_DB_NAME=mediator
    depends_on:
      - "mediator-mongo"
```

`mongo-initdb.js`
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

// The current database to use.
use(database);

// Create  collections.
db.createCollection(collectionDidAccount);
db.createCollection(collectionMessages);
db.createCollection(collectionMessagesSend);

//create index
db.getCollection(collectionDidAccount).createIndex({ 'did': 1 }, { unique: true });
// Only enforce uniqueness on non-empty arrays
db.getCollection(collectionDidAccount).createIndex({ 'alias': 1 }, { unique: true, partialFilterExpression: { "alias.0": { $exists: true } } });
db.getCollection(collectionDidAccount).createIndex({ "messagesRef.hash": 1, "messagesRef.recipient": 1 });

// There are 2 message types  `Mediator`  and `User` Please follow the Readme for more details in the section Mediator storage
const expireAfterSeconds = 7 * 24 * 60 * 60; // 7 day * 24 hours * 60 minutes * 60 seconds
db.getCollection(collectionMessages).createIndex(
    { ts: 1 },
    {
        name: "message-ttl-index",
        partialFilterExpression: { "message_type": "Mediator" },
        expireAfterSeconds: expireAfterSeconds
    }
)
```

`.env`
```bash
### MEDIATOR
MEDIATOR_PORT=127.0.0.1:8080
MEDIATOR_VERSION=1.0.0
KEY_AGREEMENT_D=xxx
KEY_AGREEMENT_X=xxx
KEY_AUTHENTICATION_D=xxx
KEY_AUTHENTICATION_X=xxx
SERVICE_ENDPOINTS="https://example.com"
```

Your `.env` file will hold all the important bits, lets trough them.

`MEDIATOR_PORT` will define which port the container will be reachable at, I have also bound it to the local interface because I want to proxy this service over SSL with `apache` or `nginx`.

`MEDIATOR_VERSION` will default to `latest` but you can also define a specific version, this is important because sometimes there are breaking changes and your `cloud-agent` or SDK will be compatible with a specific version of the `mediator` that may not be the latest.

`KEY_AGREEMENT_D`, `KEY_AGREEMENT_X`, `KEY_AUTHENTICATION_D`, `KEY_AUTHENTICATION_X` are all the secret keys that are needed for the mediator to generate `DIDs`, in this case the `mediator` generates a `did:peer` so others can route messages trough it. Follow the latest updated [guide to generate keys](https://github.com/hyperledger-identus/mediator/blob/main/mediator-identity-key-generation.md).

`SERVICE_ENDPOINTS` is how you define the *public url* of your `mediator`, this is very important, this is how other agents will reach this service, it can be the public IP or domain, you can include specific ports as well. Make sure `MEDIATOR_PORT` and `SERVICE_ENDPOINTS` are in agreement, for example, if you bound `MEDIATOR_PORT` to the local interface and your `SERVICE_ENDPOINTS` defines a public URL, you need to proxy your local port trough `apache`, `nginx` or similar, if you decide to use expose a public IP, all you need to do is to define both to the same IP and port. This is all contextual of how you run your infrastructure.

Once your done setting the environment you can run it with:

```bash
docker compose up -d
```

Your mediator should be reachable trough the same endpoint defined in `SERVICE_ENDPOINTS`.

If successful, you should see a simple web page with an invite and a QR code for your `mediator`.
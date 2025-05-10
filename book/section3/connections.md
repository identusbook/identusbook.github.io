# Connections {#sec-connections}

## Overview

Now that we have a better understanding of Wallets and DIDs, it's time to embark on our first interaction. In this chapter we are going to explore conceptually what a Connection means in SSI, take a deep dive into DID Peers, explain how they work and why they are needed for secure connections, dissect Out of Band invites and finally hands on example code to achieve connecting edge client to an agent.

Before we move forward we highly recommend to at least read the basic [Connection tutorial](https://hyperledger.github.io/identus-docs/tutorials/connections/connection) on the official [Identus documentation](https://hyperledger.github.io/identus-docs/).

## Connections in Self-Sovereign Identity

Connections are fundamental to establishing *trusted interactions* between peers. They enable secure and verifiable communication, allowing entities to exchange credentials and proofs in a decentralized manner. This relationship is established using a specific decentralized identifier standard (**Peer DID**) and is governed by a protocol (**DIDComm**) that ensure the authenticity, integrity, and privacy of the interactions between the connected parties.

There are three roles in an SSI connection:

- **Inviter**: The entity that initiates the connection by sending an invitation.
- **Invitee**: The entity that receives the invitation and responds with a connection request.
- **Mediator**: An intermediary that facilitates message delivery between entities, especially when one or both parties may not always be online.

::: {.callout-note}
We will cover [mediators](/section4/mediator.html) in detail later on. For now what you need to understand is that they are used as a service to relay messages between peers, they will store messages and deliver them whenever a peer comes back online, connects to the mediator and fetches their messages. 
:::

## PeerDIDs

They are a special kind of decentralized identifier with some unique properties that allow them to be perfect for use in order to establish private and secure communications between peers.

DID Documents such as PrismDIDs are meant to be publicly available and resolvable by arbitrary parties, therefor storing them in a VDR such as Cardano blockchain is an excellent way to achieve this requirement in a reliable way. 

However, when Alice and Bob want to interact with each other, only two parties care about the details of that connection: Alice and Bob. Instead of arbitrary parties needing to resolve their DIDs, only Alice and Bob do. Thus, PeerDIDs essentially describe a key-pair to encrypt and sign data to and from Alice and Bob, routed trough their preferred mediators, e.g. When Alice accepts an invite from Bob and they engage the connection protocol, Alice generates a PeerDID that allows her to encrypt and sign data routed trough Bob's mediator (mediator Y) that only Bob can decrypt, and vice versa, Bob will generate a PeerDID that allows him to encrypt and sign data routed trough Alice's preferred mediator (mediator X) that only Alice can decrypt.

The key benefits of PeerDIDs are:

1. Decentralized by nature.
2. No transaction cost on blockchain.
3. Private (only the concerned parties know about them).
4. Reusable without any reliance on the internet, with no degradation of trust. (adheres to the principles of [local-first](https://www.inkandswitch.com/local-first/) and [offline-first](https://offlinefirst.org))

Lets resolve a PeerDID, we call resolve to the unpacking and parsing of a DID in order to read its content and use the DID for the interactions that we need to achieve. For this example we will resolve a PeerDID from [Atala's mediator sandbox](https://sandbox-mediator.atalaprism.io).

```bash
curl https://sandbox-mediator.atalaprism.io/did

did:peer:2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0
```
We can see that the mediator returned a PeerDID when we send a `GET` request to the `/did` endpoint. In order to resolve this DID we can use an [Universal Resolver](https://dev.uniresolver.io) website for this example:

```bash
curl https://dev.uniresolver.io/1.0/identifiers/did:peer:2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0
```
```json
{
  "@context": "https://w3id.org/did-resolution/v1",
  "didDocument": {
    "@context": [
      "https://www.w3.org/ns/did/v1",
      "https://w3id.org/security/multikey/v1",
      {
        "@base": "did:peer:2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0"
      }
    ],
    "id": "did:peer:2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0",
    "verificationMethod": [
      {
        "id": "#key-2",
        "type": "Multikey",
        "controller": "did:peer:2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0",
        "publicKeyMultibase": "z6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd"
      },
      {
        "id": "#key-1",
        "type": "Multikey",
        "controller": "did:peer:2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0",
        "publicKeyMultibase": "z6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y"
      }
    ],
    "keyAgreement": [
      "#key-1"
    ],
    "authentication": [
      "#key-2"
    ],
    "assertionMethod": [
      "#key-2"
    ],
    "service": [
      {
        "serviceEndpoint": {
          "uri": "https://sandbox-mediator.atalaprism.io",
          "accept": [
            "didcomm/v2"
          ]
        },
        "type": "DIDCommMessaging",
        "id": "#service"
      },
      {
        "serviceEndpoint": {
          "uri": "wss://sandbox-mediator.atalaprism.io/ws",
          "accept": [
            "didcomm/v2"
          ]
        },
        "type": "DIDCommMessaging",
        "id": "#service-1"
      }
    ]
  },
  "didResolutionMetadata": {
    "contentType": "application/did+ld+json",
    "pattern": "^(did:peer:.+)$",
    "driverUrl": "http://uni-resolver-driver-did-uport:8081/1.0/identifiers/",
    "duration": 4,
    "driverDuration": 4,
    "did": {
      "didString": "did:peer:2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0",
      "methodSpecificId": "2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0",
      "method": "peer"
    }
  },
  "didDocumentMetadata": {}
}
```

This is what a PeerDID looks like when resolved, please bear in mind that the JSON-LD context for `did-resolution` and extra `didResolutionMetadata` entry are added by the resolver and the actual isolated PeerDID is only what we see inside the `didDocument` payload.

In simple terms, a PeerDID is essentially a JSON payload that contains a set of keys and an optional service endpoints, because this is a mediator PeerDID, it contains service endpoints for `DIDCommMessaging`, in this particular case, you can see it contains two of them, one over regular `https` and other trough `websockets`.


To go deeper in your understanding of PeerDIDs please refer to the full [Peer DID Method Specification](https://identity.foundation/peer-did-method-spec). In the Hyperledger Identus ecosystem, only PeerDIDs method 2 are supported at the time of this writing.

## Out of Band invites

Out of Band (OOB) invites are the entry point for some protocols to take place, they usually are encoded in either a JSON payload or a URL and are distributed "out of band", usually over QR codes, but could be distributed over any medium (Bluetooth, NFC, etc). They gather all required information for one peer to start interacting with another and you can think of them as a way to advertise "coordinates" for anyone that would like to establish an interaction to the inviter.

Following the same example as before, we can see that the sandbox mediator also delivers an OOB invite in a URL form.

```bash
https://sandbox-mediator.atalaprism.io?_oob=eyJpZCI6ImExNTY4YzEyLTBjZGMtNDY0Ny05ZGM2LWE1YWFkYTZmODI0NyIsInR5cGUiOiJodHRwczovL2RpZGNvbW0ub3JnL291dC1vZi1iYW5kLzIuMC9pbnZpdGF0aW9uIiwiZnJvbSI6ImRpZDpwZWVyOjIuRXo2TFNnaHdTRTQzN3duREUxcHQzWDZoVkRVUXpTanNIemlucFgzWEZ2TWpSQW03eS5WejZNa2hoMWU1Q0VZWXE2SkJVY1RaNkNwMnJhbkNXUnJ2N1lheDNMZTRONTlSNmRkLlNleUowSWpvaVpHMGlMQ0p6SWpwN0luVnlhU0k2SW1oMGRIQnpPaTh2YzJGdVpHSnZlQzF0WldScFlYUnZjaTVoZEdGc1lYQnlhWE50TG1sdklpd2lZU0k2V3lKa2FXUmpiMjF0TDNZeUlsMTlmUS5TZXlKMElqb2laRzBpTENKeklqcDdJblZ5YVNJNkluZHpjem92TDNOaGJtUmliM2d0YldWa2FXRjBiM0l1WVhSaGJHRndjbWx6YlM1cGJ5OTNjeUlzSW1FaU9sc2laR2xrWTI5dGJTOTJNaUpkZlgwIiwiYm9keSI6eyJnb2FsX2NvZGUiOiJyZXF1ZXN0LW1lZGlhdGUiLCJnb2FsIjoiUmVxdWVzdE1lZGlhdGUiLCJhY2NlcHQiOlsiZGlkY29tbS92MiJdfSwidHlwIjoiYXBwbGljYXRpb24vZGlkY29tbS1wbGFpbitqc29uIn0
```

If we decode the value of the `_oob` query variable from `base64` we get the `json` payload

```json
{
  "id" : "a1568c12-0cdc-4647-9dc6-a5aada6f8247",
  "type" : "https://didcomm.org/out-of-band/2.0/invitation",
  "from" : "did:peer:2.Ez6LSghwSE437wnDE1pt3X6hVDUQzSjsHzinpX3XFvMjRAm7y.Vz6Mkhh1e5CEYYq6JBUcTZ6Cp2ranCWRrv7Yax3Le4N59R6dd.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHBzOi8vc2FuZGJveC1tZWRpYXRvci5hdGFsYXByaXNtLmlvIiwiYSI6WyJkaWRjb21tL3YyIl19fQ.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6IndzczovL3NhbmRib3gtbWVkaWF0b3IuYXRhbGFwcmlzbS5pby93cyIsImEiOlsiZGlkY29tbS92MiJdfX0",
  "body" : {
    "goal_code" : "request-mediate",
    "goal" : "RequestMediate",
    "accept" : [
      "didcomm/v2"
    ]
  },
  "typ" : "application/didcomm-plain+json"
}
```
As you can see, an Out of Band invite is really just a way to package a PeerDID and signaling that it can be used for a particular interaction. In this case, for a `RequestMediate` goal over `DIDComm`.

As a refresher from what we covered, a PeerDID is a way to package a set of keys and optional service endpoints, and so, *because this an OOB invite from a mediator*, this invite has everything you need (a PeerDID and set of service endpoints) to establish this service as your mediator.

## Connecting two peers

Now, lets issue another kind of Out of Band invite, one from the cloud agent in order to connect.

The Cloud Agent can generate Out of Band invites, this invite then can be parsed by another peer (say another Cloud Agent or Edge Client) and use it to establish a connection, the end result should be a DID Peer on both sides that allow them to send messages to each other over `DIDComm`.

So, our first step is to generate the invite:

```bash
curl --location 'http://127.0.0.1:8080/cloud-agent/connections' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--data '{"label": "test"}'
```

::: {.callout-note}
The only parameter we can change when generating an invite is the `label`, this is optional and it is a simple string that you can use to identify the connection later, a good idea would be to use a `uuid` that you generate and manage on your systems, or could be an alias or the reason for the connection, this is all contextual to the interaction and use case so in our case we will go with "test".
:::

The Cloud Agent will respond with a payload that should look like this:

```json
{
    "connectionId": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
    "thid": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
    "label": "test",
    "role": "Inviter",
    "state": "InvitationGenerated",
    "invitation": {
        "id": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
        "type": "https://didcomm.org/out-of-band/2.0/invitation",
        "from": "did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ",
        "invitationUrl": "https://my.domain.com/path?_oob=eyJpZCI6ImZiMzZlZGRkLWQ1MWUtNDJjZi1hNmZlLWU3NmQyZTYzOGI3MCIsInR5cGUiOiJodHRwczovL2RpZGNvbW0ub3JnL291dC1vZi1iYW5kLzIuMC9pbnZpdGF0aW9uIiwiZnJvbSI6ImRpZDpwZWVyOjIuRXo2TFNnWTZZNjdtSjc1WUNaZlpZeFlFUFFKWnMzdmFFZzJDYzkxdnBwb1RBN2Nwai5WejZNa3BYN0g3U05BNm9vRzVzbm4yTXpneW9SYWRFWnRzak5TTDF4N0hpaUxrcXlWLlNleUowSWpvaVpHMGlMQ0p6SWpwN0luVnlhU0k2SW1oMGRIQTZMeTlvYjNOMExtUnZZMnRsY2k1cGJuUmxjbTVoYkRvNE1EZ3dMMlJwWkdOdmJXMGlMQ0p5SWpwYlhTd2lZU0k2V3lKa2FXUmpiMjF0TDNZeUlsMTlmUSIsImJvZHkiOnsiYWNjZXB0IjpbXX19"
    },
    "createdAt": "2025-01-04T12:37:37.059649293Z",
    "metaRetries": 5,
    "self": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
    "kind": "Connection"
}
```

Once the connection invite is created you can fetch it's details by a `GET`request passing the `connectionId`:

```bash
curl --location 'http://127.0.0.1:8080/cloud-agent/connections/fb36eddd-d51e-42cf-a6fe-e76d2e638b70' \
--header 'Accept: application/json' \
```

You should get back the same payload as when it was created unless something changed, like the `state`:

```json
{
    "connectionId": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
    "thid": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
    "label": "test",
    "role": "Inviter",
    "state": "InvitationGenerated",
    "invitation": {
        "id": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
        "type": "https://didcomm.org/out-of-band/2.0/invitation",
        "from": "did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ",
        "invitationUrl": "https://my.domain.com/path?_oob=eyJpZCI6ImZiMzZlZGRkLWQ1MWUtNDJjZi1hNmZlLWU3NmQyZTYzOGI3MCIsInR5cGUiOiJodHRwczovL2RpZGNvbW0ub3JnL291dC1vZi1iYW5kLzIuMC9pbnZpdGF0aW9uIiwiZnJvbSI6ImRpZDpwZWVyOjIuRXo2TFNnWTZZNjdtSjc1WUNaZlpZeFlFUFFKWnMzdmFFZzJDYzkxdnBwb1RBN2Nwai5WejZNa3BYN0g3U05BNm9vRzVzbm4yTXpneW9SYWRFWnRzak5TTDF4N0hpaUxrcXlWLlNleUowSWpvaVpHMGlMQ0p6SWpwN0luVnlhU0k2SW1oMGRIQTZMeTlvYjNOMExtUnZZMnRsY2k1cGJuUmxjbTVoYkRvNE1EZ3dMMlJwWkdOdmJXMGlMQ0p5SWpwYlhTd2lZU0k2V3lKa2FXUmpiMjF0TDNZeUlsMTlmUSIsImJvZHkiOnsiYWNjZXB0IjpbXX19"
    },
    "createdAt": "2025-01-04T12:37:37.059649Z",
    "metaRetries": 5,
    "self": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
    "kind": "Connection"
}
```

Lets dig a bit deeper on what this payload represents. 

This invite payload contains some important metadata:

- **connectionId**: The unique identifier of the connection resource, used to fetch the connection details.
- **thid**: The unique identifier of the *thread* this connection record belongs to. The value will identical on both sides of the connection (inviter and invitee).
- **label**: A human readable alias for the connection.
- **role**: The Cloud Agent role on this connection, either `Inviter` or `Invitee`.
- **state**: The current status of this connection, note this is contextual to the Cloud Agent role, so as `Inviter` the states could be: `InvitationGenerated`, `ConnectionRequestReceived`, `ConnectionResponsePending`, `ConnectionResponseSent`. But is also possible for the Cloud Agent to parse someone else's invitation, in that case the Cloud Agent will generate a connection with the `Invitee` role and the possible states for that role are: `InvitationReceived`, `ConnectionRequestPending`, `ConnectionRequestSent`, `ConnectionResponseReceived`.
- **invitation**: The `DIDComm` invitation details.
- **createdAt**: Date and time when this connection was created or received.
- **metaRetries**: The maximum background processing attempts remaining for this record.
- **self**: The reference to the connection resource.
- **kind**: The type of object returned. In this case a `Connection`.

Now lets unpack the `invitation` details.

- **id**: The unique identifier of the invitation. It should be used as parent thread ID (`pthid`) for the Connection Request message that follows.
- **type**: The DIDComm Message Type URI (MTURI) the invitation message complies with.
- **from**: The DID representing the sender to be used by recipients for future interactions.
- **invitationUrl**: The invitation message encoded as a URL.

Lets resolve the `from` DID:

```bash
curl https://dev.uniresolver.io/1.0/identifiers/did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ
```
```json
{
  "@context": "https://w3id.org/did-resolution/v1",
  "didDocument": {
    "@context": [
      "https://www.w3.org/ns/did/v1",
      "https://w3id.org/security/multikey/v1",
      {
        "@base": "did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ"
      }
    ],
    "id": "did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ",
    "verificationMethod": [
      {
        "id": "#key-2",
        "type": "Multikey",
        "controller": "did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ",
        "publicKeyMultibase": "z6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV"
      },
      {
        "id": "#key-1",
        "type": "Multikey",
        "controller": "did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ",
        "publicKeyMultibase": "z6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj"
      }
    ],
    "keyAgreement": [
      "#key-1"
    ],
    "authentication": [
      "#key-2"
    ],
    "assertionMethod": [
      "#key-2"
    ],
    "service": [
      {
        "serviceEndpoint": {
          "uri": "http://host.docker.internal:8080/didcomm",
          "routingKeys": [],
          "accept": [
            "didcomm/v2"
          ]
        },
        "type": "DIDCommMessaging",
        "id": "#service"
      }
    ]
  },
  "didResolutionMetadata": {
    "contentType": "application/did+ld+json",
    "pattern": "^(did:peer:.+)$",
    "driverUrl": "http://uni-resolver-driver-did-uport:8081/1.0/identifiers/",
    "duration": 3,
    "driverDuration": 3,
    "did": {
      "didString": "did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ",
      "methodSpecificId": "2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ",
      "method": "peer"
    }
  },
  "didDocumentMetadata": {}
}
```

Now this looks familiar, we have the usual set of keys and it looks similar to the mediator DID but in this case we see a `serviceEndpoint` that contains accepts `didcomm/v2` and it's intended to be used for `DIDCommMessaging`. What all this means is the Cloud Agent DID is essentially advertising how it will receive `DIDComm` messages. In other words this could be translated as: "Here is an invite to connect to me, on it you will find a DID that has my public keys and a service endpoint where I receive `DIDComm` messages".

The final piece to unpack is the `invitationUrl`, this looks a little odd at first:

```
"https://my.domain.com/path?_oob=eyJpZCI6ImZiMzZlZGRkLWQ1MWUtNDJjZi1hNmZlLWU3NmQyZTYzOGI3MCIsInR5cGUiOiJodHRwczovL2RpZGNvbW0ub3JnL291dC1vZi1iYW5kLzIuMC9pbnZpdGF0aW9uIiwiZnJvbSI6ImRpZDpwZWVyOjIuRXo2TFNnWTZZNjdtSjc1WUNaZlpZeFlFUFFKWnMzdmFFZzJDYzkxdnBwb1RBN2Nwai5WejZNa3BYN0g3U05BNm9vRzVzbm4yTXpneW9SYWRFWnRzak5TTDF4N0hpaUxrcXlWLlNleUowSWpvaVpHMGlMQ0p6SWpwN0luVnlhU0k2SW1oMGRIQTZMeTlvYjNOMExtUnZZMnRsY2k1cGJuUmxjbTVoYkRvNE1EZ3dMMlJwWkdOdmJXMGlMQ0p5SWpwYlhTd2lZU0k2V3lKa2FXUmpiMjF0TDNZeUlsMTlmUSIsImJvZHkiOnsiYWNjZXB0IjpbXX19"
```

The first thing that looks wrong is the domain, where does `my.domain.com` comes from? well, it comes from the Cloud Agent and it's hardcoded, you can't customize it but it really doesn't matter, what matters is the payload of the `_oob` field. The URL is not important as what we really need is inside the `base64` encoded field, lets unpack it.

```bash
echo 'eyJpZCI6ImZiMzZlZGRkLWQ1MWUtNDJjZi1hNmZlLWU3NmQyZTYzOGI3MCIsInR5cGUiOiJodHRwczovL2RpZGNvbW0ub3JnL291dC1vZi1iYW5kLzIuMC9pbnZpdGF0aW9uIiwiZnJvbSI6ImRpZDpwZWVyOjIuRXo2TFNnWTZZNjdtSjc1WUNaZlpZeFlFUFFKWnMzdmFFZzJDYzkxdnBwb1RBN2Nwai5WejZNa3BYN0g3U05BNm9vRzVzbm4yTXpneW9SYWRFWnRzak5TTDF4N0hpaUxrcXlWLlNleUowSWpvaVpHMGlMQ0p6SWpwN0luVnlhU0k2SW1oMGRIQTZMeTlvYjNOMExtUnZZMnRsY2k1cGJuUmxjbTVoYkRvNE1EZ3dMMlJwWkdOdmJXMGlMQ0p5SWpwYlhTd2lZU0k2V3lKa2FXUmpiMjF0TDNZeUlsMTlmUSIsImJvZHkiOnsiYWNjZXB0IjpbXX19' | base64 -d
```
```json
{
  "id": "fb36eddd-d51e-42cf-a6fe-e76d2e638b70",
  "type": "https://didcomm.org/out-of-band/2.0/invitation",
  "from": "did:peer:2.Ez6LSgY6Y67mJ75YCZfZYxYEPQJZs3vaEg2Cc91vppoTA7cpj.Vz6MkpX7H7SNA6ooG5snn2MzgyoRadEZtsjNSL1x7HiiLkqyV.SeyJ0IjoiZG0iLCJzIjp7InVyaSI6Imh0dHA6Ly9ob3N0LmRvY2tlci5pbnRlcm5hbDo4MDgwL2RpZGNvbW0iLCJyIjpbXSwiYSI6WyJkaWRjb21tL3YyIl19fQ",
  "body": {
    "accept": []
  }
}
```

And there it is, the `_oob` encoded payload contains the bare minimum to tell you it's a `DIDComm` invitation from a DID Peer.

## Connectionless Credential Presentations and Issuance

In the realm of digital identity, establishing trust and exchanging verifiable credentials typically involves a series of interactions between an issuer or verfier and a holder. While many  protocols need an existing connection or relationship, connectionless flow offers a more streamlined approach for specific scenarios. This method is particularly useful when a prior relationship between the issuer or verfier and the holder has not yet been established or is not necessary for a particular interaction.

The fundamental distinction of connectionless issuance and presentation lies in its initiation. Unlike traditional flows that might require a formal connection setup (e.g., exchanging DIDs and establishing a DIDComm channel) *before* a credential offer can be made, the connectionless flow leverages an Out-of-Band (OOB) invitation to kickstart the process directly.

When an issuer or verifier intends to offer or request proof this way, they generate an OOB invitation. This invitation is essentially a self-contained message or a reference (like a URL or QR code) that the  holder can receive through various means (QR code, email, a website, etc). Upon parsing this OOB invitation, the holder's agent can immediately understand the intent to offer or proof a credential.

The key here is that the OOB invitation itself contains enough information, or points to it, for the holder's agent to proceed with requesting the credential offer from the issuer or presenting proof to a verifier. This bypasses the need for a separate, preliminary connection handshake. The issuer or verifier, upon receiving a request derived from this OOB invitation, can then proceed to send the actual credential offer or proof request.

From the holder's acceptance of the offer onwards, the subsequent steps closely mirror those in a standard issuance and proof request protocols. The primary efficiency and distinction of the connectionless approach are concentrated at the beginning of the interaction, enabling a quicker, more direct path when a persistent connection is not a prerequisite. This makes it ideal for scenarios like anonymous attestations, one-time verifications, or public credential offerings where the overhead of establishing and managing connections for every holder is impractical.

## Example Project

### Cloud Agent

TODO: Instructions on how to create an invite on Cloud Agent.

### Swift SDK

TODO: Example code to parse invites on Swift SDK

### TypeScript SDK

TODO: Example code to parse invites on TypeScript SDK
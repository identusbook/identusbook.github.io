---
title: "Connections"
---
## Overview

Now that we have a better understanding of Wallets and DIDs, it's time to embark on our first interaction. In this chapter we are going to explore conceptually what a Connection means in SSI, take a deep dive into DID Peers, explain how they work and why they are needed for secure connections, dissect Out of Band invites and finally hands on example code to achieve connecting edge client to an agent.

Before we move forward we highly recommend to at least read the basic [Connection tutorial](https://docs.atalaprism.io/tutorials/connections/connection) on the official [Atala documentation](https://docs.atalaprism.io/).

## Connections in Self-Sovereign Identity

Connections are fundamental to establishing *trusted interactions* between peers. They enable secure and verifiable communication, allowing entities to exchange credentials and proofs in a decentralized manner. This relationship is established using a specific decentralized identifier standard (**Peer DID**) and is governed by a protocol (**DIDCOMM**) that ensure the authenticity, integrity, and privacy of the interactions between the connected parties.

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

However, when Alice and Bob want to interact with each other, only two parties care about the details of that connection: Alice and Bob. Instead of arbitrary parties needing to resolve their DIDs, only Alice and Bob do. Thus, PeerDIDs essentially describe a key-pair to encrypt and sign data to and from Alice and Bob, routed trough their preferred mediators, e.g. When Alice accepts an invite from Bob and they engage the connection protocol, Alice generates a PeerDID that allows her to encrypt and sign data routed trough Bob's mediator (mediator Y) that only Bob can decrypt, and viceversa, Bob will generate a PeerDID that allows him to encrypt and sign data routed trough Alice's preferred mediator (mediator X) that only Alice can decrypt.

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
As you can see, an Out of Band invite is really just a way to package a PeerDID and signaling that it can be used for a particular interaction. In this case, for a `RequestMediate` goal over `DIDCOMM`.

As a refresher from what we covered, a PeerDID is a way to package a set of keys and optional service endpoints, and so, *because this an OOB invite from a mediator*, this invite has everything you need (a PeerDID and set of service endpoints) to establish this service as your mediator.

## Connecting two peers

Now, lets issue another kind of Out of Band invite, one from the cloud agent in order to connect.

**TODO Checklist**

- [x] Concept of Connections
- [x] Explain PeerDIDs
- [x] How connections are achieved trough PeerDIDs
- [x] Out of Band invites
- [ ] How to Connect two peers
- [ ] DIDless connection (Atala Roadmap)

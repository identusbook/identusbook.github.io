---
title: "DIDComm"
---

## Overview

**DIDComm v2** (*Decentralized Identifier Communication version 2*) is a set of communication protocols designed to enable secure, private, and interoperable messaging between DIDs. These protocols have been adopted throughout Identus to standardize many important interactions, like sending and receiving encrypted messages between a mediator its peers.



DIDComm v2 messages are encrypted JSON Web Message (JWM) envelopes have a standard structure that includes headers, a body, and optional attachments. Messages are encrypted and signed to ensure security and integrity.

Components of DIDComm v2 Architecture

	1.	Message Structure: DIDComm v2 messages have a standard structure that includes headers, a body, and optional attachments. Messages are encrypted and signed to ensure security and integrity.
	2.	Protocols: Defines reusable interaction patterns (protocols) for common use cases, such as issuing credentials, requesting proofs, or establishing connections. Protocols are designed to be composable and extensible.
	3.	Encryption and Signing: Uses modern cryptographic techniques to provide end-to-end encryption and digital signatures. This ensures that messages cannot be read or tampered with by unauthorized parties.
	4.	Transport Agnostic: DIDComm v2 is transport-agnostic, meaning it can work over various communication channels such as HTTP, WebSockets, Bluetooth, or even physical delivery methods like QR codes.
	5.	Agents: Entities that act on behalf of DIDs to send, receive, and process DIDComm messages. Agents can be cloud-based, mobile, or desktop applications, and they manage the keys and credentials needed for secure communication.

Finding More Information

	•	DIDComm v2 Specification: The official specification can be found on the Decentralized Identity Foundation (DIF) GitHub repository.
	•	DIF Community: Engage with the community through DIF’s mailing lists, working groups, and forums. The DIF website (https://identity.foundation) provides resources and information on how to participate.
	•	Documentation and Tutorials: Many organizations working with DIDComm v2 provide extensive documentation and tutorials. Examples include the Hyperledger Aries project and the Sovrin Foundation.
	•	Academic Papers and Articles: Research papers and articles on decentralized identity and DIDComm v2 can provide deeper insights into the theoretical foundations and practical implementations of the protocol.

## Sending Messages

## Sending Files
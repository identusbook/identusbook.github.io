---
title: "Installation - Development Environment"
---
## Introduction

Hyperledger Identus, previously known as Atala PRISM, is distributed across various repositories. These repositories group together different building blocks to provide the necessary functionality for fulfilling each of the essential roles in Self-Sovereign Identity (SSI), as introduced in [Identus Concepts](/section1/identus-concepts.html) and [SSI Basics](/section1/ssi-basics.html). Throughout this book, we will detail the setup of each component.

The initial component to set up is our Cloud Agent. This agent is responsible for creating and publishing DID Documents into a Verifiable Data Registry (VDR), issuing Verifiable Credentials, and, depending on the configuration, even providing Identity Wallets to multiple users through a multi-tenancy setup. For now, our focus will be on setting up the Cloud Agent to run locally in development mode, supporting only a single tenant. This step is crucial for learning the basics and getting started. As we progress and build our example application, we will deploy the Cloud Agent in pre-production mode and, finally, in production mode with multi-tenancy support.

## Identus Releases Overview

Identus is built upon multiple interdependent building blocks, including the Cloud Agent, Wallet SDK, Mediator, and Apollo Crypto Library. To ensure compatibility among these components, it is crucial to identify the correct building block versions that are compatible between them. For this purpose, a dedicated repository named [atala-releases](https://github.com/input-output-hk/atala-releases) is available. This repository provides comprehensive documentation and a compatibility table for each Identus Release. We will be using [Identus v2.12](https://github.com/input-output-hk/atala-releases/blob/master/Atala%20PRISM/2.12.md) as our selected release because it is the latest at the time of writing (May 2024).

## Pre-requisites

### Git

GitHub is currently the primary platform for hosting repositories. As of this writing, projects are transitioning from Atala PRISM's original repositories to Hyperledger ones, with the Cloud Agent being the first to migrate. For more details, you can read the press release [here](https://iohk.io/en/blog/posts/2023/12/04/iog-contributes-atala-prism-to-hyperledger-foundation/).

If you're using a UNIX-based system (such as OS X or Linux), you likely already have `git` installed. If not, you can download the installer from [Git downloads](https://www.git-scm.com/downloads). Additionally, various [GUI clients](https://www.git-scm.com/downloads/guis) are available for those who prefer a graphical interface.

To clone the Cloud Agent repository, first go to the [Releases page](https://github.com/hyperledger/identus-cloud-agent/releases) and identify the tagged release corresponding to the Identus release you are targeting (e.g. `cloud-agent-v1.33.0` is part of [Identus v2.12](https://github.com/input-output-hk/atala-releases/blob/master/Atala%20PRISM/2.12.md) release), then clone the repository with this command:

```bash
git clone --depth 1 --branch cloud-agent-v1.33.0 https://github.com/hyperledger/identus-cloud-agent
```

::: {.callout-note}
Using `--depth 1` will skip the history of changes in the repository up until the point of the tag, this is optional.
:::

### Atala Community Projects

There is a growing list of community repositories that aim to provide some extra functionality, mostly maintained by official developers and community members on their spare time. At present time there are three [Atala Community Projects](https://github.com/atala-community-projects):

- **Pluto Encrypted:** Implementation of Pluto storage engine with encryption support.
- **Edge Agent SDK Demos:** Browser and Node versions of Edge Agent SDK integrated with Pluto Encrypted.
- **Identus Test:** Shell script helper that will checkout a particular Identus release and compatible components.

### Docker

The Cloud Agent and Mediator are distributed as Docker containers, which is the recommended method for starting and stopping the various components required to run the cloud infrastructure.

To begin, install [Docker Desktop](https://www.docker.com/products/docker-desktop/), which provides everything you need to get started.

### WSL

For Windows users, please refer to [How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install). 

::: {.callout-note}
Windows is the least tested environment, the community have already found some issues and workarounds on how to get the Cloud Agent working. We will try to always include instructions regarding this use case.
:::

## Installation


---
title: Installation - Local Environment
---
## Overview

Hyperledger Identus, previously known as Atala PRISM, is distributed across various repositories. These repositories group together different building blocks to provide the necessary functionality for fulfilling each of the essential roles in Self-Sovereign Identity (SSI), as introduced in [Identus Concepts](/section1/identus-concepts.html) and [SSI Basics](/section1/ssi-basics.html). Throughout this book, we will detail the setup of each component.

The initial component to set up is our Cloud Agent. This agent is responsible for creating and publishing DID Documents into a Verifiable Data Registry ([VDR](/glossary.html#vdr)), issuing Verifiable Credentials, and, depending on the configuration, even providing Identity Wallets to multiple users through a multi-tenancy setup. For now, our focus will be on setting up the Cloud Agent to run locally in development mode, supporting only a single tenant. This step is crucial for learning the basics and getting started. As we progress and build our example application, we will deploy the Cloud Agent in development mode first, then set it up to connect to Cardano `testnet` as our VDR on pre-production mode and, finally, in production mode with multi-tenancy support connected to Cardano `mainnet`. This will be a gradual process as we need to familiarize on each stage of the Cloud Agent.

## Identus Releases Overview

Identus is built upon multiple interdependent building blocks, including the Cloud Agent, Wallet SDK, Mediator, and Apollo Crypto Library. To ensure compatibility among these components, it is crucial to identify the correct building block versions that are compatible between them. Identus releases are listed [here](https://github.com/hyperledger/identus/releases). The release notes for each vesion provides a compatibility table for each Identus release.  This guide will focus on the latest stable Identus release at the time of writing (December 2024).

## Pre-requisites

### Git

If you're using a UNIX-based system (such as OS X or Linux), you likely already have `git` installed. If not, you can download the installer from [Git downloads](https://www.git-scm.com/downloads). Additionally, various [GUI clients](https://www.git-scm.com/downloads/guis) are available for those who prefer a graphical interface.

To clone the Cloud Agent repository, first go to the [Releases page](https://github.com/hyperledger/identus/releases) and identify the tagged release corresponding to the Identus release you are targeting (e.g. `cloud-agent-v1.40.0` is part of [Identus v2.14](https://github.com/hyperledger/identus/releases/tag/v2.14) release), then clone the repository with this command:

```bash
git clone --depth 1 --branch cloud-agent-v1.40.0 https://github.com/hyperledger/identus-cloud-agent
```

::: {.callout-note}
Using `--depth 1` will skip the history of changes in the repository up until the point of the tag, this is optional.
:::

### Docker

The Cloud Agent and Mediator are distributed as Docker containers, which is the recommended method for starting and stopping the various components required to run the cloud infrastructure.

To begin, install [Docker Desktop](https://www.docker.com/products/docker-desktop/), which provides everything you need to get started.

### WSL

For Windows users, please refer to [How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install). 

::: {.callout-note}
Windows is the least tested environment, the community has already found some issues and workarounds on how to get the Cloud Agent working. We will try to always include instructions regarding this use case.
:::

## Before We Run The Agent

Once you have cloned the `identus-cloud-agent` repository and Docker is up and running you can jump right ahead and [run the agent](#running-the-cloud-agent), but before we do that if you are not yet familiar with the community projects or the structure of the agent itself, we recommend you to spend a little time exploring the following information, this is optional and you can skip it.

### Atala Community Projects

There is a growing list of community repositories that aim to provide some extra functionality, mostly maintained by official developers and community members on their spare time. 

Some notable projects are:

- [**Pluto Encrypted:**](https://github.com/trust0-project/pluto-encrypted) Implementation of Pluto storage engine with encryption support.
- [**Identus Store**](https://github.com/trust0-project/identus-store) A secure light-weight and dependency free database wrapper.
- [**NeoPrism Resolver:**](https://neoprism.patlo.dev/resolver) A did:prism resover and explorer.
- [**Blocktrust Mediator**](https://blocktrust.dev/mediator) A DIDCommv2 compliant mediator, written in C#.
- **Edge Agent SDK Demos:** Browser and Node versions of Edge Agent SDK integrated with Pluto Encrypted.
- **Identus Test:** Shell script helper that will checkout a particular Identus release and compatible components.

### Exploring The Repository

There are two fundamental directories inside the repository if you are an end user.

1. `docs` Where all the latest *technical documentation* will be available, this includes the Architecture Decision Records ([ADR](/glossary.html#adr)), general insights, guides to deploy, examples and tutorials about how to handle VC Schemas, Connections, secrets, etc. We will do our best to explain in detail all this procedures as we build our example app.
2. `infrastructure` This directory holds the agent's Docker file and related scripts to run the agent in different modes such as `dev`, `local` or `multi`. **The way to change the agent setup is by customizing environmental variables trough the Docker file**, so we really advise you to get familiar with the `shared` directory content, because that's the base for every other mode, in essence, every mode is a customization of the shared Docker file.

Our first mode to explore and the simplest one should be `local` mode, which by default will run a single agent as a single-tenant, meaning that this instance will control only a single [Identity Wallet](/glossary.html#identity-wallet) that will be automatically created and seeded upon the first start of the agent.

The `multi` mode essentially runs 3 different `local` agents but each is assigned a particular role such as `issuer`, `holder` and `verifier`. This is useful in order try test more complex interactions between independent actors.

Finally the `dev` mode is meant to be used for development and provides an easy way to modify the Cloud Agent source code, it does not rely on the pre-built Docker images that the `local` mode fetches and run. We will not use this mode at all trough the book but feel free to explore this option if you would like to make contributions to the Cloud Agent in the future.

## Running The Cloud Agent

### Environment Variables

Inside `infrastructure/local` directory, you will find three important files, `run.sh`and `stop.sh` scripts and the `.env` file.

Our `local` environment file should look like this

```bash
AGENT_VERSION=1.40.0
PRISM_NODE_VERSION=2.5.0
VAULT_DEV_ROOT_TOKEN_ID=root
```

This will tell Docker which versions of the Cloud Agent and PRISM Node to run, plus a default value for the `VAULT_DEV_ROOT_TOKEN_ID`, this value corresponds to the HashiCorp Token ID. HashiCorp is a secrets storage engine and it will become relevant later on when we need to prepare the agent to run in `prepod` and `production` modes, for now the `local` mode will ignore this value because by default it will use a local `postgres` database for it's secret storage engine.

The `run.sh` script options:

```bash
./run.sh --help
Run an instance of the ATALA bulding-block stack locally

Syntax: run.sh [-n/--name NAME|-p/--port PORT|-b/--background|-e/--env|-w/--wait|--network|-h/--help]
options:
-n/--name              Name of this instance - defaults to dev.
-p/--port              Port to run this instance on - defaults to 80.
-b/--background        Run in docker-compose daemon mode in the background.
-e/--env               Provide your own .env file with versions.
-w/--wait              Wait until all containers are healthy (only in the background).
--network              Specify a docker network to run containers on.
--webhook              Specify webhook URL for agent events
--webhook-api-key      Specify api key to secure webhook if required
--debug                Run additional services for debug using docker-compose debug profile.
-h/--help              Print this help text.
```

For our first interaction with the agent all we have to do is to call the run script. If you have any conflicts with the port 80 already in use or you don't want that as the default, you can pass `--port 8080` or any other available port that you would like to use.

So, from the root of the repository you can run:

```bash
./infrastructure/local/run.sh
```

This will take a while the first time as Docker will fetch the required container images and get them running. To check the status of the Cloud Agent you can use `curl` or open a browser window at same endpoint URL (make sure to specify a custom port if you changed it in the previous step, e.g. use `http://localhost:8080`):

```bash
curl http://localhost/cloud-agent/_system/health
{"version":"1.40.0"}
```

The `version` should match the version of the Cloud Agent defined in the `.env`file.

To stop the agent, you can press `Control + C` or run:

```bash
./infrastructure/local/stop.sh
```

Congratulations! you have successfully setup the agent in `local` mode. Next we will explore our Docker file in detail and interact with our agent using the REST API.
---
title: "Installation - Development Environment"
---

## Introduction

Hyperledger Identus, formerly Atala PRISM, comes distributed in separate repositories that group together different building blocks in order to provide the required functionality to fulfill each of the essential roles in SSI that we introduced in [Identus Concepts](/section1/identus-concepts.html) and [SSI Basics](/section1/ssi-basics.html). We will explain the setup in detail of each component as we progress trough this book.

The first component we need to setup is our Cloud Agent, responsible for creating and publishing DID Documents into a Verifiable Data Registry (VDR), issuing Verifiable Credentials and depending on the configuration, even providing Identity Wallets to multiple users trough a multi-tenancy setup. For now, we will focus on how to setup the Cloud Agent to run locally in development mode supporting only a single-tenant, this is essential to learn the basics and get you started, as we progress and build our example application, we will deploy the same Cloud Agent in pre-production mode and finally in production mode with multi-tenancy.

## Pre-requisites

### WSL

For Windows users, please refer to [How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install). 

::: {.callout-note}
Windows is the least tested environment, the community have already found some issues and workarounds on how to get the Cloud Agent working. We will try to always include instructions regarding this use case.
:::

### Git

The current home for every repository is [GitHub](https://github.com), although at the time of this writing the project is still transitioning from Atala PRISM original repositories into Hyperledger ones, the Cloud Agent being the first one to migrate (you can read the press release [here](https://iohk.io/en/blog/posts/2023/12/04/iog-contributes-atala-prism-to-hyperledger-foundation/))

Chances are if you develop in a UNIX based machine (OS X, Linux, etc) you already have `git`, if not you can get the installer from [Git downloads](https://www.git-scm.com/downloads) or even checkout one of the [GUI clients](https://www.git-scm.com/downloads/guis) available.

```bash
git clone https://github.com/hyperledger/identus-cloud-agent
```
### Docker

Both the Cloud Agent and Mediator come distributed as docker containers and it's the preferred way to start and stop all the different components needed to run the cloud infrastructure.

Installing [Docker Desktop](https://www.docker.com/products/docker-desktop/) will get you all you need to get started.
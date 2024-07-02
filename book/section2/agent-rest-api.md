---
title: Agent REST API
---
## The Cloud Agent API

The only way to interact with our newly created Cloud Agent is trough the REST API, this means any action of the Cloud Agent such as establishing connections, creating Credential Schemas, issuing Verifiable Credentials, etc. will be triggered trough the agent API endpoints.

It is crucial to understand that the API is essentially an abstraction of the agent's Identity Wallet. In our initial setup our agent is running on single-tenant mode, therefor it is managing a single default wallet which is assigned the following Entity ID (UUID) `00000000-0000-0000-0000-000000000000`. You can confirm this by running the following command:

```bash
docker logs local-cloud-agent-1 | grep "default"


... Initializing default wallet.
... Default wallet seed is not provided. New seed will be generated.
... Entity created: Entity(00000000-0000-0000-0000-000000000000,default,00000000-0000-0000-0000-000000000000,1970-01-01T00:00:00Z,1970-01-01T00:00:00Z)
```

Later on the book, once we start building our example app, we will setup the agent in multi-tenant mode, meaning that a single agent instance will be capable of managing multiple wallets, we refer to those wallets as custodian wallets. This more advanced setup is useful when your users would want to delegate their Identity Wallet custody to a service instead of managing the wallet themselves.

Besides the `_system/health` endpoint we used earlier to confirm the agent version, there is one more endpoint used to debug runtime metrics:

```bash
curl http://127.0.0.1:80/cloud-agent/_system/metrics
# HELP jvm_memory_bytes_used  
# TYPE jvm_memory_bytes_used gauge
jvm_memory_bytes_used{area="heap",} 1.07522616E8
jvm_memory_bytes_used{area="nonheap",} 1.74044936E8
...
```
This will be useful when debugging a memory or performance issue or when developing.

## OpenAPI Specification

The OpenAPI Specification (`OAS`) defines a standard, language-agnostic interface to HTTP APIs. The Cloud Agent API documentation can be found at [https://docs.atalaprism.io/agent-api/](https://docs.atalaprism.io/agent-api/)  and besides being very detailed and always updated to the latest, it also comes with the OAS spec yaml file that will allow us to setup `Postman` to easily test our API or use the same OAS standard to auto generate code for client libraries on different language stacks. We will not attempt to repeat this API documentation in the book, rather lets focus on how to setup this tools to issue our first `DID` and later on, connect two agents together and issue a Credential Schema and Verifiable Credential as our first example interactions.

## Postman Setup

`Postman`is perhaps the most popular API tool among developers, it allow us to easily interact and debug API endpoints but has many killer feature like  enabling teams to share and work together on the same API, run automated tests, automatically renew tokens, etc.

The big time saver for us is that because it supports OAS, we can easily import the whole API definition. So, let's try it:

1. If you don't already have it, first you should [Download Postman](https://www.postman.com/downloads/) and [Sign Up](https://identity.getpostman.com/signup?continue=https%3A%2F%2Fgo.postman.co%2Fhome) for a free account.
2. Head to the [API docs](https://docs.atalaprism.io/agent-api/) and click the Download button, or copy this direct link `https://docs.atalaprism.io/redocusaurus/plugin-redoc-0.yaml`
3. Inside Postman, go to `File -> Import` and either drag & drop your yaml file if you downloaded it or paste the URL in the box, this will auto advance to the next step.
4. On the "How to import" step select "OpenAPI 3.0 with a Postman Collection" and click import.

If everything goes correctly, you should see "Identus Cloud Agent API Reference" in your collections.

TODO:

- Test the agent by creating a DID and schema definition (to be used in section3 with edge sdk)
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

## OpenAPI Spec

TODO:

- Explain Postman
- Reference the API Docs https://docs.atalaprism.io/agent-api/
- Download OpenAPI Spec file and import in Postman
- Test the agent by creating a schema definition (to be used in section3 with edge sdk)
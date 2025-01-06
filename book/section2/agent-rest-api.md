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
curl http://localhost/cloud-agent/_system/metrics
# HELP jvm_memory_bytes_used  
# TYPE jvm_memory_bytes_used gauge
jvm_memory_bytes_used{area="heap",} 1.07522616E8
jvm_memory_bytes_used{area="nonheap",} 1.74044936E8
...
```
This will be useful when debugging a memory or performance issue or when developing.

## OpenAPI Specification

The OpenAPI Specification (`OAS`) defines a standard, language-agnostic interface to HTTP APIs. The Cloud Agent API documentation can be found at [https://hyperledger.github.io/identus-docs/agent-api/](https://hyperledger.github.io/identus-docs/agent-api/)  and besides being very detailed and always updated to the latest, it also comes with the OAS spec yaml file that will allow us to setup `Postman` to easily test our API or use the same OAS standard to auto generate code for client libraries on different language stacks. We will not attempt to repeat this API documentation in the book, rather lets focus on complement the existing documentation and explain with more detail how everything works.

## APISIX Gateway

APISIX is in charge of proxying different services inside the container, exposing three routes trough the port you specified to the `run.sh` script (remember it runs on `port 80` by default).

- [http://localhost/cloud-agent/](http://localhost/cloud-agent/) This will be the Cloud Agent API.
- [http://localhost/apidocs/](http://localhost/apidocs/) Swagger UI interface test the API.
- [http://localhost/didcomm/](http://localhost/didcomm/) Our public [DIDCOMM](/section3/didcomm.html) endpoint, this is our communication channel and it's how we send end to end encrypted messages to another peer trough a mediator. We will take a deep dive into [DIDCOMM](/section3/didcomm.html) later in the book.

APISIX by default will just expose this services but trough plugins it can be setup as Ingress controller, load balancer, authentication and much more. You can read the [APISIX documentation](https://apisix.apache.org/docs/) to learn more.

::: {.callout-warning}
This is where `CORS` ([Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)) is setup, be default it will allow any origin, here you can restrict which domains should be allowed to connect to this endpoints. We will revisit this on our customization guide.
:::

## Swagger UI

[Swagger UI](https://swagger.io/tools/swagger-ui/) it's a visualization and interactive tool to explore an API. It's automatically generated from your OpenAPI (formerly known as Swagger) Specification and it's often used to support the API documentation.

Our Cloud Agent Docker file includes a container for Swagger UI that is exposed trough APISIX as explained earlier. This means you can use this tool right away after the agent is running.

To use it, just open [http://localhost/apidocs/](http://localhost/apidocs/) in your browser and from the server list select:

- `http://localhost/cloud-agent - The local instance of the Cloud Agent behind the APISIX proxy`.

Then, click the `Authorize` button and a small modal window will popup, in there you need to define an `apikey`, even if by default you haven't defined one, this means you can put any value in here, of course later in the book when we set an actual `apikey` you will need to use it here, for now just use `test` as value and it should be fine.

After authorizing, the modal should look like this:

![Swagger UI Apikey Modal](/section2/swagger-ui-apikey-modal.png)

You can close that modal window and try your first request, click to expand the `GET /connections` endpoint and click `Try it out` button, that will enable the text inputs for any available parameters, for now, all three parameters should be blank (`offet`, `limit`, `thid`).

Finally, just click the `Execute` button to actually perform the request. This should return something like this:

```json
{ "contents": [], "kind": "ConnectionsPage", "self": "", "pageOf": "" }
```

Congratulations! you have connected to the API and asked for a list of `connections`, right now there are no connections so the empty array you get back is correct.

::: {.callout-note}
You can use Swagger UI to copy `curl` commands that you can paste in your terminal, this will run exactly the same API request. For example:

```bash
curl -X 'GET' \
  'http://localhost/cloud-agent/connections' \
  -H 'accept: application/json' \
  -H 'apikey: test'
{"contents":[],"kind":"ConnectionsPage","self":"","pageOf":""}
```
:::
## Postman

`Postman`is perhaps the most popular API tool among developers, it allow us to easily interact and debug API endpoints but has many killer feature like  enabling teams to share and work together on the same API, run automated tests, automatically renew tokens, keeps the state of your interactions with the API, copy code snippets to make API calls over many languages, etc. So it's really a better overall option versus the Swagger UI interface or just directly using `curl`.

The big time saver for us is that because it supports OAS, we can easily import the whole API definition. So, let's try it:

1. If you don't already have it, first you should [Download Postman](https://www.postman.com/downloads/) and [Sign Up](https://identity.getpostman.com/signup?continue=https%3A%2F%2Fgo.postman.co%2Fhome) for a free account.
2. Head to the [API docs](https://hyperledger.github.io/identus-docs/agent-api/) and click the Download button, or copy this direct link `https://hyperledger.github.io/identus-docs/redocusaurus/plugin-redoc-0.yaml`
3. Inside Postman, go to `File -> Import` and either drag & drop your yaml file if you downloaded it or paste the URL in the box, this will auto advance to the next step.
4. On the "How to import" step select "OpenAPI 3.0 with a Postman Collection" and click import.

If everything goes correctly, you should see "Identus Cloud Agent API Reference" in your collections.

## Tutorials

The official documentation contains a [tutorials](https://hyperledger.github.io/identus-docs/tutorials/) section with detailed walkthroughs for each of the most important interactions like connecting to another peer, managing DIDs, managing VC Schemas, issuing a VC, etc. We highly encourage you to follow those and get familiar with the API, this will come in very handy very soon when we start building our own example app.
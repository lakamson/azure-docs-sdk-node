---
title: 
keywords: Azure, javascript, SDK, API, @azure/web-pubsub-express, web-pubsub
ms.date: 02/28/2025
ms.topic: reference
ms.devlang: javascript
ms.service: web-pubsub
---
# Azure Web PubSub CloudEvents handlers for Express

[Azure Web PubSub service](https://aka.ms/awps/doc) is an Azure-managed service that helps developers easily build web applications with real-time features and publish-subscribe pattern. Any scenario that requires real-time publish-subscribe messaging between server and clients or among clients can use Azure Web PubSub service. Traditional real-time features that often require polling from server or submitting HTTP requests can also use Azure Web PubSub service.

When a WebSocket connection connects, the Web PubSub service transforms the connection lifecycle and messages into [events in CloudEvents format](https://learn.microsoft.com/azure/azure-web-pubsub/concept-service-internals#workflow). This library provides an express middleware to handle events representing the WebSocket connection's lifecycle and messages, as shown in below diagram:

![cloudevents](https://user-images.githubusercontent.com/668244/140321213-6442b3b8-72ee-4c28-aec1-127f9ea8f5d9.png)

Details about the terms used here are described in [Key concepts](#key-concepts) section.

[Source code](https://github.com/Azure/azure-sdk-for-js/blob/@azure/web-pubsub-express_1.0.6/sdk/web-pubsub/web-pubsub-express) |
[Package (NPM)](https://www.npmjs.com/package/@azure/web-pubsub-express) |
[API reference documentation](https://aka.ms/awps/sdk/js) |
[Product documentation](https://aka.ms/awps/doc) |
[Samples][samples_ref]

## Getting started

### Currently supported environments

- [LTS versions of Node.js](https://github.com/nodejs/release#release-schedule)
- [Express](https://expressjs.com/) version 4.x.x or higher

### Prerequisites

- An [Azure subscription][azure_sub].
- An existing Azure Web PubSub endpoint.

### 1. Install the `@azure/web-pubsub-express` package

```bash
npm install @azure/web-pubsub-express
```

### 2. Create a `WebPubSubEventHandler`

```ts snippet:ReadmeSampleCreateClient
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat");

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

## Key concepts

### Connection

A connection, also known as a client or a client connection, represents an individual WebSocket connection connected to the Web PubSub service. When successfully connected, a unique connection ID is assigned to this connection by the Web PubSub service.

### Hub

A hub is a logical concept for a set of client connections. Usually you use one hub for one purpose, for example, a chat hub, or a notification hub. When a client connection is created, it connects to a hub, and during its lifetime, it belongs to that hub. Different applications can share one Azure Web PubSub service by using different hub names.

### Group

A group is a subset of connections to the hub. You can add a client connection to a group, or remove the client connection from the group, anytime you want. For example, when a client joins a chat room, or when a client leaves the chat room, this chat room can be considered to be a group. A client can join multiple groups, and a group can contain multiple clients.

### User

Connections to Web PubSub can belong to one user. A user might have multiple connections, for example when a single user is connected across multiple devices or multiple browser tabs.

### Client Events

Events are created during the lifecycle of a client connection. For example, a simple WebSocket client connection creates a `connect` event when it tries to connect to the service, a `connected` event when it successfully connected to the service, a `message` event when it sends messages to the service and a `disconnected` event when it disconnects from the service.

### Event Handler

Event handler contains the logic to handle the client events. Event handler needs to be registered and configured in the service through the portal or Azure CLI beforehand. The place to host the event handler logic is generally considered as the server-side.

## Examples

### Handle the `connect` request and assign `<userId>`

```ts snippet:ReadmeSampleConnect
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  handleConnect: (req, res) => {
    // auth the connection and set the userId of the connection
    res.success({
      userId: "<userId>",
    });
  },
  allowedEndpoints: ["https://<yourAllowedService>.webpubsub.azure.com"],
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Handle the `connect` request and reject the connection if auth failed

```ts snippet:ReadmeSampleConnectAndReject
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  handleConnect: (req, res) => {
    // auth the connection and reject the connection if auth failed
    res.fail(401, "Unauthorized");
    // the following method is also a valid approach
    // res.failWith({ code: 401, detail: "Unauthorized" });
  },
  allowedEndpoints: ["https://<yourAllowedService>.webpubsub.azure.com"],
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Handle the `connected` request

```ts snippet:ReadmeSampleConnected
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  onConnected: (connectedRequest) => {
    // Your onConnected logic goes here
  },
  allowedEndpoints: ["https://<yourAllowedService>.webpubsub.azure.com"],
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Handle the `onDisconnected` request

```ts snippet:ReadmeSampleDisconnected
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  onDisconnected: (disconnectedRequest) => {
    // Your onDisconnected logic goes here
  },
  allowedEndpoints: ["https://<yourAllowedService>.webpubsub.azure.com"],
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Handle the `connect` request for mqtt and assign `<userId>` and `<mqtt>` properties

```ts snippet:ReadmeSampleConnectMqtt
import { WebPubSubEventHandler, MqttConnectRequest } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  handleConnect: (req, res) => {
    if (req.context.clientProtocol === "mqtt") {
      // return mqtt response when request is of MQTT kind
      // get connect request as mqtt request and print it
      const mqttRequest = req as MqttConnectRequest;
      console.log(mqttRequest);

      // auth the connection and return mqtt response
      res.success({
        userId: "user1",
        mqtt: { userProperties: [{ name: "a", value: "b" }] },
      });
    } else {
      res.success({
        userId: "user1",
      });
    }
  },
  allowedEndpoints: ["https://<yourAllowedService>.webpubsub.azure.com"],
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Handle the `connect` request for mqtt and reject the connection if auth failed

```ts snippet:ReadmeSampleConnectMqttAndReject
import { WebPubSubEventHandler, MqttConnectRequest } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  handleConnect: (req, res) => {
    // auth the connection and reject the connection if auth failed
    if (req.context.clientProtocol === "mqtt") {
      // return mqtt error response when request is of MQTT kind
      // get connect request as mqtt request and print it
      const mqttRequest = req as MqttConnectRequest;
      console.log(mqttRequest);

      // auth the connection and return mqtt failure response
      res.fail(401, "Not Authorized");

      // Or use below method for more fine-grained control over the MQTT return code
      // res.failWith({ mqtt: { code: MqttV500ConnectReasonCode.NotAuthorized } });
    } else res.success();
  },
  allowedEndpoints: ["https://<yourAllowedService>.webpubsub.azure.com"],
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Handle the `onDisconnected` for mqtt request

```ts snippet:ReadmeSampleDisconnectedMqtt
import { WebPubSubEventHandler, MqttDisconnectedRequest } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  onDisconnected: (disconnectedRequest) => {
    if (disconnectedRequest.context.clientProtocol === "mqtt") {
      // get disconnect request as mqtt request and print it
      const mqttRequest = disconnectedRequest as MqttDisconnectedRequest;
      console.log(mqttRequest.mqtt);
      // Your onDisconnected logic goes here
    } else {
      console.log(disconnectedRequest);
      // Your onDisconnected logic goes here
    }
  },
  allowedEndpoints: ["https://<yourAllowedService>.webpubsub.azure.com"],
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Only allow specified endpoints

```ts snippet:ReadmeSampleAllowedEndpoints
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  allowedEndpoints: [
    "https://<yourAllowedService1>.webpubsub.azure.com",
    "https://<yourAllowedService2>.webpubsub.azure.com",
  ],
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Set custom event handler path

```ts snippet:ReadmeSampleCustomPath
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  path: "/customPath1",
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  // Azure WebPubSub Upstream ready at http://localhost:3000/customPath1
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

### Set and read connection state

```ts snippet:ReadmeSampleState
import { WebPubSubEventHandler } from "@azure/web-pubsub-express";
import express from "express";

const handler = new WebPubSubEventHandler("chat", {
  handleConnect(req, res) {
    // You can set the state for the connection, it lasts throughout the lifetime of the connection
    res.setState("calledTime", 1);
    res.success();
  },
  handleUserEvent(req, res) {
    const calledTime = req.context.states.calledTime++;
    console.log(calledTime);
    // You can also set the state here
    res.setState("calledTime", calledTime);
    res.success();
  },
});

const app = express();

app.use(handler.getMiddleware());

app.listen(3000, () =>
  console.log(`Azure WebPubSub Upstream ready at http://localhost:3000${handler.path}`),
);
```

## Troubleshooting

### Enable logs

Enabling logging may help uncover useful information about failures. In order to see a log of HTTP requests and responses, set the `AZURE_LOG_LEVEL` environment variable to `info`.

```bash
export AZURE_LOG_LEVEL=verbose
```

Alternatively, logging can be enabled at runtime by calling `setLogLevel` in the `@azure/logger`:

```ts snippet:SetLogLevel
import { setLogLevel } from "@azure/logger";

setLogLevel("info");
```

For more detailed instructions on how to enable logs, you can look at the [@azure/logger package docs](https://github.com/Azure/azure-sdk-for-js/tree/@azure/web-pubsub-express_1.0.6/sdk/core/logger).

### Live Trace

Use **Live Trace** from the Web PubSub service portal to view the live traffic.

## Next steps

Please take a look at the
[samples][samples_ref]
directory for detailed examples on how to use this library.

## Contributing

If you'd like to contribute to this library, please read the [contributing guide](https://github.com/Azure/azure-sdk-for-js/blob/@azure/web-pubsub-express_1.0.6/CONTRIBUTING.md) to learn more about how to build and test the code.

## Related projects

- [Microsoft Azure SDK for Javascript](https://github.com/Azure/azure-sdk-for-js)

[azure_sub]: https://azure.microsoft.com/free/
[samples_ref]: https://github.com/Azure/azure-webpubsub/tree/main/samples/javascript/


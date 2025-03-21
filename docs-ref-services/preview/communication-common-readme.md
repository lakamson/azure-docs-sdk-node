---
title: Azure Communication Common client library for JavaScript
keywords: Azure, javascript, SDK, API, @azure/communication-common, communication
ms.date: 03/29/2023
ms.topic: reference
ms.devlang: javascript
ms.service: azure-communication-services
---
# Azure Communication Common client library for JavaScript - version 3.0.0-beta.1 


This package contains common code for Azure Communication Service libraries.

## Getting started

### Prerequisites

- An [Azure subscription][azure_sub].
- An existing Communication Services resource. If you need to create the resource, you can use the [Azure Portal][azure_portal], the [Azure PowerShell][azure_powershell], or the [Azure CLI][azure_cli].

### Installing

```bash
npm install @azure/communication-common
```

### Browser support

#### JavaScript Bundle

To use this client library in the browser, first you need to use a bundler. For details on how to do this, please refer to our [bundling documentation](https://aka.ms/AzureSDKBundling).

## Key concepts

### CommunicationTokenCredential and AzureCommunicationTokenCredential

The `CommunicationTokenCredential` is an interface used to authenticate a user with Communication Services, such as Chat or Calling.

The `AzureCommunicationTokenCredential` offers a convenient way to create a credential implementing the said interface and allows you to take advantage of the built-in auto-refresh logic.

Depending on your scenario, you may want to initialize the `AzureCommunicationTokenCredential` with:

- a static token (suitable for short-lived clients used to e.g. send one-off Chat messages) or
- a callback function that ensures a continuous authentication state during communications (ideal e.g. for long Calling sessions).

The tokens supplied to the `AzureCommunicationTokenCredential` either through the constructor or via the token refresher callback can be obtained using the Azure Communication Identity library.

## Examples

### Create a credential with a static token

For a short-lived clients, refreshing the token upon expiry is not necessary and the `AzureCommunicationTokenCredential` may be instantiated with a static token.

```typescript
const tokenCredential = new AzureCommunicationTokenCredential(
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjM2MDB9.adM-ddBZZlQ1WlN3pdPBOF5G4Wh9iZpxNP_fSvpF4cWs"
);
```

### Create a credential with a callback

Here we assume that we have a function `fetchTokenFromMyServerForUser` that makes a network request to retrieve a JWT token string for a user. We pass it into the credential to fetch a token for Bob from our own server. Our server would use the Azure Communication Identity library to issue tokens. It's necessary that the `fetchTokenFromMyServerForUser` function returns a valid token (with an expiration date set in the future) at all times.

```typescript
const tokenCredential = new AzureCommunicationTokenCredential({
  tokenRefresher: async () => fetchTokenFromMyServerForUser("bob@contoso.com"),
});
```

### Create a credential with proactive refreshing

Setting `refreshProactively` to true will call your `tokenRefresher` function when the token is close to expiry.

```typescript
const tokenCredential = new AzureCommunicationTokenCredential({
  tokenRefresher: async () => fetchTokenFromMyServerForUser("bob@contoso.com"),
  refreshProactively: true,
});
```

### Create a credential with proactive refreshing and an initial token

Passing `initialToken` is an optional optimization to skip the first call to `tokenRefresher`. You can use this to separate the boot from your application from subsequent token refresh cycles.

```typescript
const tokenCredential = new AzureCommunicationTokenCredential({
  tokenRefresher: async () => fetchTokenFromMyServerForUser("bob@contoso.com"),
  refreshProactively: true,
  token:
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjM2MDB9.adM-ddBZZlQ1WlN3pdPBOF5G4Wh9iZpxNP_fSvpF4cWs",
});
```

## Troubleshooting

- **Invalid token specified**: Make sure the token you are passing to the `AzureCommunicationTokenCredential` constructor or to the `tokenRefresher` callback is a bare JWT token string. E.g. if you're using the [Azure Communication Identity library][invalid_token_sdk] or [REST API][invalid_token_rest] to obtain the token, make sure you're passing just the `token` part of the response object.

## Next steps

- [Read more about Communication user access tokens](/azure/communication-services/concepts/authentication?tabs=javascript)

## Contributing

If you'd like to contribute to this library, please read the [contributing guide](https://github.com/Azure/azure-sdk-for-js/blob/@azure/communication-common_3.0.0-beta.1/CONTRIBUTING.md) to learn more about how to build and test the code.

## Related projects

- [Microsoft Azure SDK for Javascript](https://github.com/Azure/azure-sdk-for-js)

[azure_cli]: /cli/azure
[azure_sub]: https://azure.microsoft.com/free/
[azure_portal]: https://portal.azure.com
[azure_powershell]: /powershell/module/az.communication/new-azcommunicationservice
[invalid_token_sdk]: /javascript/api/@azure/communication-identity/communicationaccesstoken#@azure-communication-identity-communicationaccesstoken-token
[invalid_token_rest]: /rest/api/communication/communication-identity/issue-access-token#communicationidentityaccesstoken




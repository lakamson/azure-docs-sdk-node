---
title: Azure DocumentIntelligence (formerly FormRecognizer) REST client library for JavaScript
keywords: Azure, javascript, SDK, API, @azure-rest/ai-document-intelligence, documentintelligence
ms.date: 12/17/2024
ms.topic: reference
ms.devlang: javascript
ms.service: documentintelligence
---
# Azure DocumentIntelligence (formerly FormRecognizer) REST client library for JavaScript - version 1.0.0 


Extracts content, layout, and structured data from documents.

**Please rely heavily on our [REST client docs](https://github.com/Azure/azure-sdk-for-js/blob/@azure-rest/ai-document-intelligence_1.0.0/documentation/rest-clients.md) to use this library**

> NOTE: Form Recognizer has been rebranded to Document Intelligence. Please check the [Migration Guide from `@azure/ai-form-recognizer` to `@azure-rest/ai-document-intelligence`](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/documentintelligence/ai-document-intelligence-rest/MIGRATION-FR_v4-DI_v1.md).

Key links:

- [Source code](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/documentintelligence/ai-document-intelligence-rest)
- [Package (NPM)](https://www.npmjs.com/package/@azure-rest/ai-document-intelligence)
- [API reference documentation](/javascript/api/@azure-rest/ai-document-intelligence?view=azure-node-preview)
- [Samples](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/documentintelligence/ai-document-intelligence-rest/samples)
- [Changelog](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/documentintelligence/ai-document-intelligence-rest/CHANGELOG.md)
- [Migration Guide from Form Recognizer](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/documentintelligence/ai-document-intelligence-rest/MIGRATION-FR_v4-DI_v1.md)

> This version of the client library defaults to the `"2024-11-30"` version of the service.

This table shows the relationship between SDK versions and supported API versions of the service:

| SDK version  | Supported API version of service |
| ------------ | -------------------------------- |
| 1.0.0        | 2024-11-30                       |

> Please rely on the older `@azure/ai-form-recognizer` library through the older service API versions for retired models, such as `"prebuilt-businessCard"` and `"prebuilt-document"`. For more information, see [Changelog](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/documentintelligence/ai-document-intelligence-rest/CHANGELOG.md).

The below table describes the relationship of each client and its supported API version(s):

| Service API version | Supported clients                                            | Package                                                       |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------- |
| 2024-11-30          | DocumentIntelligenceClient                                   | `@azure-rest/ai-document-intelligence` version `1.0.0`        |
| 2023-07-31          | DocumentAnalysisClient and DocumentModelAdministrationClient | `@azure/ai-form-recognizer` version `^5.0.0`                  |
| 2022-08-01          | DocumentAnalysisClient and DocumentModelAdministrationClient | `@azure/ai-form-recognizer` version `^4.0.0`                  |

## Getting started

### Currently supported environments

- LTS versions of Node.js

### Prerequisites

- You must have an [Azure subscription](https://azure.microsoft.com/free/) to use this package.

### Install the `@azure-rest/ai-document-intelligence` package

Install the Azure DocumentIntelligence(formerlyFormRecognizer) REST client REST client library for JavaScript with `npm`:

```bash
npm install @azure-rest/ai-document-intelligence
```

### Create and authenticate a `DocumentIntelligenceClient`

To use an [Azure Active Directory (AAD) token credential](https://github.com/Azure/azure-sdk-for-js/blob/@azure-rest/ai-document-intelligence_1.0.0/sdk/identity/identity/samples/AzureIdentityExamples.md#authenticating-with-a-pre-fetched-access-token),
provide an instance of the desired credential type obtained from the
[@azure/identity](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/identity/identity#credentials) library.

To authenticate with AAD, you must first `npm` install [`@azure/identity`](https://www.npmjs.com/package/@azure/identity)

After setup, you can choose which type of [credential](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/identity/identity#credentials) from `@azure/identity` to use.
As an example, [DefaultAzureCredential](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/identity/identity#defaultazurecredential)
can be used to authenticate the client.

Set the values of the client ID, tenant ID, and client secret of the AAD application as environment variables:
AZURE_CLIENT_ID, AZURE_TENANT_ID, AZURE_CLIENT_SECRET

### Using a Token Credential

```ts
import DocumentIntelligence from "@azure-rest/ai-document-intelligence";

const client = DocumentIntelligence(
  process.env["DOCUMENT_INTELLIGENCE_ENDPOINT"],
  new DefaultAzureCredential()
);
```

### Using an API KEY

```ts
import DocumentIntelligence from "@azure-rest/ai-document-intelligence";

const client = DocumentIntelligence(process.env["DOCUMENT_INTELLIGENCE_ENDPOINT"], {
  key: process.env["DOCUMENT_INTELLIGENCE_API_KEY"],
});
```

## Document Models

### Analyze prebuilt-layout (urlSource)

```ts
const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-layout")
  .post({
    contentType: "application/json",
    body: {
      urlSource:
        "https://raw.githubusercontent.com/Azure/azure-sdk-for-js/6704eff082aaaf2d97c1371a28461f512f8d748a/sdk/formrecognizer/ai-form-recognizer/assets/forms/Invoice_1.pdf",
    },
    queryParameters: { locale: "en-IN" },
  });
```

### Analyze prebuilt-layout (base64Source)

```ts
import fs from "fs";
import path from "path";

const filePath = path.join(ASSET_PATH, "forms", "Invoice_1.pdf");
const base64Source = fs.readFileSync(filePath, { encoding: "base64" });
const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-layout")
  .post({
    contentType: "application/json",
    body: {
      base64Source,
    },
    queryParameters: { locale: "en-IN" },
  });
```

Continue creating the poller from initial response

```ts
import {
  getLongRunningPoller,
  AnalyzeResultOperationOutput,
  isUnexpected,
} from "@azure-rest/ai-document-intelligence";

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}
const poller = getLongRunningPoller(client, initialResponse);
const result = (await poller.pollUntilDone()).body as AnalyzeResultOperationOutput;
console.log(result);
// {
//   status: 'succeeded',
//   createdDateTime: '2023-11-10T13:31:31Z',
//   lastUpdatedDateTime: '2023-11-10T13:31:34Z',
//   analyzeResult: {
//     apiVersion: '2023-10-31-preview',
//     .
//     .
//     .
//     contentFormat: 'text'
//   }
// }
```

## Batch analysis

```ts
import { parseResultIdFromResponse, isUnexpected } from "@azure-rest/ai-document-intelligence";

// 1. Analyze a batch of documents
const initialResponse = await client
  .path("/documentModels/{modelId}:analyzeBatch", "prebuilt-layout")
  .post({
    contentType: "application/json",
    body: {
      azureBlobSource: {
        containerUrl: batchTrainingFilesContainerUrl(),
      },
      resultContainerUrl: batchTrainingFilesResultContainerUrl(),
      resultPrefix: "result",
    },
  });

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}
const resultId = parseResultIdFromResponse(initialResponse);
console.log("resultId: ", resultId);

// (Optional) You can poll for the batch analysis result but be aware that a job may take unexpectedly long time, and polling could incur additional costs.
// const poller = getLongRunningPoller(client, initialResponse);
// await poller.pollUntilDone();

// 2. At a later time, you can retrieve the operation result using the resultId
const output = await client
  .path("/documentModels/{modelId}/analyzeResults/{resultId}", "prebuilt-layout", resultId)
  .get();
console.log(output);
```

### Markdown content format

Supports output with Markdown content format along with the default plain _text_. For now, this is only supported for "prebuilt-layout". Markdown content format is deemed a more friendly format for LLM consumption in a chat or automation use scenario.

Service follows the GFM spec ([GitHub Flavored Markdown](https://github.github.com/gfm/)) for the Markdown format. Also introduces a new _contentFormat_ property with value "text" or "markdown" to indicate the result content format.

```ts
import DocumentIntelligence from "@azure-rest/ai-document-intelligence";
const client = DocumentIntelligence(process.env["DOCUMENT_INTELLIGENCE_ENDPOINT"], {
  key: process.env["DOCUMENT_INTELLIGENCE_API_KEY"],
});

const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-layout")
  .post({
    contentType: "application/json",
    body: {
      urlSource:
        "https://raw.githubusercontent.com/Azure/azure-sdk-for-js/6704eff082aaaf2d97c1371a28461f512f8d748a/sdk/formrecognizer/ai-form-recognizer/assets/forms/Invoice_1.pdf",
    },
    queryParameters: { outputContentFormat: "markdown" }, // <-- new query parameter
  });
```

### Query Fields

When this feature flag is specified, the service will further extract the values of the fields specified via the queryFields query parameter to supplement any existing fields defined by the model as fallback.

```ts
await client.path("/documentModels/{modelId}:analyze", "prebuilt-layout").post({
  contentType: "application/json",
  body: { urlSource: "..." },
  queryParameters: {
    features: ["queryFields"],
    queryFields: ["NumberOfGuests", "StoreNumber"],
  }, // <-- new query parameter
});
```

### Split Options

In the previous API versions supported by the older `@azure/ai-form-recognizer` library, document splitting and classification operation (`"/documentClassifiers/{classifierId}:analyze"`) always tried to split the input file into multiple documents.

To enable a wider set of scenarios, service introduces a "split" query parameter with the new "2023-10-31-preview" service version. The following values are supported:

- `split: "auto"`

  Let service determine where to split.

- `split: "none"`

  The entire file is treated as a single document. No splitting is performed.

- `split: "perPage"`

  Each page is treated as a separate document. Each empty page is kept as its own document.

## Document Classifiers #Build

```ts
import {
  DocumentClassifierBuildOperationDetailsOutput,
  getLongRunningPoller,
  isUnexpected,
} from "@azure-rest/ai-document-intelligence";

const containerSasUrl = (): string =>
  process.env["DOCUMENT_INTELLIGENCE_TRAINING_CONTAINER_SAS_URL"];
const initialResponse = await client.path("/documentClassifiers:build").post({
  body: {
    classifierId: `customClassifier${getRandomNumber()}`,
    description: "Custom classifier description",
    docTypes: {
      foo: {
        azureBlobSource: {
          containerUrl: containerSasUrl(),
        },
      },
      bar: {
        azureBlobSource: {
          containerUrl: containerSasUrl(),
        },
      },
    },
  },
});

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}
const poller = getLongRunningPoller(client, initialResponse);
const response = (await poller.pollUntilDone())
  .body as DocumentClassifierBuildOperationDetailsOutput;
console.log(response);
//  {
//    operationId: '31466834048_f3ee629e-73fb-48ab-993b-1d55d73ca460',
//    kind: 'documentClassifierBuild',
//    status: 'succeeded',
//    .
//    .
//    result: {
//      classifierId: 'customClassifier10978',
//      createdDateTime: '2023-11-09T12:45:56Z',
//      .
//      .
//      description: 'Custom classifier description'
//    },
//    apiVersion: '2023-10-31-preview'
//  }
```

## Get the generated PDF output from document analysis

```ts
const filePath = path.join(ASSET_PATH, "layout-pageobject.pdf");

const base64Source = await fs.readFile(filePath, { encoding: "base64" });

const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-read")
  .post({
    contentType: "application/json",
    body: {
      base64Source,
    },
    queryParameters: { output: ["pdf"] },
  });

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

const poller = getLongRunningPoller(client, initialResponse);

await poller.pollUntilDone();

const output = await client
  .path(
    "/documentModels/{modelId}/analyzeResults/{resultId}/pdf",
    "prebuilt-read",
    parseResultIdFromResponse(initialResponse)
  )
  .get()
  .asNodeStream(); // output.body would be NodeJS.ReadableStream

if (output.status !== "200" || !output.body) {
  throw new Error("The response was unexpected, expected NodeJS.ReadableStream in the body.");
}

const pdfData = await streamToUint8Array(output.body);
fs.promises.writeFile(`./output.pdf`, pdfData);
// Or you can consume the NodeJS.ReadableStream directly
```

## Get the generated cropped image of specified figure from document analysis

```ts
const filePath = path.join(ASSET_PATH, "layout-pageobject.pdf");

const base64Source = fs.readFileSync(filePath, { encoding: "base64" });

const initialResponse = await client
  .path("/documentModels/{modelId}:analyze", "prebuilt-layout")
  .post({
    contentType: "application/json",
    body: {
      base64Source,
    },
    queryParameters: { output: ["figures"] },
  });

if (isUnexpected(initialResponse)) {
  throw initialResponse.body.error;
}

const poller = getLongRunningPoller(client, initialResponse, { ...testPollingOptions });

const result = (await poller.pollUntilDone()).body as AnalyzeResultOperationOutput;
const figures = result.analyzeResult?.figures;
assert.isArray(figures);
assert.isNotEmpty(figures?.[0]);
const figureId = figures?.[0].id || "";
assert.isDefined(figureId);

const output = await client
  .path(
    "/documentModels/{modelId}/analyzeResults/{resultId}/figures/{figureId}",
    "prebuilt-layout",
    parseResultIdFromResponse(initialResponse),
    figureId
  )
  .get()
  .asNodeStream(); // output.body would be NodeJS.ReadableStream

if (output.status !== "200" || !output.body) {
  throw new Error("The response was unexpected, expected NodeJS.ReadableStream in the body.");
}

const imageData = await streamToUint8Array(output.body);
fs.promises.writeFile(`./figures/${figureId}.png`, imageData);
// Or you can consume the NodeJS.ReadableStream directly
```

## Get Info

```ts
const response = await client.path("/info").get();
if (isUnexpected(response)) {
  throw response.body.error;
}
console.log(response.body.customDocumentModels.limit);
// 20000
```

## List Document Models

```ts
import { paginate } from "@azure-rest/ai-document-intelligence";
const response = await client.path("/documentModels").get();
if (isUnexpected(response)) {
  throw response.body.error;
}

const modelsInAccount: string[] = [];
for await (const model of paginate(client, response)) {
  console.log(model.modelId);
}
```

## Troubleshooting

### Logging

Enabling logging may help uncover useful information about failures. In order to see a log of HTTP requests and responses, set the `AZURE_LOG_LEVEL` environment variable to `info`. Alternatively, logging can be enabled at runtime by calling `setLogLevel` in the `@azure/logger`:

```javascript
const { setLogLevel } = require("@azure/logger");

setLogLevel("info");
```

For more detailed instructions on how to enable logs, you can look at the [@azure/logger package docs](https://github.com/Azure/azure-sdk-for-js/tree/@azure-rest/ai-document-intelligence_1.0.0/sdk/core/logger).


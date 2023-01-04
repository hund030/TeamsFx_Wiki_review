## Introduction
Azure API Management enables professional developers to publish their Teams App backend service as APIs, and easily export these APIs to the Power Platform (Power Apps and Power Automate) as custom connectors for discovery and consumption by citizen developers.

## Prerequisites
- A Teams tab app with Azure Functions. [Sample](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-tab-with-backend)
- A client AAD used to visit the backend API. Both the client app and the web API app must be registered in the same tenant.

## Add Azure API Management resource in Azure

1. Update the bicep file `infra/azure.bicep` to create API management resources.
```
param apimServiceName string = resourceBaseName
param apimProductName string = resourceBaseName
param apimServiceAuthServerName string = resourceBaseName
param apimApiName string = resourceBaseName

param publisherEmail string
param publisherName string
param apimServiceSku string
var authorizationEndpoint = uri(aadAppOauthAuthorityHost, '${aadAppTenantId}/oauth2/v2.0/authorize')
var tokenEndpoint = uri(aadAppOauthAuthorityHost, '${aadAppTenantId}/oauth2/v2.0/token')
var scope = '${aadApplicationIdUri}/.default'

resource apimService 'Microsoft.ApiManagement/service@2022-04-01-preview' = {
  name: apimServiceName
  location: location
  sku: {
    name: apimServiceSku
    capacity: 0
  }
  properties: {
    publisherEmail: publisherEmail
    publisherName: publisherName
  }
}

resource apimServiceProduct 'Microsoft.ApiManagement/service/products@2022-04-01-preview' = {
  parent: apimService
  name: apimProductName
  properties: {
    displayName: apimProductName
    description: 'Created by TeamsFx.'
    subscriptionRequired: false
    state: 'published'
  }
}

resource apimAuthorizationServers 'Microsoft.ApiManagement/service/authorizationServers@2022-04-01-preview' = {
  parent: apimService
  name: apimServiceAuthServerName
  properties: {
    displayName: apimServiceAuthServerName
    description: 'Created by TeamsFx.'
    clientRegistrationEndpoint: 'http://localhost'
    authorizationEndpoint: authorizationEndpoint
    authorizationMethods: [
      'GET'
      'POST'
    ]
    clientAuthenticationMethod: [
      'Body'
    ]
    tokenEndpoint: tokenEndpoint
    defaultScope: scope
    grantTypes: [
      'authorizationCode'
    ]
    bearerTokenSendingMethods: [
      'authorizationHeader'
    ]
    clientId: aadAppClientId
    clientSecret: aadAppClientSecret
  }
}

resource api 'Microsoft.ApiManagement/service/apis@2022-04-01-preview' = {
  parent: apimService
  name: apimApiName
  properties: {
    displayName: apimApiName
    apiRevision: '1'
    subscriptionRequired: false
    serviceUrl: 'https://apimts010330c5d0api.azurewebsites.net/api'
    path: apimApiName
    protocols: [
      'https'
    ]
    authenticationSettings: {
      oAuth2: {
        authorizationServerId: apimServiceAuthServerName
      }
    }
    subscriptionKeyParameterNames: {
      header: 'Ocp-Apim-Subscription-Key'
      query: 'subscription-key'
    }
    isCurrent: true
  }
}

```
2. Update file `infra/azure.parameters.json` to add the new parameters.
```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "publisherEmail": {
      "value": "sample@sample.com"
    },
    "publisherName": {
      "value": "sample@sample.com"
    },
    "apimServiceSku": {
      "value": "Consumption"
    }
  }
}
```
3. Add the client id of the client AAD to the `authorizedClientApplicationIds` of Function App.
```
var apimClientAppClientId = '...'
var authorizedClientApplicationIds = '${apimClientAppClientId};${teamsMobileOrDesktopAppClientId};
```

## Configure the AAD created by Teams Toolkit
1. Add the client id of the client AAD to the `knownClientApplications` in the file `aad.manifest.template.json`.
```
{
    "id": "${{AAD_APP_OBJECT_ID}}",
    "appId": "${{AAD_APP_CLIENT_ID}}",
    "knownClientApplications": [
		"${{CLIENT_APP_CLIENT_ID}}"
	],
    ...
}
```
## Provision the resources
Run Teams: Provision in the cloud command in Visual Studio Code to apply the bicep to Azure.

## Configure the client APP registration to visit the backend API
1. To configure the client APP registration to access the Teams App Backend API, the permission of Teams App should be added to the client AAD. [Here](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-access-web-apis) are the detail steps.
2. Enable hybrid flows of the client APP registration in the portal.


## Deploy the API in API management
You can upload the OpenAPI specification of the backend API in Azure Portal according to this [document](https://learn.microsoft.com/en-us/azure/api-management/import-and-publish#import-and-publish-a-backend-api).

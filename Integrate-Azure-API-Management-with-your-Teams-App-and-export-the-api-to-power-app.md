# Integrate Azure API Management with your Teams App and export the api to power app

## Introduction
Azure API Management enables professional developers to publish their Teams App backend service as APIs, and easily export these APIs to the Power Platform (Power Apps and Power Automate) as custom connectors for discovery and consumption by citizen developers.

## Prerequisites
- A Teams tab app with Azure Functions. [Sample](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-tab-with-backend)
- A client Azure AD app used to visit the backend API. You can register a new Azure AD app add a client secret according to this [document](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app). Both the client app and the web API app must be registered in the same tenant.

## Add Azure API Management resource in Azure
1. Add the the client id and client secret of the client Azure AD app to `env/.env.dev`.
   ```
   APIM_CLIENT_AAD_CLIENT_ID=<your-apim-client-aad-client-id>
   APIM_CLIENT_AAD_CLIENT_SECRET=<your-apim-client-aad-client-secret>
   ```
1. Update the parameter file `infra/azure.parameters.json`.
   ```
   {
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       ...
       "publisherEmail": {
         "value": "sample@sample.com"
       },
       "publisherName": {
         "value": "sample@sample.com"
       },
       "apimServiceSku": {
         "value": "Consumption"
       },
       "apimClientAadClientId":{
         "value": "${{APIM_CLIENT_AAD_CLIENT_ID}}"
       },
       "apimClientAadClientSecret":{
         "value": "${{APIM_CLIENT_AAD_CLIENT_SECRET}}"
       }
     }
   }
   ```
1. Update the bicep file `infra/azure.bicep`, add `apimClientAadClientId` to `authorizedClientApplicationIds` of the function app.
   ```
   param apimClientAadClientId string
   var authorizedClientApplicationIds = '${apimClientAadClientId};${teamsMobileOrDesktopAppClientId};${teamsWebAppClientId};${officeWebAppClientId1};${officeWebAppClientId2};${outlookDesktopAppClientId};${outlookWebAppClientId}'
   ```
1. Update the bicep file `infra/azure.bicep` to create API management resources.
    ```
    param apimServiceName string = resourceBaseName
    param apimProductName string = resourceBaseName
    param apimServiceAuthServerName string = resourceBaseName
    param apimApiName string = resourceBaseName

    param publisherEmail string
    param publisherName string
    param apimServiceSku string
    param apimClientAadClientSecret string

    var authorizationEndpoint = uri(aadAppOauthAuthorityHost, '${aadAppTenantId}/oauth2/v2.0/authorize')
    var tokenEndpoint = uri(aadAppOauthAuthorityHost, '${aadAppTenantId}/oauth2/v2.0/token')
    var scope = '${aadApplicationIdUri}/.default'
    var apimOpenApiDocument = loadTextContent('openapi.json')

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
        clientId: apimClientAadClientId
        clientSecret: apimClientAadClientSecret
      }
    }

    resource api 'Microsoft.ApiManagement/service/apis@2022-04-01-preview' = {
      parent: apimService
      name: apimApiName
      properties: {
        displayName: apimApiName
        apiRevision: '1'
        subscriptionRequired: false
        serviceUrl: '${apiEndpoint}/api'
        path: apimApiName
        protocols: [
          'https'
        ]
        authenticationSettings: {
          oAuth2AuthenticationSettings: [
            {
              authorizationServerId: apimServiceAuthServerName
            }
          ]
          openidAuthenticationSettings: []
        }
        subscriptionKeyParameterNames: {
          header: 'Ocp-Apim-Subscription-Key'
          query: 'subscription-key'
        }
        isCurrent: true
        format: 'openapi+json'
        value: apimOpenApiDocument
      }
      dependsOn: [
        apimAuthorizationServers
      ]
    }
    ```

## Configure the AAD created by Teams Toolkit
Add the client id of the client AAD to the `knownClientApplications` in the file `aad.manifest.json`.
```
{
  "id": "${{AAD_APP_OBJECT_ID}}",
  "appId": "${{AAD_APP_CLIENT_ID}}",
  "knownClientApplications": [
    "${{APIM_CLIENT_AAD_CLIENT_ID}}"
  ],
  ...
}
```
## Add the OpenAPI document
Add the OpenAPI document of the backend APIs to `infra/openapi.json`. The following is an example.
```json
{
    "openapi": "3.0.1",
    "info": {
        "title": "sample-backend-api",
        "version": "v1"
    },
    "paths": {
        "/getUserProfile": {
            "get": {
                "summary": "Get User Profile",
                "operationId": "get-user-profile",
                "responses": {
                    "200": {
                        "description": "200 response",
                        "content": {
                            "application/json": {
                                "schema": {
                                    "type": "object",
                                    "properties": {
                                        "receivedHTTPRequestBody": {
                                            "type": "string"
                                        },
                                        "userInfoMessage": {
                                            "type": "string"
                                        },
                                        "graphClientMessage": {
                                            "type": "object"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

## Provision the resources
Run Teams: Provision in the cloud command in Visual Studio Code to apply the bicep to Azure.

## Create the service principle for the AAD created by Teams Toolkit
1. The value of `AAD_APP_CLIENT_ID` can be found in environment file `env/.env.dev`.
1. Create the service principle.
   - Use Azure CLI
     Login with M365 account.
     ```
     az login
     az ad sp create --id ${AAD_APP_CLIENT_ID}
     ```
   - Use Azure Portal
     1. Sign in to the [Azure portal](https://portal.azure.com/).
     1. Search for and select Azure Active Directory.
     1. Under Manage, select App registrations.
     1. Select your application with client id `AAD_APP_CLIENT_ID`.
     1. Click `Create Service Principle`.
     ![image](https://user-images.githubusercontent.com/49138419/210941316-512c01ca-7beb-4544-bd85-e435a54f81fa.png)

## Configure the client APP registration to visit the backend API
To configure the client APP registration to access the Teams App Backend API, the permission of Teams App should be added to the client AAD. 
1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Search for and select Azure Active Directory.
1. Under Manage, select App registrations.
1. Select your client application with client id `APIM_CLIENT_AAD_CLIENT_ID`.
1. Select API permissions > Add a permission > My APIs.
1. Select the web API with the client id `AAD_APP_CLIENT_ID` which registered by Teams Toolkit.
![add api](https://user-images.githubusercontent.com/49138419/210943559-03433abf-acdd-438f-8b91-ad1b6f2a272a.jpg)

You can learn more from [Here](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-access-web-apis).

## Deploy the API in API management
You can upload the OpenAPI specification of the backend API in Azure Portal according to this 
 [document](https://learn.microsoft.com/en-us/azure/api-management/import-and-publish#import-and-publish-a-backend-api).

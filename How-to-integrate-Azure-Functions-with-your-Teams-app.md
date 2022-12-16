# Integrate Azure-Functions-with-your-Teams-app

## Introduction

Azure Functions is a server-less, event-driven compute solution that allows you to write less code.
It's a great way to add server-side behaviors to any Teams application.
Learn more from [Azure Functions Overview](https://learn.microsoft.com/azure/azure-functions/functions-overview)

## Prerequisites

- [Install Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local).
- [.NET SDK 6.0](https://dotnet.microsoft.com/download/dotnet/6.0), which is required by TeamsFx binding extension for authorization.
- A Teams tab app with Single Sign On enabled. [Enabled SSO for Teams tab app](https://aka.ms/teamsfx-add-sso-readme)

## Add your first Function to your Teams app

Create a function app project with HTTP trigger in the `api` folder with Azure Function Core Tools.

    ```
    > mkdir api
    > cd ./api
    > func new --template "Http Trigger" --name getUserProfile
    ```

After adding function app project, your folder structure may be like:

    ```
    .
    |-- .vscode/
    |-- teasmfx/
    |-- infra/
    |-- api/                      <!--function app source code-->
    |   |-- getUserProfile/       <!--HTTP trigger name-->
    |   |   |-- function.json
    |   |   |-- index.ts
    |   |-- package.json
    |-- src/                      <!--your current source code-->
    |   |-- index.ts
    |-- package.json
    ```

## Setup local debug environment in VSC

You can find a complete sample debug profile for VSC. [here](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-tab-with-backend/.vscode).

1. In `launch.json` file, add `Attach to Backend` configuration and ensure the it is cascaded by `Attach to Frontend` and be depended by `Debug` compounds.

      ```json
      "configurations": [
        ...
        {
          "name": "Attach to Frontend (Edge)",
          "cascadeTerminateToConfigurations": [
            "Attach to Backend"
          ],
          ...
        },
        {
          "name": "Attach to Backend",
          "type": "node",
          "request": "attach",
          "port": 9229,
          "restart": true,
          "presentation": {
            "group": "all",
            "hidden": true
          },
          "internalConsoleOptions": "neverOpen"
        }
      ],
      "compounds": [
        {
          "name": "Debug (Edge)",
          "configurations": [
            "Attach to Frontend (Edge)",
            "Attach to Backend"
          ],
        },
        ...
      ]
      ```

1. In `tasks.json` file, add `Start backend` and `Watch backend` tasks, and ensure `Start services` is depended by `Start services` task.

    ```
    {
      "label": "Start backend",
      "type": "shell",
      "command": "npm run dev:teamsfx",
      "isBackground": true,
      "options": {
        "cwd": "${workspaceFolder}/api",
        "env": {
          "PATH": "${command:fx-extension.get-func-path}${env:PATH}"
        }
      },
      "problemMatcher": {
        "pattern": {
          "regexp": "^.*$",
          "file": 0,
          "location": 1,
          "message": 2
        },
        "background": {
          "activeOnStart": true,
          "beginsPattern": "^.*(Job host stopped|signaling restart).*$",
          "endsPattern": "^.*(Worker process started and initialized|Host lock lease acquired by instance ID).*$"
        }
      },
      "presentation": {
        "reveal": "silent"
      },
      "dependsOn": ["Watch backend"]
    },
    {
      "label": "Watch backend",
      "type": "shell",
      "command": "npm run watch",
      "isBackground": true,
      "options": {
        "cwd": "${workspaceFolder}/api"
      },
      "problemMatcher": "$tsc-watch",
      "presentation": {
        "reveal": "silent"
      }
    }
    ```

1. Copy the script [run.api.js](https://github.com/OfficeDev/TeamsFx-Samples/blob/v3/hello-world-tab-with-backend/teamsfx/script/run.api.js) to `./teamsfx/script` folder, and add NPM scripts for the function app start.

      ```json
      "scripts": {
        "dev:teamsfx": "node ../teamsfx/script/run.api.js ../ ../teamsfx/.env.local",
        "dev": "func start --typescript --language-worker=\"--inspect=9229\" --port \"7071\" --cors \"*\"",
        ...
      }
      ```

1. Add a cli/runNpmCommand action in deploy stage in `./teamsfx/app.local.yml` file. This action trigger `npm install` before launching your function app.

      ```yml
      deploy:
        - uses: cli/runNpmCommand # Run npm command
          with:
            args: install --no-audit
            workingDirectory: ./api
      ```

1. Start debugging, you will find your tab app launching with function app running behind.

## Add Authorization for http trigger

Yet, your Azure Function app is public to any client.
With [TeamsFx binding extension](https://github.com/OfficeDev/TeamsFx/tree/main/packages/function-extension), your function is able to reject unauthorized client.
Here are the steps to add authorization.

1. Remove the `extensionBundle` section in `host.json` file.
1. Install TeamsFx binding extension.

    ```
    > cd api
    > func extensions install --package Microsoft.Azure.WebJobs.Extensions.TeamsFx --version 1.0.*
    ```

1. Refer TeamsFx binding in `function.json`.

    ```json
    {
      "bindings": [
        ...,
        {
          "direction": "in",
          "name": "teamsfxContext",
          "type": "TeamsFx"
        }
      ]
    }
    ```

Remember in [run.api.js](#setup-local-debug-environment-in-vsc), we have set variables M365_CLIENT_ID and ALLOWED_APP_IDS in environment.
Now the Function App can only be called by those client whose client id is in the list of ALLOWED_APP_IDS or equals to M365_CLIENT_ID.
M365_CLIENT_ID should be the client id of your Teams app. So that your Teams tab app is able to call your function.

## Call the function from your client with TeamsFx SDK

1. We recommend setting function endpoint and function name in environment variables. In `teamsfx/app.local.yml` file, find the action `file/updateEnv` and add new envs.
    ```yml
    - uses: file/updateEnv # Generate env to .env file
      with:
        envs:
          TAB_DOMAIN: localhost:53000
          TAB_ENDPOINT: https://localhost:53000
          FUNC_NAME: getUserProfile
          FUNC_ENDPOINT: http://localhost:7071
    ```

    Then update `teamsfx/script/run.js` script to set the environment variable in tab app service process.
    M365_CLIENT_ID should already be set to environment.

    ```js
    process.env.REACT_APP_FUNC_NAME = envs.FUNC_NAME;
    process.env.REACT_APP_FUNC_ENDPOINT = envs.FUNC_ENDPOINT;
    ```

1. Call your Azure Function with TeamsFx SDK.

   ```ts
   const functionName = process.env.REACT_APP_FUNC_NAME;
   const functionEndpoint = process.env.REACT_APP_FUNC_ENDPOINT;

   const teamsfx = useContext(TeamsFxContext).teamsfx
   const credential = teamsfx.getCredential();
   const apiClient = createApiClient(
      `${functionEndpoint}/api/`,
      new BearerTokenAuthProvider(async () => (await credential.getToken(""))!.token));
    const response = await apiClient.get(functionName);
   ```

   The `createApiClient` will handle the authorization header. You can find a complete React component for calling Azure Function [here](https://github.com/OfficeDev/TeamsFx-Samples/blob/v3/hello-world-tab-with-backend/src/components/sample/AzureFunctions.tsx).

1. Start debugging to test the logic.

## Call Graph API with TeamsFx SDK

You can find a complete sample [here](https://github.com/OfficeDev/TeamsFx-Samples/blob/v3/hello-world-tab-with-backend/api/getUserProfile/index.ts).

1. Install TeamsFx SDK and isomorphic-fetch, which is required by msgraph-sdk-javascript. [More information](https://www.npmjs.com/package/@microsoft/microsoft-graph-client#installation)

   ```
   > cd api/
   > npm i @microsoft/teamsfx & isomorphic-fetch
   ```

1. Here is an example for calling the Graph API with TeamsFx SDK. We have set the environment variables in [run.api.js](#setup-local-debug-environment-in-vsc).

    ```ts
    import "isomorphic-fetch";
    import {
      createMicrosoftGraphClientWithCredential,
      OnBehalfOfCredentialAuthConfig,
      OnBehalfOfUserCredential,
    } from "@microsoft/teamsfx";

    const httpTrigger: AzureFunction = async function (
      context: Context,
      req: HttpRequest,
      teamsfxContext: { [key: string]: any },
    ): Promise<void> {
      // Get accessToken from TeamsFxContext.
      const accessToken = teamsfxContext["AccessToken"];
      const oboAuthConfig: OnBehalfOfCredentialAuthConfig = {
        authorityHost: process.env.M365_AUTHORITY_HOST,
        tenantId: process.env.M365_TENANT_ID,
        clientId: process.env.M365_CLIENT_ID,
        clientSecret: process.env.M365_CLIENT_SECRET,
      };
      const oboCredential = new OnBehalfOfUserCredential(accessToken, oboAuthConfig);
      // Create a graph client with default scope to access user's Microsoft 365 data after user has consented.
      const graphClient = createMicrosoftGraphClientWithCredential(
        oboCredential,
        [".default"]
      );
      const profile: any = await graphClient.api("/me").get();
      context.res.body = { graphClientMessage: profile };
    }
    ```

## Move the application to Azure

1. Update bicep and azure.parameter.json to configure Azure Function App. You will need a Storage Account, an App Service Plan and a Function App Service.
You can find the complete sample [here](https://github.com/OfficeDev/TeamsFx-Samples/blob/v3/hello-world-tab-with-backend/infra/azure.bicep).

    ```
    param resourceBaseName string
    param functionStorageSKU string
    param functionAppSKU string

    param aadAppClientId string
    param aadAppTenantId string
    param aadAppOauthAuthorityHost string
    @secure()
    param aadAppClientSecret string

    param location string = resourceGroup().location
    param serverfarmsName string = resourceBaseName
    param functionAppName string = resourceBaseName
    param functionStorageName string = '${resourceBaseName}api'
    var oauthAuthority = uri(aadAppOauthAuthorityHost, aadAppTenantId)

    var tabEndpoint = ${TabAppEndpoint}
    var aadApplicationIdUri = 'api://${TabAppDomain}/${aadAppClientId}'

    // Compute resources for Azure Functions
    resource serverfarms 'Microsoft.Web/serverfarms@2021-02-01' = {
      name: serverfarmsName
      location: location
      sku: {
        name: functionAppSKU // You can follow https://aka.ms/teamsfx-bicep-add-param-tutorial to add functionServerfarmsSku property to provisionParameters to override the default value "Y1".
      }
      properties: {}
    }

    // Azure Functions that hosts your function code
    resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
      name: functionAppName
      kind: 'functionapp'
      location: location
      properties: {
        serverFarmId: serverfarms.id
        httpsOnly: true
        siteConfig: {
          alwaysOn: true
          cors: {
            allowedOrigins: [ tabEndpoint ]
          }
          appSettings: [
            {
              name: ' AzureWebJobsDashboard'
              value: 'DefaultEndpointsProtocol=https;AccountName=${functionStorage.name};AccountKey=${listKeys(functionStorage.id, functionStorage.apiVersion).keys[0].value};EndpointSuffix=${environment().suffixes.storage}' // Azure Functions internal setting
            }
            {
              name: 'AzureWebJobsStorage'
              value: 'DefaultEndpointsProtocol=https;AccountName=${functionStorage.name};AccountKey=${listKeys(functionStorage.id, functionStorage.apiVersion).keys[0].value};EndpointSuffix=${environment().suffixes.storage}' // Azure Functions internal setting
            }
            {
              name: 'FUNCTIONS_EXTENSION_VERSION'
              value: '~4' // Use Azure Functions runtime v4
            }
            {
              name: 'FUNCTIONS_WORKER_RUNTIME'
              value: 'node' // Set runtime to NodeJS
            }
            {
              name: 'WEBSITE_CONTENTAZUREFILECONNECTIONSTRING'
              value: 'DefaultEndpointsProtocol=https;AccountName=${storage.name};AccountKey=${listKeys(storage.id, storage.apiVersion).keys[0].value};EndpointSuffix=${environment().suffixes.storage}' // Azure Functions internal setting
            }
            {
              name: 'WEBSITE_RUN_FROM_PACKAGE'
              value: '1' // Run Azure Functions from a package file
            }
            {
              name: 'WEBSITE_NODE_DEFAULT_VERSION'
              value: '~16' // Set NodeJS version to 16.x
            }
            {
              name: 'M365_CLIENT_ID'
              value: aadAppClientId
            }
            {
              name: 'M365_CLIENT_SECRET'
              value: aadAppClientSecret
            }
            {
              name: 'M365_TENANT_ID'
              value: aadAppTenantId
            }
            {
              name: 'M365_AUTHORITY_HOST'
              value: aadAppOauthAuthorityHost
            }
            {
              name: 'M365_APPLICATION_ID_URI'
              value: aadApplicationIdUri
            }
          ]
          ftpsState: 'FtpsOnly'
        }
      }
    }
    var apiEndpoint = 'https://${functionApp.properties.defaultHostName}'

    resource authSettings 'Microsoft.Web/sites/config@2021-02-01' = {
      name: '${functionApp.name}/authsettings'
      properties: {
        enabled: true
        defaultProvider: 'AzureActiveDirectory'
        clientId: aadAppClientId
        issuer: '${oauthAuthority}/v2.0'
        allowedAudiences: [
          aadAppClientId
          aadApplicationIdUri
        ]
      }
    }

    // Azure Storage is required when creating Azure Functions instance
    resource functionStorage 'Microsoft.Storage/storageAccounts@2021-06-01' = {
      name: functionStorageName
      kind: 'StorageV2'
      location: location
      sku: {
        name: functionStorageSKU// You can follow https://aka.ms/teamsfx-bicep-add-param-tutorial to add functionStorageSKUproperty to provisionParameters to override the default value "Standard_LRS".
      }
    }

    // The output will be persisted in .env.{envName}. Visit https://aka.ms/teamsfx-provision-arm#output for more details.
    output API_FUNCTION_ENDPOINT string = apiEndpoint
    output API_FUNCTION_RESOURCE_ID string = functionApp.id
    ```

    Add new parameters in azure.parameter.json file.
    ```
    {
      "parameters": {
        "aadAppClientId": {
          "value": "${{AAD_APP_CLIENT_ID}}"
        },
        "aadAppClientSecret": {
          "value": "${{SECRET_AAD_APP_CLIENT_SECRET}}"
        },
        "aadAppTenantId": {
          "value": "${{AAD_APP_TENANT_ID}}"
        },
        "aadAppOauthAuthorityHost": {
          "value": "${{AAD_APP_OAUTH_AUTHORITY_HOST}}"
        },
        "functionAppSKU": {
          "value": "B1"
        },
        "functionStorageSKU": {
          "value": "Standard_LRS"
        }
      }
    }
    ```

1. Run `Teams: Provision in the cloud` command in Visual Studio Code to apply the bicep to Azure.

1. Add new actions in `teamsfx/app.yaml` to setup deployment.
You can find the complete sample [here](https://github.com/OfficeDev/TeamsFx-Samples/blob/v3/hello-world-tab-with-backend/teamsfx/app.yml)

    ```
    deploy:
      - uses: cli/runNpmCommand # Run npm command
        with:
          workingDirectory: ./api
          args: install
      - uses: cli/runNpmCommand # Run npm command
        with:
          workingDirectory: ./api
          args: run build --if-present
      - uses: azureFunctions/deploy
        with:
          # deploy base folder
          distributionPath: ./api
          # the resource id of the cloud resource to be deployed to
          resourceId: ${{API_FUNCTION_RESOURCE_ID}}
    ```

1. Add function name and function endpoint to tab app's environment, so that your tab app can call your function http trigger.
    ```
    deploy:
      - uses: cli/runNpmCommand # Run npm command
        env:
          REACT_APP_CLIENT_ID: ${{AAD_APP_CLIENT_ID}}
          REACT_APP_START_LOGIN_PAGE_URL: ${{TAB_ENDPOINT}}/auth-start.html
          REACT_APP_FUNC_NAME: getUserProfile
          REACT_APP_FUNC_ENDPOINT: ${{API_FUNCTION_ENDPOINT}}
    ```

1. Run `Teams: Deploy to cloud` command in Visual Studio Code to deploy your Tab app code to Azure.

1. Open the `Run and Debug Activity Panel` and select `Launch Remote (Edge)` or `Launch Remote (Chrome)`. Press F5 to preview your Teams app.


## What's next

- [Set up CI/CD pipelines](#How-to-add-CICD)
- [Integrate Azure Sql with your Teams app](#How-to-add-sql)

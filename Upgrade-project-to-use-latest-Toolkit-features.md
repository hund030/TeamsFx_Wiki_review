Teams Toolkit continuously evolves to offer more powerful features for developers. Recently a group of new features has been available to preview in insider features, such as provisioning Azure resources with ARM templates and managing multiple environments. These insider features will require an update to your existing project code structures. Teams Toolkit can automatically upgrade your original project to support Multiple Environments and ARM provision features.

> Please note that ARM and multiple environment features will be enabled by default in the future release of Teams Toolkit. We thrive to make Teams Toolkit compatible with existing projects but long time backward compatibility is not 100% guaranteed thus we strongly recommend you update your project configurations to continue using the latest Teams Toolkit.

## How to Upgrade
Teams Toolkit will automatically upgrade your original project folder structure and not change your custom code.

## File Structure Change
Once migration succeeds, your project file structure will be changed.

### Project Configuration Files
The existing project configuration files under the `.fx` folder are outdated and incompatible with the current version of Teams Toolkit. So some clean-ups are made and now your `.fx` folder will consist:
* `azure.parameters.*.json:` Parameters for Provisioning Azure Resource, specific for each environment.
* `config.*.json:` Configurations for Manifest, AAD, etc, specific for each environment.
* `projectSettings.json:` Project Settings, including capabilities, programming languages, etc.
* `localSettings.json:` Local Settings, including necessary information to start debugging the project locally.

We will update those files according to your original project settings and move existing ones into `.backup` folder for your reference. You can safely delete the `.backup` folder after you have compared and reviewed the changes.

### ARM Template and Manifest Template File.
Pre-cooked ARM templates tailored to your project will be automatically added under the `templates/azure` folder. The ARM templates are authored using [Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview). 

Teams app manifest files are located in `templates/appPackage` folder. The folder contains two manifest files:
* `manifest.local.template.json`: Manifest file for Teams app running locally.
* `manifest.remote.template.json`: Manifest file for Teams app running remotely (After deployed to Azure).

Both files contain template arguments with `{...}` statements which will be replaced at build time. You may add any extra properties or permissions you require to this file. See the [schema reference](https://docs.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema) for more information.

### Environment Specific Configuration Files
Teams Toolkit now supports creating multiple environments for a project, and you can customize the configurations for a specific environment, for example using different app names in the manifest for different environments. You can read more about what you can do in [this wiki](https://github.com/OfficeDev/TeamsFx/wiki/Enable-Preview-Features-in-Teams-Toolkit#managing-multiple-environments-in-teams-toolkit).

## What you should do after upgrading

### Reprovision Bot Project
If your project contains bot capability which has already been successfully provisioned and just got upgraded by Toolkit, the project requires re-provision bot resource. Since Bot Channels Registration become a [legacy production](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration?view=azure-bot-service-4.0&tabs=userassigned#create-the-resource), the toolkit helps upgrade Bot Channels Registration to Azure Bot Service, so you will be asked to provision again before deploying or publishing the project. If you still want to use the existing bot, please follow [these steps](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration?view=azure-bot-service-4.0&tabs=userassigned#create-the-resource).

Below are manual steps to use an existing Bot:
There you need to modify three files
1. Modify `./templates/azure/provision/bot.bicep` with following config  
    ```bicep
    resource botService 'Microsoft.BotService/botServices@2021-03-01' = {
      kind: 'bot'
      location: 'global'
      name: botServiceName
      properties: {
        displayName: botDisplayName
        endpoint: uri('https://${webApp.properties.defaultHostName}', '/api/messages')
        msaAppId: botAadAppClientId
      }
      sku: {
        name: 'F0'
      }
    }
    ```
2. Add four parameter to `.fx/configs/azure.parameter.dev.json`, and replace the placeholder with the target value in `.backup/.fx/env.default.json`.
    ```json
    "botServiceName": "${fx-resource-bot.botChannelReg}",
    "botDisplayName": "${fx-resource-bot.botChannelReg}",
    "botServerfarmsName": "${fx-resource-bot.appServicePlan}",
    "botSitesName": "${fx-resource-bot.siteName}"
    ```
3. In the `.fx/states/state.dev.json`,
you need to remove the existing fx-resource-bot object, and add following fx-resource-bot object, please replace the placeholder with the value in `.backup/.fx/env.default.json`
    ```json
    "fx-resource-bot": {
            "botId": "${fx-resource-bot.botId}",
            "botPassword": "{{fx-resource-bot.botPassword}}",
            "objectId": "${fx-resource-bot.objectId}",
            "skuName": "${fx-resource-bot.skuName}",
            "siteName": "${fx-resource-bot.siteName}",
            "validDomain": "${fx-resource-bot.validDomain}",
            "appServicePlanName": "${fx-resource-bot.appServicePlan}",
            "botChannelRegName": "${fx-resource-bot.botChannelReg}",
            "botWebAppResourceId": "/subscriptions/${solution.subscriptionId}/resourceGroups/${solution.resourceGroupName}/providers/Microsoft.Web/sites/${fx-resource-bot.siteName}",
            "siteEndpoint": "${fx-resource-bot.siteEndpoint}"
        }
    ```

### Customize APIM Service
Please note that **SKU**, **publisher email** and **publisher name** properties for APIM service will be updated based on the definition in `*./templates/azure/provision/apim.bicep*`, which might not be the same as your previous setting. If you prefer to keep previous values, you can customize the bicep file (*./templates/azure/provision/apim.bicep*) / parameter files (*.fx/configs/azure.parameters.dev.json*).

1. SKU, publisher name and publisher email of APIM service might be updated. To customize them, update SKU in *./templates/azure/provision/apim.bicep* directly and add `apimPublisherEmail` and `apimPublisherName` as customized parameters in *./.fx/configs/azure.parameters.dev.json*.
    ```bicep
    // ./templates/azure/provision/apim.bicep
    resource apimService 'Microsoft.ApiManagement/service@2020-12-01' = {
      name: apimServiceName
      location: resourceGroup().location
      sku: {
        name: '$your-customized-sku-name'
        capacity: 0
      }
      properties: {
        publisherEmail: publisherEmail
        publisherName: publisherName
      }
    }
    ```
    ```json
    //./.fx/configs/azure.parameters.dev.json
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "provisionParameters": {
                "value": {
                    ...
                    "apimPublisherEmail": "your-apim-publisher-email",
                    "apimPublisherName": "your-apim-publisher-name",
                }
            }
        }
    }
    ```
2. If you prefer using an existing APIM service and don't want the toolkit to update it. You can update Bicep file *./templates/azure/provision/apim.bicep* to use existing APIM service:
    ```bicep
    resource apimService 'Microsoft.ApiManagement/service@2020-12-01' existing = {
      name: apimServiceName
    }
    ```
    Same for APIM product service:
    ```bicep
    resource apimServiceProduct 'Microsoft.ApiManagement/service/products@2020-12-01' existing = {
      parent: apimService
      name: productName
    }
    ```

### Local Debug with SQL
Teams Toolkit support connecting to an Azure SQL database when local debug the Teams app.
You can connect to a SQL instance for frontend (tabs/), function (api/) and bot (bot/) components. There may already be a `.env.teamsfx.local` file under each component folder. If not you can create a new `.env.teamsfx.local` by yourself, and add the following environment variables in `.env.teamsfx.local` to specify the SQL connection information:

```
SQL_ENDPOINT=YOUR_SQL_ENDPOINT
SQL_DATABASE_NAME=YOUR_YOUR_DATABASE_NAME
SQL_USER_NAME=YOUR_SQL_USER_NAME
SQL_PASSWORD=YOUR_SQL_USER_PASSWORD
```

### Change resource name when creating a new environment
Teams Toolkit will reuse your provisioned resource when upgrading (except Bot Service), when you wish to add a new environment after project upgrading, please remember to change resource name in `azure.parameters.{your_env_name}.json` to avoid name conflicts. 

## Known Issues
* Local Debug will create a new teams App added to the Teams Developer Portal after migration success. You can get the app id from `.fx/configs/localSettings.json` file.
* Once upgrade success, if you provision resources in a new group resource using a newly created environment, this operation will cause an error. For example, you create a new environment named as test, in order to provision successful, just delete all parameters which value has exact value in  `.fx/configs/azure.parameters.test.json` 

## Upgrade your project manually
There are chances when upgrading will fail, you can still manually update configuration files to make sure it works with the Teams Toolkit, Here are the steps:
1. Create or update these three files: `env.default.json`, `settings.json`, `manifest.source.json` under `.fx` folder based on the details below.
2. Reload Visual Studio Code and click confirm in the upgrade dialog.


### For Azure Based App
1. Copy the following content to the file named **`env.default.json`** under the `.fx` folder. And make these updates if necessary:
  * If your project does not have `tabs` folder
    - Delete `trustDevCert`
  * If your project does not have `bot` folder
    - Delete `fx-resource-bot`
    - Delete `skipNgrok` and `localBotEndpoint` in fx-resource-local-debug
```json
{
    "solution": {},
    "fx-resource-frontend-hosting": {},
    "fx-resource-bot": {
        "skuName": "F1"
    },
    "fx-resource-aad-app-for-teams": {},
    "fx-resource-local-debug": {
        "trustDevCert": "{{fx-resource-local-debug.trustDevCert}}",
        "skipNgrok": "{{fx-resource-local-debug.skipNgrok}}",
        "localBotEndpoint": "{{fx-resource-local-debug.localBotEndpoint}}"
    },
    "fx-resource-appstudio": {},
    "fx-resource-simple-auth": {}
}
```

2. Copy the following content to the file named **`settings.json`** under the `.fx` folder. And make these updates if necessary:
   * Replace the value of `appName / projectId / programmingLanguage` with yours.
   * If your project does not have `tabs` folder:
     - Delete `Tab` in capabilities
     - Delete `fx-resource-frontend-hosting` and `fx-resource-simple-auth` in activeResourcePlugins
   * If your project does not have `bot` folder
    - Delete `Bot` and `MessagingExtension` in capabilities
    - Delete `fx-resource-bot` in activeResourcePlugins
```json
{
    "appName": "{your appName}",
    "projectId": "{your projectId}",
    "solutionSettings": {
        "name": "fx-solution-azure",
        "version": "1.0.0",
        "hostType": "Azure",
        "azureResources": [],
        "capabilities": [
            "Tab",
            "Bot",
            "MessagingExtension"
        ],
        "activeResourcePlugins": [
            "fx-resource-frontend-hosting",
            "fx-resource-bot",
            "fx-resource-aad-app-for-teams",
            "fx-resource-local-debug",
            "fx-resource-appstudio",
            "fx-resource-simple-auth"
        ]
    },
    "version": "2.0.0",
    "isFromSample": false,
    "programmingLanguage": "{javascript or typescript}"
}
```

3. Copy the following content to the file named **`manifest.source.json`** under the `appPackge` folder. And make these updates if necessary:
  * `color.png` and `outline.png` should be in the `appPackage` folder.
  * Replace `{your appName}` with your app name.
  * If your project does not have `tabs` folder
    - Replace `"configurableTabs": [...]` and `"staticTabs": [...]` with `"configurableTabs": []` and `"staticTabs": []`
  * If your project does not have `bot` folder
    - Replace `"bots": [...]` and `"composeExtensions": [...]` with `"bots": []` and `"composeExtensions": []`
```json
{
    "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.11/MicrosoftTeams.schema.json",
    "manifestVersion": "1.11",
    "version": "1.0.0",
    "id": "{appid}",
    "packageName": "com.microsoft.teams.extension",
    "developer": {
        "name": "Teams App, Inc.",
        "websiteUrl": "{baseUrl}",
        "privacyUrl": "{baseUrl}/index.html#/privacy",
        "termsOfUseUrl": "{baseUrl}/index.html#/termsofuse"
    },
    "icons": {
        "color": "color.png",
        "outline": "outline.png"
    },
    "name": {
        "short": "{your appName}",
        "full": "This field is not used"
    },
    "description": {
        "short": "Short description of {your appName}.",
        "full": "Full description of {your appName}."
    },
    "accentColor": "#FFFFFF",
    "bots": [
        {
            "botId": "{botId}",
            "scopes": [
                "personal",
                "team",
                "groupchat"
            ],
            "supportsFiles": false,
            "isNotificationOnly": false,
            "commandLists": [
                {
                    "scopes": [
                        "personal",
                        "team",
                        "groupchat"
                    ],
                    "commands": [
                        {
                            "title": "welcome",
                            "description": "Resend welcome card of this Bot"
                        },
                        {
                            "title": "learn",
                            "description": "Learn about Adaptive Card and Bot Command"
                        }
                    ]
                }
            ]
        }
    ],
    "composeExtensions": [
        {
            "botId": "{botId}",
            "commands": [
                {
                    "id": "createCard",
                    "context": [
                        "compose"
                    ],
                    "description": "Command to run action to create a Card from Compose Box",
                    "title": "Create Card",
                    "type": "action",
                    "parameters": [
                        {
                            "name": "title",
                            "title": "Card title",
                            "description": "Title for the card",
                            "inputType": "text"
                        },
                        {
                            "name": "subTitle",
                            "title": "Subtitle",
                            "description": "Subtitle for the card",
                            "inputType": "text"
                        },
                        {
                            "name": "text",
                            "title": "Text",
                            "description": "Text for the card",
                            "inputType": "textarea"
                        }
                    ]
                },
                {
                    "id": "shareMessage",
                    "context": [
                        "message"
                    ],
                    "description": "Test command to run action on message context (message sharing)",
                    "title": "Share Message",
                    "type": "action",
                    "parameters": [
                        {
                            "name": "includeImage",
                            "title": "Include Image",
                            "description": "Include image in Hero Card",
                            "inputType": "toggle"
                        }
                    ]
                },
                {
                    "id": "searchQuery",
                    "context": [
                        "compose",
                        "commandBox"
                    ],
                    "description": "Test command to run query",
                    "title": "Search",
                    "type": "query",
                    "parameters": [
                        {
                            "name": "searchQuery",
                            "title": "Search Query",
                            "description": "Your search query",
                            "inputType": "text"
                        }
                    ]
                }
            ],
            "messageHandlers": [
                {
                    "type": "link",
                    "value": {
                        "domains": [
                            "*.botframework.com"
                        ]
                    }
                }
            ]
        }
    ],
    "configurableTabs": [
        {
            "configurationUrl": "{baseUrl}/index.html#/config",
            "canUpdateConfiguration": true,
            "scopes": [
                "team",
                "groupchat"
            ]
        }
    ],
    "staticTabs": [
        {
            "entityId": "index",
            "name": "Personal Tab",
            "contentUrl": "{baseUrl}/index.html#/tab",
            "websiteUrl": "{baseUrl}/index.html#/tab",
            "scopes": [
                "personal"
            ]
        }
    ],
    "permissions": [
        "identity",
        "messageTeamMembers"
    ],
    "validDomains": [],
    "webApplicationInfo": {
        "id": "{appClientId}",
        "resource": "{webApplicationInfoResource}"
    }
}
```

### For SharePoint Based App

Copy the following content to the file named **`env.default.json`** under the `.fx` folder.
```json
{
    "solution": {},
    "fx-resource-spfx": {},
    "fx-resource-local-debug": {},
    "fx-resource-appstudio": {}
}
```

Copy the following content to the file named **`settings.json`** under the `.fx` folder. And make these updates if necessary:
  * Replace the value of `appName / projectId / programmingLanguage` with yours.

```json
{
    "appName": "{your appName}",
    "projectId": "{your projectId}",
    "solutionSettings": {
        "name": "fx-solution-azure",
        "version": "1.0.0",
        "hostType": "SPFx",
        "azureResources": [],
        "capabilities": [
            "Tab"
        ],
        "activeResourcePlugins": [
            "fx-resource-spfx",
            "fx-resource-local-debug",
            "fx-resource-appstudio"
        ]
    },
    "version": "2.0.0",
    "isFromSample": false,
    "programmingLanguage": "typescript"
}
```

Copy the following content to the file named **`manifest.source.json`** under the `appPackge` folder. And make these updates if necessary:
  * `color.png` and `outline.png` should be in the `appPackage` folder.
  * Replace `{your web part name}` with your app name.
  * Replace `{your entityId}` with your entity id.
```json
{
    "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.11/MicrosoftTeams.schema.json",
    "manifestVersion": "1.11",
    "packageName": "{your entityId}",
    "id": "{your appid}",
    "version": "1.0.0",
    "developer": {
        "name": "SPFx + Teams Dev",
        "websiteUrl": "https://products.office.com/en-us/sharepoint/collaboration",
        "privacyUrl": "https://privacy.microsoft.com/en-us/privacystatement",
        "termsOfUseUrl": "https://www.microsoft.com/en-us/servicesagreement"
    },
    "name": {
        "short": "{your web part name}"
    },
    "description": {
        "short": "{your web part name}",
        "full": "{your web part name}"
    },
    "icons": {
        "outline": "outline.png",
        "color": "color.png"
    },
    "accentColor": "#004578",
    "staticTabs": [
        {
            "entityId": "{your entityId}",
            "name": "{your web part name}",
            "contentUrl": "https://{teamSiteDomain}/_layouts/15/TeamsLogon.aspx?SPFX=true&dest=/_layouts/15/teamshostedapp.aspx%3Fteams%26personal%26componentId={your entityId}%26forceLocale={locale}",
            "websiteUrl": "https://products.office.com/en-us/sharepoint/collaboration",
            "scopes": [
                "personal"
            ]
        }
    ],
    "configurableTabs": [
        {
            "configurationUrl": "https://{teamSiteDomain}{teamSitePath}/_layouts/15/TeamsLogon.aspx?SPFX=true&dest={teamSitePath}/_layouts/15/teamshostedapp.aspx%3FopenPropertyPane=true%26teams%26componentId={your entityId}%26forceLocale={locale}",
            "canUpdateConfiguration": true,
            "scopes": [
                "team"
            ]
        }
    ],
    "permissions": [
        "identity",
        "messageTeamMembers"
    ],
    "validDomains": [
        "*.login.microsoftonline.com",
        "*.sharepoint.com",
        "*.sharepoint-df.com",
        "spoppe-a.akamaihd.net",
        "spoprod-a.akamaihd.net",
        "resourceseng.blob.core.windows.net",
        "msft.spoppe.com"
    ],
    "webApplicationInfo": {
        "resource": "https://{teamSiteDomain}",
        "id": "00000003-0000-0ff1-ce00-000000000000"
    }
}
```

## Restore your project configuration
If you still want to restore your project configuration after the upgrade is successful and continue to use Teams Toolkit V2, Here are the steps:
1. Copy the `.backup/.fx` folder to your project root path.
2. Copy the `.backup/appPackage` folder to your project root path.
3. Open VSCode settings and make sure the setting `fx-extension: (Obsolete)WorkOnTeamsToolkitV2ProjectConfigurationFiles` is enabled.
> Note: This setting temporarily work on Teams Toolkit V2 project configuration files. It is obsolete and will be removed soon.
4. Reload VSCode.

If your upgrade is failed, you just need to do steps **3** and **4**.
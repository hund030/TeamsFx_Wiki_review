> This section describes the built-in app setting support by Teams Toolkit. You're free to use other approaches to configure your app settings.

In this document, you will learn:

* [How to add app settings to bot / message extension hosted on Azure Web App](#add-app-settings-to-bot--messaging-extension-hosted-by-azure-web-app)
* [How to add app settings to api hosted on Azure Functions](#add-app-settings-to-api-hosted-by-azure-functions)
* [How to add app settings for local debugging](#add-app-setting-for-local-debugging)
* [How to find app settings predefined by Teams Toolkit](#app-settings-predefined-by-teams-toolkit)
* [Understand how Teams Toolkit handles app setting for you](#how-teams-toolkit-handles-app-setting-for-you)

# Add app setting to Azure services
If you use Azure to host your application, you can declare your app settings for your Azure services via ARM template. We suggest you go through this [document](https://docs.microsoft.com/en-us/microsoftteams/platform/toolkit/provision) to gain deeper understanding about the provision behavior of Teams Toolkit as well as how to customize the ARM template in the project.

## Add app settings to bot / message extension (hosted by Azure Web App)
You can follow these steps to add app settings to the Azure Web App that hosts your bot:
1. Open `{your_project_root}/.fx/configs/azure.parameters.{env}.json`
2. Add new properties to the value of provisionParameters, which will be available during provision. Here is an example:
    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "provisionParameters": {
                "value": {
                    ...
                    "valueOfMyAppSettingToBeAdded": "actual app setting value"
                }
            }
        }
    }
    ```
    > Note: You can reference environment variable when setting the value. Visit this [document](https://docs.microsoft.com/en-us/microsoftteams/platform/toolkit/provision#referencing-environment-variables-in-parameter-files) for more details.
3. Open `{your_project_root}/templates/azure/provision/bot.bicep`
4. Find the `webApp` resource declaration in the file, the resource looks like this: `resource webApp 'Microsoft.Web/sites@2021-02-01'`
5. Declare your app settings in the app settings property, and set the values your defined in parameter file to them. Here is an example:
    ```
    resource webApp 'Microsoft.Web/sites@2021-02-01' = {
        kind: 'app'
        location: resourceGroup().location
        name: webAppName
        properties: {
            siteConfig: {
                appSettings: [
                    ...
                    {
                        name: 'MY_APP_SETTING_NAME'
                        value: provisionParameters.valueOfMyAppSettingToBeAdded // the value comes from parameter file
                    }
                ]
            }
        }
    }
    ```
6. Run `Teams: Provision in the cloud` command to apply your changes to Azure

## Add app settings to api (hosted by Azure Functions)
You can follow these steps to add app settings to the Azure Web App that hosts your bot:
1. Open `{your_project_root}/.fx/configs/azure.parameters.{env}.json`
2. Add new properties to the value of provisionParameters, which will be available during provision. Here is an example:
    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "provisionParameters": {
                "value": {
                    ...
                    "valueOfMyAppSettingToBeAdded": "actual app setting value"
                }
            }
        }
    }
    ```
    > Note: You can reference environment variable when setting the value. Visit this [document](https://docs.microsoft.com/en-us/microsoftteams/platform/toolkit/provision#referencing-environment-variables-in-parameter-files) for more details.
3. Open `{your_project_root}/templates/azure/provision/function.bicep`
4. Find the `functionApp` resource declaration in the file, the resource looks like this: `resource functionApp 'Microsoft.Web/sites@2021-02-01'`
5. Declare your app settings in the app settings property, and set the values your defined in parameter file to them. Here is an example:
    ```
    resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
        kind: 'functionapp'
        location: resourceGroup().location
        name: functionAppName
        properties: {
            siteConfig: {
                appSettings: [
                    ...
                    {
                        name: 'MY_APP_SETTING_NAME'
                        value: provisionParameters.valueOfMyAppSettingToBeAdded // the value comes from parameter file
                    }
                ]
            }
        }
    }
    ```
6. Run `Teams: Provision in the cloud` command to apply your changes to Azure

# Add app setting for local debugging

Each project (tab / bot / api) has `.env.teamsfx.local` file in the root that defines environment variables available during local debugging. You can follow the syntax of [dotenv](https://www.npmjs.com/package/dotenv) package to add customized environment variables to it. If `.env.teamsfx.local` file does not exist, you can create an empty file and modify it.

For example, to add `CUSTOM_APP_SETTING` to your bot project, you can edit `bot/.env.teamsfx.local` file and add following content to it:
```
CUSTOM_APP_SETTING=my_app_setting_value
```

# App settings predefined by Teams Toolkit

Teams toolkit will define necessary app settings for your Teams app. You can refer following table to learn where to find the predefined app settings:
| Project | Predefined app settings for Azure (via [bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/)) | Predefined app settings for local debugging (via [dotenv](https://www.npmjs.com/package/dotenv)) |
| --- | --- | --- |
| tab | N/A | tabs/.env.teamsfx.local |
| bot | templates/azure/teamsfx/bot.bicep | bot/.env.teamsfx.local |
| api | templates/azure/teamsfx/function.bicep | api/.env.teamsfx.local |

> Note: the .env.teamsfx.local is generated during local debugging. You need to local debug first to see this file.

# How Teams Toolkit handles app setting for you

## How Teams Toolkit provide app settings to your app during local debug

When Teams Toolkit starts your app locally, it will read the `.env.teamsfx.local` file under tab/api/bot folder and add environment variables defined in the file to your app process. There is no need to use packages like dotenv to read the file content in your app.

## How Teams Toolkit provide app settings to your app when host in Azure

The Azure Web App and Azure Functions used to host your Teams app has [configuration](https://docs.microsoft.com/en-us/azure/app-service/configure-common?tabs=portal) support. It will make all the configurations available as environment variables in your app. Teams Toolkit uses ARM template to update the service configurations. You can refer above tutorials to add your own app settings via ARM template too.

## How Teams Toolkit define app setting values for different environment

Teams Toolkit gives you ability to manage multiple environments. You can learn more about the multiple environment feature at [here](https://docs.microsoft.com/en-us/microsoftteams/platform/toolkit/teamsfx-multi-env).

Here is an example about where to define different app setting values for different environment. Imaging you have 3 environments: `local`, `dev`, `prod`, you need to add your app setting values to following files for each environment:
| Environment | File to modify |
| --- | --- |
| local | .env.teamsfx.local under tab/api/bot folder |
| dev (hosted on Azure) | .fx/configs/azure.parameters.dev.json |
| prod (hosted on Azure) | .fx/configs/azure.parameters.prod.json |

> Note: the `azure.parameters.{env}.json` file is consumed by ARM template under `templates/azure` folder. You need to update the ARM template to reference the values in parameter file first before adding values to the parameter file. You can refer the tutorials at the beginning of this page to understand how to add an extra app settings.
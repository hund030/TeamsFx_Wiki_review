You may want to change the out-of-box provision behavior provided by the tooling. This section introduces what kind of customization you can achieve.
# Use an existing AAD app for your Teams app
You can add following configuration snippet to `.fx/configs/config.{your_env_name}.json` to use an AAD app created by yourself for your Teams app. You can follow https://aka.ms/teamsfx-existing-aad-doc to create an AAD app that can be used here.
```
"auth": {
    "clientId": "<your AAD app client id",
    "clientSecret": "{{$env.ENV_NAME_THAT_STORES_YOUR_SECRET}}",
    "objectId": "<your AAD app object id>",
    "accessAsUserScopeId": "<id of the access_as_user scope>"
}
```

After added above snippet, add your secret to related environment variable so the tooling can resolve the actual secret during provision.

> Note: You should not share one AAD app in multiple environments. If you do not have permission to update the AAD app, you will get a warning with instructions about how to manually update the AAD app. Please follow the instructions to update your AAD app after provision.

# Use an existing AAD app for your bot
You can add following configuration snippet to `.fx/configs/config.{your_env_name}.json` file to use an AAD app created by yourself for your bot.
```
"bot": {
    "appId": "<your AAD app client id>",
    "appPassword": "{{$env.ENV_NAME_THAT_STORES_YOUR_SECRET}}"
}
```

After added above snippet, add your secret to related environment variable so the tooling can resolve the actual secret during provision.

> Note: You should not share one AAD app in multiple environments.

# Skip adding user for SQL database
Sometimes you may get insufficient permission error when the tool tries to add user to SQL database. You can add following configuration snippet to `.fx/configs/config.{your_env_name}.json` file to skip adding SQL database user.
```
"skipAddingSqlUser": true
```

# Customize Azure resource group name
Before provision, the tool will ask you if you want to create a new resource group or use an existing one. You can provide a new resource group name or choose an existing one in this step.

# Customize Azure resource being created
The tooling uses ARM template to define Azure resources. The ARM template is a set of bicep files that defines the infrastructure and configuration for your project. You can customize Azure resources being created by modifying the ARM template. Visit [bicep document](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/) to learn more about how to use bicep to author ARM template.

You have two ways to customize Azure resources being created: customizing the parameter files and customizing the bicep files.

## Customize ARM template parameter files
The tooling provides a set of predefined parameters for you to customize the Azure resources. The parameter files are located at `.fx/configs/azure.parameters.{your_env_name}.json` and all the available parameters are defined in the `provisionParameters` property. It's preferred to customize the parameter files if the predefined parameters satisfies your requirement.

Here's a list of predefined parameters available:
| Parameter name | Default Value | What can be customized by the parameter | Value constraints |
| --- | --- | --- | --- |
| resourceBaseName | auto generated for each environment | Default name for all resources | 2-20 lowercase letters and numbers |
| simpleAuthServerFarmsName | ${resourceBaseName}simpleAuth | Name of Simple Auth App Service Plan | 1-40 alphanumerics and hyphens |
| simpleAuthWebAppName | ${resourceBaseName}simpleAuth | Name of Simple Auth Web App | 2-60 alphanumerics and hyphens <br /> Cannot start or end with hyphen |
| simpleAuthSku | F1 | SKU of Simple Auth App Service Plan | Please refer Web App's available SKU in your region in [pricing](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/#pricing) page |
| frontendHostingStorageName | ${resourceBaseName}tab | Name of Frontend Hosting Storage Account | 3-24 lowercase letters and numbers |
| functionServerfarmsName | ${resourceBaseName}api | Name of Function App's App Service Plan | 1-40 alphanumerics and hyphens |
| functionAppName | ${resourceBaseName}api | Name of Function App | 2-60 alphanumerics and hyphens <br /> Cannot start or end with hyphen |
| functionStorageName | ${resourceBaseName}api | Name of Function App's Storage Account | 3-24 lowercase letters and numbers |
| botServiceName | ${resourceBaseName} | Name of Azure Bot service | 2-64 alphanumerics, underscores, periods, and hyphens <br /> Start with alphanumeric |
| botDisplayName | ${resourceBaseName} | Display name of your bot | 1-42 characters |
| botServerfarmsName | ${resourceBaseName}bot | Name of Bot's App Service Plan | 1-40 alphanumerics and hyphens |
| botWebAppName | ${resourceBaseName}bot | Name of Bot's Web App | 2-60 alphanumerics and hyphens <br /> Cannot start or end with hyphen |
| botWebAppSKU | F1 | SKU of Bot App Service Plan | Please refer Web App's available SKU in your region in [pricing](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/#pricing) page |
| userAssignedIdentityName | ${resourceBaseName} | Name of user assigned identity | 3-128 alphanumerics, hyphens, and underscores <br /> Start with letter or number |
| azureSqlServerName | ${resourceBaseName} | Name of Azure SQL Server | 1-63 lowercase letters, numbers, and hyphens <br /> Cannot start or end with hyphen |
| azureSqlDatabaseName | ${resourceBaseName} | Name of Azure SQL Database | 1-128 characters, can't use <>*%&:\/? or control characters <br /> Can't end with period or space |
| apimServiceName | ${resourceBaseName} | Name of APIM Service | 1-50 alphanumerics and hyphens <br /> Start with letter and end with alphanumeric |
| apimProductName | ${resourceBaseName} | Name of APIM Product | 1-80 alphanumerics and hyphens <br /> Start with letter and end with alphanumeric |
| apimOauthServerName | ${resourceBaseName} | Name of APIM OAuth Server | 1-80 alphanumerics and hyphens <br /> Start with letter and end with alphanumeric |

In the meanwhile, following parameters are available with values populated during provision. If you want to change value of these parameters, you can delete the placeholder and fill your expected value to it directly. The purpose of these placeholders is to ensure we can create new resources for you when you created a new environment. The actual values are resolved from `.fx/states/state.{your_env_name}.json`.
| Parameter name | Default value place holder | Meaning of the place holder |
| --- | --- | --- |
| m365ClientId | {{state.fx-resource-aad-app-for-teams.clientId}} | Your app's AAD app client id created during provision |
| m365ClientSecret | {{state.fx-resource-aad-app-for-teams.clientSecret}} | Your app's AAD app client secret created during provision |
| m365TenantId | {{state.fx-resource-aad-app-for-teams.tenantId}} | Tenant id of your app's AAD app |
| m365OauthAuthorityHost | {{state.fx-resource-aad-app-for-teams.oauthHost}} | OAuth authority host of your app's AAD app |
| azureSqlAdmin | {{state.fx-resource-azure-sql.admin}} | Azure SQL Server admin account you provided during provision |
| azureSqlAdminPassword | {{state.fx-resource-azure-sql.adminPassword}} | Azure SQL Server admin password you provided during provision |
| botAadAppClientId | {{state.fx-resource-bot.botId}} | Bot's AAD app client id created during provision |
| botAadAppClientSecret | {{state.fx-resource-bot.botPassword}} | Bot's AAD app client secret created during provision |
| apimClientId | {{state.fx-resource-apim.apimClientAADClientId}} | APIM's AAD app client id created during provision |
| apimClientSecret | {{state.fx-resource-apim.apimClientAADClientSecret}} | APIM's AAD app client secret created during provision |
| apimPublisherEmail | {{state.fx-resource-apim.publisherEmail}} | APIM's publisher email, default value is your Azure account |
| apimPublisherName | {{state.fx-resource-apim.publisherName}} | APIM's publisher name, default value is your Azure account |

### Referencing environment variables in parameter files
Some times you may not want to hardcode the values in parameter files. For example, when the value is a secret. The parameter files support referencing the values from environment variables. You can use syntax `{{$env.YOUR_ENV_VARIABLE_NAME}}` in parameter values to tell the tooling that the value needs to be resolved from current environment variable.

Following example will read the value of `mySelfHostedDbConnectionString` parameter from environment variable `DB_CONNECTION_STRING`:
```
...
    "mySelfHostedDbConnectionString": "{{$env.DB_CONNECTION_STRING}}"
...
```

### Example: specifying the name of Function App instance
This this example, I will specify another name `contosoteamsappapi` for Function App instance instead of using the default name.

The steps are:
1. Open `.fx/configs/azure.parameters.{env}.json` for your current environment.
2. Add a new property `functionAppName` to the value of parameter `provisionParameters`.
3. Fill "contosoteamsappapi" as value of `functionAppName`
4. The final parameter file will looks like following
    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "provisionParameters": {
            "value": {
                "functionAppName": "contosoteamsappapi"
                ...
                }
            }
        }
    }
    ```

Note: If you already provisioned the environment, specifying the name will create a new Function App instance for you, instead of renaming the instance created previously.

## Customize ARM template
If the predefined parameters does not satisfy your requirement, you can customize the ARM template under `templates/azure` folder. For example, you can customize the ARM template to create some additional Azure resources for your app. This is an advance scenario and requires you have basic knowledge of bicep language which is used to author ARM template. You can get started with bicep at [bicep documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/).

> Note: the ARM template is shared by all environments. You can use [conditional deployment](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/conditional-resource-deployment) if the provision behavior various between environments.

### ARM template structure
The ARM template located at `templates/azure` contains following files

| File | What does it do | Can be customized |
| --- | --- | --- |
| main.bicep | Entry point for Azure resource provision | Yes |
| provision.bicep | Create and config Azure resources | Yes |
| config.bicep | Add TeamsFx required configurations to Azure resources | Yes |
| provision/xxx.bicep | Consumed by provision.bicep to create and config each Azure resources | Yes |
| teamsfx/xxx.bicep | Consumed by config.bicep to add TeamsFx required configurations to each Azure resources | No |

> Note: teamsfx/xxx.bicep will be regenerated when you add resources / capabilities to your project. That's why it's marked as not customizable. If you realy have requirement to modify these bicep files, go ahead. We suggest using Git to track your changes to teamsfx/xxx.bicep files so you won't lose your changes when adding resources / capabilities.

### Requirement to customized ARM template
To ensure the TeamsFx tooling functions properly, please ensure your customized ARM template satisfies following requirement. If you uses other tooling for further development, you can ignore these requirement.

1. Keep the folder structure and file name unchanged. The tool may append new content to existing files when you adds more resources / capabilities to your project.
2. Keep the name of auto generated parameters as well as its property names unchanged. The auto generated parameters may be used when you adds more resources / capabilities to your project.
3. Keep the output of auto generated ARM template unchanged. You can add additional outputs to ARM template. The output will be persisted to `.fx/states/state.{env}.json` and used in other features like deploy, validate manifest file, etc.

### Sample: Add other Azure resource (Azure Storage) to the application. 
Consider the scenario, you want to add Azure Storage to your Azure Function backend to store some blob data. There is no auto flow to update the bicep template with Azure Storage support. However, you can edit the bicep file and add the resource. The steps are as follows

1. Create a tab project
2. Add a function to the project. You can visit [Add Resources](./add-resource.md) to learn more about adding resources.
3. Declare the new Storage Account in ARM template. For simplify, we will declare the resource at `templates/azure/provision/function.bicep` directly. You're free to declare the resources in other places.

    `````````
    var applicationStorageAccountName = 'myapp${uniqueString(resourceGroup().id)}'
    resource applicationStorageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
        name: applicationStorageAccountName
        location: resourceGroup().location
        kind: 'Storage'
        sku: {
            name: 'Standard_LRS'
        }
    }
    `````````

4. Update the Azure Function App Settings with Azure Storage connection string in `templates/azure/provision/function.bicep`, which is the same file in step 3. Add following snippet to `functionApp` resource's `appSettings` array.

    ``````````````````    
    {
        name: 'MyAppStorageAccountConnectionString'
        value: 'DefaultEndpointsProtocol=https;AccountName=${applicationStorageAccount.name};AccountKey=${listKeys(applicationStorageAccount.id, '2021-06-01').key1}'
    }
    ```````````````````    

5. Now your can update your function with Azure Storage output bindings.
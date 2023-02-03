Teams Toolkit continuously evolves to offer more powerful features for developers. A group of new features has been made available in the latest VS Code Teams Toolkit pre-release, such as customizing Teams Toolkit's lifecycle definition. These new features will require an update to your existing project structure. Teams Toolkit can automatically upgrade your project.

> Please note that the new features will be enabled by default in the future release of Teams Toolkit. We thrive to make Teams Toolkit compatible with existing projects but long term backwards compatibility is not 100% guaranteed, thus we strongly recommend updating your project configurations to continue using the latest Teams Toolkit.

## How to Upgrade

Teams Toolkit automatically upgrades your project without changing your existing application code.

> We recommend using git to track changes during upgrade.

## File Structure Change

Teams Toolkit performs the following changes to your project during upgrade.

### File changes

1. Created `teamsapp.yml` and `teamsapp.local.yml` in your project's root folder
    > These file contains lifecycle configuration for your project
2. Moved environment files in `.fx` to `.env.{env}` in `env` folder
    > The environment files in old project contains `state.{env}.json`, `config.{env}.json` and `{env}.userdata`
3. Moved `templates/appPackage` to `appPackage` and updated placeholders in it per latest tooling's requirement.
    > This step also renamed `appPackage/manifest.template.json` to `appPackage/manifest.json`
    > The tooling now uses `${{ENV_VAR_NAME}}` to reference environment variables and all old placeholders have been renamed to environment variable
4. Moved `templates/appPackage/aad.template.json` to `aad.manifest.json` and updated placeholders in it per latest tooling's requirement.
    > The tooling now uses `${{ENV_VAR_NAME}}` to reference environment variables and all old placeholders have been renamed to environment variable
5. Updated `.vscode/tasks.json` and `.vscode/launch.json`.
6. Updated `.gitignore` to ignore new environment files.
7. Removed `.fx` folder
    > Content under this folder is no longer used.

> All existing files will be moved into `.backup` folder for your reference. You can safely delete the `.backup` folder after you reviewed the changes.

## Restore your project configuration

If you still want to restore your project configuration after the upgrade is successful and continue to use old version Teams Toolkit, you can:
1. Copy every folder / file under `.backup` folder to your project root path
    > You can safely overwrite any existing files if you didn't change the files after upgrade.
2. Remove the new folders / files mentioned in [File Changes](#file-changes) section:
    * teamsapp.yml and teamsapp.local.yml
    * aad.manifest.template.json
    * teamsAppEnv folder
    * appPackage folder

## Known issues

### STATE__FX_RESOURCE_FRONTEND_HOSTING__ENDPOINT missing error in some projects
If your project only contains a bot, an error might occur saying `STATE__FX_RESOURCE_FRONTEND_HOSTING__ENDPOINT` is missing when executing commands. Replace this placeholder with a valid URL in `appPackage/manifest.template.json` to fix it.
> This happens in projects created using very old Teams Toolkit. The old tooling provides a default example URL for your project if this placeholder does not exist. In latest tooling, we want to make everything more transparent so requires you to provide your URL here.

## Feature changes that impact your development flow

If you're using VS Code Teams Toolkit, there're some changes to existing features you should be aware of:

### Environment management

1. All the environment files will be gitignored by default. You need to sync the environment variables in `.env.{env_name}` files under `teamsfx` folder by yourselves to other machines to operate corresponding environments.
2. When creating new environments, you need to fill customized fields in the new `.env.{env_name}` file. Usually you need to provide values for all environment values with `CONFIG__` prefix.
3. When creating new environments, you need to manually create `templates/azure/azure.parameters.{env_name}.json` as Azure provision parameters and fill the parameter values accordingly.
4. Some additional steps are required if you added SQL or APIM to your project. Refer [Provision SQL databases](#provision-sql-databases) and [Provision APIM service](#provision-apim-service) section to learn more.

### Launch your app

1. When launching your app for a remote environment, Teams Toolkit will no longer ask you to select an environment. You need to manually change environment name for `${dev:teamsAppId}` in `.vscode/launch.json` to launch your app for a certain environment.

### Provision SQL databases

1. When you provision a new environment, you need to provide values for `STATE__FX_RESOURCE_AZURE_SQL__ADMIN` and `SECRET_FX_RESOURCE_AZURE_SQL__ADMINPASSWORD` in `.env.{env_name}` which are required inputs for creating SQL databases.
    > If you're provisioning an existing environment, you don't need this step.
2. You need to grant permission to user assigned identity manually after provisioning a new environment. This [document](https://aka.ms/teamsfx-add-azure-sql) includes tutorials about how to access SQL databases using user assigned identity.

### Provision APIM service

1. When you provision an environment, you need to provide values for `APIM__PUBLISHEREMAIL` and `APIM__PUBLISHERNAME` in `.env.{env_name}` which are required inputs for creating or updating APIM services.
2. You need to manually create an Azure Active Directory app for APIM service when provisioning a new environment. This [document](https://aka.ms/teamsfx-add-azure-apim) includes tutorials about how to create an Azure Active Directory app for APIM service.
3. Teams Toolkit no longer support deploy API spec to APIM any more.
Teams Toolkit will reuse your provisioned resource when upgrading (except Bot Service), when you wish to add a new environment after project upgrading, please remember to change resource name in `azure.parameters.{your_env_name}.json` to avoid name conflicts. 

## Upgrade your projects manually


This section helps you understand how to upgrade the old project template (created by Teams Toolkit 4.X) to latest structure. If you have customized the project after creation, you can make additional changes during the upgrade steps for your customized part. In general, the upgrade process contains 2 steps:

1. Create a new project using latest Teams Toolkit and copy your source code to it

2. Transform the placeholders used by old Teams Toolkit to new format

### Step 1: create a new project and copy your source code to it

#### Project with single Teams capability that hosted on Azure

If your project only contains 1 tab, bot or message extension, follow these steps:

1. Use Teams Toolkit 5.0.0 to create a new project with same capability

2. Copy your project's source code to the new project

    1. If your project is a tab app, copy everything under `tab` folder to the new project's root folder

    2. If your project is a bot app or message extension app, copy everything under `bot` folder to the new project's root folder

3. Copy your ARM template to the new project

    1. Remove `azure.bicep` and `azure.parameters.json` under the new project's `infra` folder. 

    2. Copy everything under `templates/azure` to the new project's `infra` folder. 

    3. Rename `main.bicep` to `azure.bicep`.

    4. Copy `.fx/configs/azure.parameters.{env}.json` to the new project's `infra` folder.

    5. Update the new project's `teamsapp.yml` file, change `parameters: ./infra/azure.parameters.json` to `parameters: ./infra/azure.parameters.${{TEAMSFX_ENV}}.json`

4. Copy your Teams manifest to the new project

    1. Remove everything under the new project's `appPackage` folder

    2. Copy everything under your existing project's `templates/appPackage` folder to `appPackage` in your new project

    3. Rename `appPackage/manifest.template.json` to `appPackage/manifest.json`.

5. If your existing project contains `templates/appPackage/aad.template.json`, copy the content of this file to `aad.manifest.json` (under root of the project folder) in your new project.

6. Refer [Step 2: transform the placeholders to new format](#step-2-transform-the-placeholders-to-new-format) to update placeholders in `appPackage/manifest.json`, `aad.manifest.json` (if have) and `infra/azure.parameters.{env}.json`.

#### Project with multiple Teams capability that hosted on Azure

If your project contains multiple capabilities, for example, has both tab and bot, follow these steps:

1. Download sample project based on your project's capabilities
    | Capabilities | Sample project location |
    | --- | --- |
    | tab, api | [sample project](https://github.com/OfficeDev/TeamsFx/tree/chyuan/add-manual-upgrad-sample-project/docs/vscode-extension/5.0-multi-capability-sample/tab-and-api) |
    | tab, bot / message extension  | [sample project]() |
    | tab, api, bot / message extension | [sample project]() |

2. Copy your project's source code to the new project
    1. If your project contains a tab, copy everything under `tab` folder to the new project's `tab` folder
    2. If your project contains api, copy everything under `api` folder to the new project's `api` folder
    3. If your project contains a bot or/and message extension, copy everything under `bot` folder to the new project's `bot` folder

3. Copy your ARM template to the new project

    1. Remove `azure.bicep` and `azure.parameters.json` under the new project's `infra` folder. 

    2. Copy everything under `templates/azure` to the new project's `infra` folder. 

    3. Rename `main.bicep` to `azure.bicep`.

    4. Copy `.fx/configs/azure.parameters.{env}.json` to the new project's `infra` folder.

    5. Update the new project's `teamsapp.yml` file, change `parameters: ./infra/azure.parameters.json` to `parameters: ./infra/azure.parameters.${{TEAMSFX_ENV}}.json`

4. Copy your Teams manifest to the new project

    1. Remove everything under the new project's `appPackage` folder

    2. Copy everything under your existing project's `templates/appPackage` folder to `appPackage` in your new project
    
    3. Rename `appPackage/manifest.template.json` to `appPackage/manifest.json`.

5. If your existing project contains `templates/appPackage/aad.template.json`, copy the content of this file to `aad.manifest.json` (under root of the project folder) in your new project.

6. Refer [Step 2: transform the placeholders to new format](#step-2-transform-the-placeholders-to-new-format) to update placeholders in `appPackage/manifest.json`, `aad.manifest.json` (if have) and `infra/azure.parameters.{env}.json`.

#### SPFx tab project

If you build your Teams app using SPFx, follow these steps:

1. Use Teams Toolkit 5.0.0 to create a new SPFx tab project

2. Copy everything under your project's `SPFx` folder to the new proejct's root folder

4. Copy your Teams manifest to the new project

    1. Copy your existing project's `templates/appPackage/resources` folder to `appPackage` in your new project

    2. Record the values of `contentUrl` and `configurationUrl` in the new project's `appPackage/manifest.json` file and `appPackage/manifest.local.json` file

    3. Copy content of your existing project's `template/appPackage/manifest.template.json` file to the new project's `appPackage/manifest.json` and `appPackage/manifest.local.json`

    4. Copy the value of `contentUrl` and `configurationUrl` recorded in step 2 back to `appPackage/manifest.json` and `appPackage/manifest.local.json`

5. Refer [Step 2: transform the placeholders to new format](#step-2-transform-the-placeholders-to-new-format) to update placeholders in `appPackage/manifest.json`, `appPackage/manifest.local.json`.


### Step 2: transform the placeholders to new format

The latest Teams Toolkit can resolve environment variables in your manifest / parameter files, which can be easily integrated with different engineering system. This requires you to update the placeholders in your existing manifest / parameter files. For every file, you need to:

1. Add new environment variables to `env/.env.{env}` files. Each environment variable represents an old placeholders in your project.
    > If `.fx/configs/config.{env}.json` or `.fx/states/state.{env}.json` contains value for the old placeholder, you also need to copy the value to corresponding environment variable in `env/.env.{env}` files.

2. Replace the old placeholders `{{xxx}}` and `{{{xxx}}}` in your manifest / parameter files with new format `${{ENV_NAME}}`. The `ENV_NAME` in the new placeholder should be same with what you added in above step.

The detailed steps to update the older placeholders are:

1. Replace `{{config.xxx}}` placeholders in your manifest / parameter files. Copy following sample snippet to `env/.env.{env}` and update your manifest / parameter files to reference the environment variables in the samples.
    ```
    COLOR_ICON=resources/color.png # use ${{COLOR_ICON}} to replace the old placeholder {{config.manifest.icons.color}}
    OUTLINE_ICON=resources/outline.png # use ${{OUTLINE_ICON}} to replace the old placeholder {{config.manifest.icons.outline}}
    DESCRIPTION_SHORT=Short description of myapp # use ${{DESCRIPTION_SHORT}} to replace the old placeholder {{config.manifest.appName.short}}
    DESCRIPTION_FULL=Full description of myapp # use ${{DESCRIPTION_FULL}} to replace the old placeholder {{config.manifest.appName.full}}
    APP_NAME_SHORT=myapp # use ${{APP_NAME_SHORT}} to replace the old placeholder {{config.manifest.description.short}}
    APP_NAME_FULL=Full name for myapp # use ${{APP_NAME_FULL}} to replace the old placeholder {{config.manifest.description.full}}
    ```
    > You need to change the values to the actual one for your project. The values can be found in `.fx/configs/config.{env}.json`. You can also change the environment variable names as you want.

    > Your project may not use all the placeholders in the sample, you can delete the unused one.
    
    > If your project uses additional `{{config.xxx}}` placeholders, you can refer the sample to add additional environment variables.

2. Replace `{{{state.fx-resource-aad-app-for-teams.applicationIdUris}}}` placeholder.
    1. If your project only enables SSO for tab, convert it to `api://{{state.fx-resource-frontend-hosting.domain}}/${{AAD_APP_CLIENT_ID}}`
    2. If your project only enables SSO for bot or message extension, convert it to `api://botid-${{BOT_ID}}`
    3. If your project enables SSO for both and bot / message extension, convert it to `api://{{state.fx-resource-frontend-hosting.domain}}/botid-${{BOT_ID}}`
        > The `{{state.fx-resource-frontend-hosting.domain}}` with be replaced again in later step, don't worry on having this placeholder temporary

3. Replace `{{state.fx-resource-aad-app-for-teams.frontendEndpoint}}` to `{{state.fx-resource-frontend-hosting.endpoint}}`
    > The new placeholder will be replaced again in later step, don't worry on having this placeholder temporary

3. Replace part of `{{state.xxx}}` placeholders generated by Teams Toolkit. Copy following sample snippet to `env/.env.{env}` and update your manifest / parameter files to reference the environment variables in the samples.
    ```
    TEAMS_APP_TENANT_ID=00000000-0000-0000-0000-000000000000 # use ${{TEAMS_APP_TENANT_ID}} to replace the old placeholder {{state.fx-resource-appstudio.tenantId}}
    TEAMS_APP_ID=00000000-0000-0000-0000-000000000000 # use ${{TEAMS_APP_ID}} to replace the old placeholder {{state.fx-resource-appstudio.teamsAppId}}
    AAD_APP_CLIENT_ID=00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_CLIENT_ID}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.clientId}} or {{state.aad-app.clientId}}
    SECRET_AAD_APP_CLIENT_SECRET=your-aad-app-secret # use ${{SECRET_AAD_APP_CLIENT_SECRET}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.clientSecret}} or {{state.aad-app.clientSecret}}
    AAD_APP_OBJECT_ID=00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_OBJECT_ID}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.objectId}} or {{state.aad-app.objectId}}
    AAD_APP_ACCESS_AS_USER_PERMISSION_ID=00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.oauth2PermissionScopeId}} or {{state.aad-app.oauth2PermissionScopeId}}
    AAD_APP_TENANT_ID=00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_TENANT_ID}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.tenantId}} or {{state.aad-app.tenantId}}
    AAD_APP_OAUTH_AUTHORITY_HOST=https://login.microsoftonline.com # use ${{AAD_APP_OAUTH_AUTHORITY_HOST}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.oauthHost}} or {{state.aad-app.oauthHost}}
    AAD_APP_OAUTH_AUTHORITY=https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_OAUTH_AUTHORITY}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.oauthAuthority}} or {{state.aad-app.oauthAuthority}}
    BOT_ID=00000000-0000-0000-0000-000000000000 # use ${{BOT_ID}} to replace the old placeholder {{state.fx-resource-bot.botId}} or {{state.teams-bot.botId}}
    SECRET_BOT_PASSWORD=your-aad-app-secret # use ${{SECRET_BOT_PASSWORD}} to replace the old placeholder {{state.fx-resource-bot.botPassword}} or {{state.teams-bot.botPassword}}
    ```
    > You need to change the values to the actual one for your project. The values can be found in `.fx/states/state.{env}.json`. If there is no `state.{env}.json` file or `state.{env}.json` does not contain a property for the placeholder, leave the environment variable value empty.

    > Value of `SECRET_AAD_APP_CLIENT_SECRET` and `SECRET_BOT_PASSWORD` can be found in `.fx/states/{env}.userdata`. You need to use Teams Toolkit 4.X to inspect the encrypted secret in this file.

    > You should not change the environment variable names in above sample.

    > Your project may not use all the placeholders in the sample, you can delete the unused one.

4. Replace remaining `{{state.xxx}}` placeholders, which is generated by ARM deployment.
    1. Visit [arm/deploy document](https://aka.ms/teamsfx-actions/arm-deploy) to understand how Teams Toolkit convert ARM deployment output to environment variables. For example, if your project has following output in `infra/provision.bicep`

        ```
        output azureStorageTabOutput object = {
            teamsFxPluginId: 'teams-tab'
            domain: azureStorageTabProvision.outputs.domain
            endpoint: azureStorageTabProvision.outputs.endpoint
            indexPath: azureStorageTabProvision.outputs.indexPath
            storageResourceId: azureStorageTabProvision.outputs.storageResourceId
        }
        ```

        Teams Toolkit will transform them to

        ```
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__TEAMSFXPLUGINID=
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__DOMAIN=
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT=
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__INDEXPATH=
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__STORAGERESOURCEID=
        ```

    2. The middle part in `{{state.xxx}}` represents the `teamsFxPluginId` in `infra/provision.bicep`. For example, `{{state.fx-resource-frontend-hosting.endpoint}}` comes from output with `teamsFxPluginId=fx-resource-frontend-hosting`. Follow this rule to find outputs in `infra/provision.bicep` for old placeholders and figure out the new environment variable names. Add the new environment variables to `env/.env.{env}` and replace them in your manifest / parameter files.

        If you don't find plugin ID in `infra/provision.bicep`, map the plugin ID from `{{state.xxx}}` to the new one based on following table
        | old plugin ID | new plugin ID |
        | --- | --- |
        | fx-resource-frontend-hosting | teams-tab |
        | fx-resource-function | teams-api |
        | fx-resource-bot | teams-bot |
        | fx-resource-aad-app-for-teams | aad-app |

        For example, the steps to figure out the environment variable name for placeholder `{{{state.fx-resource-frontend-hosting.endpoint}}}` are:
           1. The plugin id for this placeholder is `fx-resource-frontend-hosting`.
           2. Refer above table, find bicep output `azureStorageTabOutput` that contains plugin id `fx-resource-frontend-hosting` or `teams-tab`.
           3. The environment variable name should be `PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT`

    3. Find `file/updateEnv` in `teamsapp.local.yml`, add the new environment variables you figured out in this step, and provide values for them when you run your app locally. For example, add `PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT` if your project contains a tab:
        ```
        - uses: file/updateEnv
          with:
          envs:
            PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT: https://localhost:53000 # the value is just an example
            # other environment variables ...
        ```
    
    4. Find `Start local tunnel` in `.vscode/tasks.json`, update the output to output `endpoint` and `domain` to the new environment variables for bot. For example:
        ``` json
        {
            "label": "Start local tunnel",
            "type": "teamsfx",
            "command": "debug-start-local-tunnel",
            "args": {
                "ngrokArgs": "http 3978 --log=stdout --log-format=logfmt",
                "env": "local",
                "output": {
                    "endpoint": "PROVISIONOUTPUT__AZUREWEBAPPBOTOUTPUT__SITEENDPOINT", // you need to update the environment variable name here
                    "domain": "PROVISIONOUTPUT__AZUREWEBAPPBOTOUTPUT__DOMAIN" // you need to update the environment variable name here
                }
            },
            "isBackground": true,
            "problemMatcher": "$teamsfx-local-tunnel-watch"
        }
        ```

    5. Open `teamsapp.yml`, for any placeholder not added by you, replace them based on their meaning. For example, `azureStorage/deploy` requires an Azure Storage resource id, so you need to change the placeholder to `${{PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__STORAGERESOURCEID}}`:
        ``` yml
        - uses: azureStorage/deploy
          with:
            distributionPath: ./build
            resourceId: ${{TAB_AZURE_STORAGE_RESOURCE_ID}} # you need to change ${{TAB_AZURE_STORAGE_RESOURCE_ID}} to ${{PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__STORAGERESOURCEID}}
        ```

## Migration
### MigrationAppPackageNotExist
The project to be migrated has no appPackage folder under templates which is required for migration. 
### ManifestTemplateNotExist
The project to be migrated has no manifest.template.json file under templates/appPackage which is required for migration. 
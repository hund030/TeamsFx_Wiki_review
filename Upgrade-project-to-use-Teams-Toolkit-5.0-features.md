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
    > The tooling now uses `${{ENV_VAR_NAME}}` to reference environment variables and all old placeholders have been renamed to environment variable
4. Moved `templates/appPackage/aad.template.json` to `aad.manifest.template.json` and updated placeholders in it per latest tooling's requirement.
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

If any error occurs during upgrade, you can follow these steps to initialize your project again to develop it with latest Teams Toolkit. Visit [Teams Toolkit 5.0 guide](https://aka.ms/teamsfx-v5.0-guide) to learn more about the new concepts in Teams Toolkit 5.0. And refer [Teams Toolkit initialize document](https://aka.ms/teamsfx-init) to learn how to initialize debug or remote environment configurations.

1. Run `npm install -g @microsoft/teamsfx-cli@alpha` to install latest TeamsFx CLI
> This step requires nodejs installed in your development machine.

2. Set environment variable `TEAMSFX_V3` to `true`

3. Create a backup folder (for example `.backup`) and move `.fx` folder to it.

4. Run `teamsfx init infra` and `teamsfx init debug` under your project root folder

5. Update the placeholders in `templates/appPackage/manifest.template.json`, `templates/aad.template.json` (if have) and `.fx/configs/azure.parameters.{env}.json` (if have). The old placeholders `{{xxx}}` and `{{{xxx}}}` in these files need to be replaced with new format `${{ENV_NAME}}`.
    1. For `{{config.xxx}}` placeholders, you can give them a preferred environment variable name using the new format. And put their values to `env/.env.{env}` files. The values can be found in `.fx/configs/config.{env}.json`.
    2. For `{{state.xxx}}` placeholders, if they exist in below mapping table, you can replace them directly. And put their values to `env/.env.{env}` files if you have provisioned the environment. The values can be found in `.fx/states/state.{env}.json`.

        | Old placeholder | New placeholder |
        | --- | --- |
        | state.fx-resource-appstudio.tenantId | TEAMS_APP_TENANT_ID |
        | state.fx-resource-appstudio.teamsAppId | TEAMS_APP_ID |
        | state.fx-resource-aad-app-for-teams.clientId | AAD_APP_CLIENT_ID |
        | state.fx-resource-aad-app-for-teams.clientSecret | SECRET_AAD_APP_CLIENT_SECRET |
        | state.fx-resource-aad-app-for-teams.objectId | AAD_APP_OBJECT_ID |
        | state.fx-resource-aad-app-for-teams.oauth2PermissionScopeId | AAD_APP_ACCESS_AS_USER_PERMISSION_ID |
        | state.fx-resource-aad-app-for-teams.tenantId | AAD_APP_TENANT_ID |
        | state.fx-resource-aad-app-for-teams.oauthHost | AAD_APP_OAUTH_AUTHORITY_HOST |
        | state.fx-resource-aad-app-for-teams.oauthAuthority | AAD_APP_OAUTH_AUTHORITY |
        | state.fx-resource-bot.botId | BOT_ID |
        | state.fx-resource-bot.botPassword | SECRET_BOT_PASSWORD |
        | state.aad-app.clientId | AAD_APP_CLIENT_ID |
        | state.aad-app.clientSecret | SECRET_AAD_APP_CLIENT_SECRET |
        | state.aad-app.objectId | AAD_APP_OBJECT_ID |
        | state.aad-app.oauth2PermissionScopeId | AAD_APP_ACCESS_AS_USER_PERMISSION_ID |
        | state.aad-app.tenantId | AAD_APP_TENANT_ID |
        | state.aad-app.oauthHost | AAD_APP_OAUTH_AUTHORITY_HOST |
        | state.aad-app.oauthAuthority | AAD_APP_OAUTH_AUTHORITY |
        | state.teams-bot.botId | BOT_ID |
        | state.teams-bot.botPassword | SECRET_BOT_PASSWORD |

    3. For `{{state.xxx}}` placeholders not exist in above mapping table, they come from ARM deployment output. You can update `templates/azure/main.bicep` to output additional outputs with your preferred environment variable name and use it in the new placeholder. You can visit https://aka.ms/teamsfx-actions/arm-deploy to learn how Teams Toolkit convert ARM deployment output to environment variables.
    
        For example, the `{{{state.fx-resource-frontend-hosting.endpoint}}}` placeholder means the endpoint of your tab app, so you can add following output to `main.bicep`:
        ```
        output TAB_ENDPOINT string = provision.outputs.azureStorageTabOutput.endpoint
        ```
        Now you can use `${{TAB_ENDPOINT}}` as the new placeholder in these files. Remember put their values to `env/.env.{env}` files if you have provisioned the environment. The values can be found in `.fx/states/state.{env}.json`.

    4. For `{{{state.fx-resource-aad-app-for-teams.applicationIdUris}}}`, covert it based on following rule:
        1. If your project only enables SSO for tab, convert it to `api://{{state.fx-resource-frontend-hosting.domain}}/${{AAD_APP_CLIENT_ID}}`
            > You need to replace `{{state.fx-resource-frontend-hosting.domain}}` with actual placeholder that represents your tab's endpoint
        2. If your project only enables SSO for bot or message extension, convert it to `api://botid-${{BOT_ID}}`
        3. If your project enables SSO for both and bot / message extension, convert it to `api://{{state.fx-resource-frontend-hosting.domain}}/botid-${{BOT_ID}}`
            > You need to replace `{{state.fx-resource-frontend-hosting.domain}}` with actual placeholder that represents your tab's endpoint


6. We recommend you to move following files to new positions so Teams Toolkit can find them automatically when executing certain commands:
    1. Move `templates/appPackage/aad.template.json` to `aad.manifest.json` if your project contains this file
    2. Move `templates/appPackage/*` to `appPackage/*` and rename `manifest.template.json` under `appPackage` folder to `manifest.json`
    > You need to update the paths in `teamsapp.yml` and `teamsapp.local.yml` in step 7 and 8 if you changes the file location and name.

7. Follow the instructions in https://aka.ms/teamsfx-infra to update your remote environment configurations. You can visit https://aka.ms/teamsfx-actions to learn the syntax of each action when authoring teamsapp.yml.
   <details>
      <summary>If you're building app hosted on Azure, you can refer following sample when authoring `teamsapp.yml`</summary>

      ``` yaml
    version: 1.0.0

    projectId: <your-project-id> # Put your project id here. You can find the value from .fx/configs/projectSettings.json

    environmentFolderPath: ./env # You can use any folder you want

    provision:
      - uses: botAadApp/create # You can remove this if your project does not contain a bot or message extension.
        with:
          name: <your-preferred-bot-aad-app-name>

      - uses: arm/deploy
        with:
          subscriptionId: ${{AZURE_SUBSCRIPTION_ID}}
          resourceGroupName: ${{AZURE_RESOURCE_GROUP_NAME}}
          templates:
          - path: ./templates/azure/main.bicep
            parameters: ./templates/azure/azure.parameters.${{TEAMSFX_ENV}}.json # We recommend you move `.fx/configs/azure.parameters.xxx.json` to `templates/azure` folder (or other places as you wish), because you do not need `.fx` folder any more.
            deploymentName: teams_toolkit_deployment
          bicepCliVersion: v0.4.613 
        
      - uses: azureStorage/enableStaticWebsite # You can remove this if your project does not contain a tab.
        with:
          storageResourceId: <azure-storage-resource-id> # Reference the Azure Storage resource id outputted in arm/deploy action. You can output the resource id in `main.bicep` with your preferred environment variable name. For example: use `output TAB_STORAGE_RESOURCE_ID string = provision.outputs.azureStorageTabOutput.storageResourceId` to output the Azure Storage resource id to `TAB_STORAGE_RESOURCE_ID` in .env.{env} file.
          indexPage: index.html
          errorPage: error.html

    deploy:
    # Deploy your tab. You can remove following 3 actions if your project does not contain a tab.
      - uses: cli/runNpmCommand 
        with:
          workingDirectory: tabs
          args: install
      - uses: cli/runNpmCommand
        env:
          REACT_APP_CLIENT_ID: ${{AAD_APP_CLIENT_ID}}
          REACT_APP_START_LOGIN_PAGE_URL: <your-tab-endpoint>/auth-start.html # Reference the tab endpoint outputted in arm/deploy action. You can output the resource id in `main.bicep` with your preferred environment variable name. For example: use `output TAB_ENDPOINT string = provision.outputs.azureStorageTabOutput.endpoint` to output the Azure Storage static website endpoint to `TAB_ENDPOINT` in .env.{env} file.
        with:
          workingDirectory: tabs
          args: run build --if-present
      - uses: azureStorage/deploy
        with:
          workingDirectory: tabs
          distributionPath: build
          ignoreFile:  # Leave blank will ignore nothing
          resourceId: <azure-storage-resource-id> # Reference the Azure Storage resource id outputted in arm/deploy action.

    # Deploy your bot. You can remove following 3 actions if your project does not contain a bot or message extension.
    # Note: if you bot is hosted in Azure Functions, you need to adjust your actions to build and deploy to Azure Functions.
      - uses: cli/runNpmCommand
        with:
          workingDirectory: bot
          args: install
      - uses: cli/runNpmCommand
        with:
          workingDirectory: bot
          args: run build --if-present
      - uses: azureAppService/deploy
        with:
          workingDirectory: bot
          distributionPath: .
          ignoreFile: # Leave blank will ignore nothing
          resourceId: <app-service-resource-id> # Reference the Azure App Service resource id outputted in arm/deploy action.

    registerApp:
      - uses: aadApp/create # You can remove this if your project does not require an AAD app.
        with:
          name: <your-preferred-aad-app-name> # The name will be modified based on AAD app manifest in aadApp/update action. It's recommended to use same name defined in the manifest file.
          generateClientSecret: true 

      - uses: teamsApp/create
        with:
          name: <your-preferred-teams-app-name> # The name will be modified based on Teams app manifest in teamsApp/update action. It's recommended to use same name defined in the manifest file.

    configureApp:
      - uses: aadApp/update # You can remove this if your project does not require an AAD app.
        with:
          manifestPath: ./templates/aad.template.json 
          outputFilePath : ./build/aad.manifest.${{TEAMSFX_ENV}}.json

      - uses: teamsApp/validate
        with:
          manifestPath: ./templates/appPackage/manifest.template.json
      - uses: teamsApp/zipAppPackage
        with:
          manifestPath: ./templates/appPackage/manifest.template.json
          outputZipPath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
          outputJsonPath: ./build/appPackage/manifest.${{TEAMSFX_ENV}}.json
      - uses: teamsApp/update
        with:
          appPackagePath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip

    publish:
      - uses: teamsApp/validate
        with:
          manifestPath: ./templates/appPackage/manifest.template.json
      - uses: teamsApp/zipAppPackage
        with:
          manifestPath: ./templates/appPackage/manifest.template.json
          outputZipPath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
          outputJsonPath: ./build/appPackage/manifest.${{TEAMSFX_ENV}}.json
      - uses: teamsApp/publishAppPackage
        with:
          appPackagePath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
      ```
   </details>

   <details>
      <summary>If you're building app using SPFx, you can refer following sample when authoring `teamsapp.yml`</summary>

      ``` yaml
    version: 1.0.0

    projectId: <your-project-id> # Put your project id here. You can find the value from .fx/configs/projectSettings.json

    environmentFolderPath: ./env # You can use any folder you want

    deploy:
      - uses: cli/runNpmCommand
        with:
          args: install
          workingDirectory: ./SPFx
      - uses: cli/runNpxCommand
        with:
          workingDirectory: ./SPFx
          args: gulp bundle --ship --no-color
      - uses: cli/runNpxCommand
        with:
          workingDirectory: ./SPFx
          args: gulp package-solution --ship --no-color
      - uses: spfx/deploy
        with:
          createAppCatalogIfNotExist: false
          packageSolutionPath: ./SPFx/config/package-solution.json

    registerApp:
      - uses: teamsApp/create
        with:
          name: <your-preferred-teams-app-name> # Put your preferred app name here

    configureApp:
      - uses: teamsApp/validate
        with:
          manifestPath: ./templates/appPackage/manifest.json
      - uses: teamsApp/zipAppPackage
        with:
          manifestPath: ./templates/appPackage/manifest.json
          outputZipPath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
          outputJsonPath: ./build/appPackage/manifest.${{TEAMSFX_ENV}}.json
      - uses: teamsApp/update
        with:
          appPackagePath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip

    publish:
      - uses: teamsApp/validate
        with:
          manifestPath: ./templates/appPackage/manifest.json
      - uses: teamsApp/zipAppPackage
        with:
          manifestPath: ./templates/appPackage/manifest.json
          outputZipPath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
          outputJsonPath: ./build/manifest.${{TEAMSFX_ENV}}.json
      - uses: teamsApp/copyAppPackageToSPFx
        with:
          appPackagePath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
          spfxFolder: ./src
      - uses: teamsApp/publishAppPackage
        with:
          appPackagePath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
      ```
   </details>

8. Follow the instructions in https://aka.ms/teamsfx-debug to update your debug configurations
   <details>
      <summary>If you're building app hosted on Azure, you can refer following sample when authoring `teamsapp.local.yml`</summary>

      ```yaml
      version: 1.0.0

      registerApp:
        - uses: aadApp/create # You can remove this if your project does not enable SSO.
          with:
            name: <your-preferred-aad-app-name> # Put your preferred app name here
            generateClientSecret: true

        - uses: teamsApp/create
          with:
            name: <your-preferred-teams-app-name> # Put your preferred app name here

      provision:
        - uses: botAadApp/create # You can remove this if your app does not contains a bot or message extension
          with:
            name: <your-preferred-bot-aad-app-name> # Put your preferred app name here

        - uses: botFramework/create # You can remove this if your app does not contains a bot or message extension
          with:
            botId: ${{BOT_ID}}
            name: <your-preferred-bot-name> # Put your preferred bot name here
            messagingEndpoint: <your-ngrok-endpoint>/api/messages # Put your ngrok endpoint here, can reference environment variables outputted by Teams Toolkit tasks in `.vscode/tasks.json` here
            description: ""
            channels:
              - name: msteams

      configureApp:
        - uses: file/updateEnv
          with:
            envs: # Add necessary environment variables for your app here
              PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__DOMAIN: localhost:53000
              PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT: https://localhost:53000
              PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__INDEXPATH: /index.html#

        - uses: aadApp/update
          with:
            manifestPath: ./templates/aad.template.json
            outputFilePath : ./build/aad.manifest.${{TEAMSFX_ENV}}.json

        - uses: teamsApp/validate
          with:
            manifestPath: ./templates/appPackage/manifest.template.json

        - uses: teamsApp/zipAppPackage
          with:
            manifestPath: ./templates/appPackage/manifest.template.json
            outputZipPath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
            outputJsonPath: ./build/appPackage/manifest.${{TEAMSFX_ENV}}.json

        - uses: teamsApp/update
          with:
            appPackagePath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip

      deploy:
        - uses: prerequisite/install
          with:
            devCert:
              trust: true

        - uses: file/updateEnv # Add necessary environment variables for your tab app, you can remove this if your project does not contain a tab
          with:
            target: ./tabs/.env.teamsfx.local
            envs:
              BROWSER: none
              HTTPS: true
              PORT: 53000
              SSL_CRT_FILE: ${{SSL_CRT_FILE}}
              SSL_KEY_FILE: ${{SSL_KEY_FILE}}

        - uses: file/updateEnv # Add necessary environment variables for your bot app, you can remove this if your project does not contain a bot or message extension
          with:
            target: ./bot/.env.teamsfx.local
            envs:
              BOT_ID: ${{BOT_ID}}
              BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}

        - uses: file/updateEnv # Add necessary environment variable for your tab app, you can remove this if your project does not contain a tab
          with:
            target: ./tabs/.env.teamsfx.local
            envs:
              REACT_APP_START_LOGIN_PAGE_URL: ${{PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT}}/auth-start.html
              REACT_APP_CLIENT_ID: ${{AAD_APP_CLIENT_ID}}

        - uses: file/updateEnv # Add necessary environment variable for your bot app, you can remove this if your project does not contain a bot or message extension
          with:
            target: ./bot/.env.teamsfx.local
            envs:
              M365_CLIENT_ID: ${{AAD_APP_CLIENT_ID}}
              M365_CLIENT_SECRET: ${{SECRET_AAD_APP_CLIENT_SECRET}}
              M365_TENANT_ID: ${{AAD_APP_TENANT_ID}}
              M365_AUTHORITY_HOST: ${{AAD_APP_OAUTH_AUTHORITY_HOST}}
              INITIATE_LOGIN_ENDPOINT: ${{PROVISIONOUTPUT__AZUREWEBAPPBOTOUTPUT__SITEENDPOINT}}/auth-start.html
              M365_APPLICATION_ID_URI: api://${{PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT}}/botid-${{BOT_ID}}

        - uses: cli/runNpmCommand # You can remove this if your project does not contain a tab
          with:
            args: install --no-audit
            workingDirectory: ./tabs

        - uses: cli/runNpmCommand # You can remove this if your project does not contain a bot or message extension
          with:
            args: install --no-audit
            workingDirectory: ./bot
      ```
   </details>

   <details>
      <summary>If you're building app using SPFx, you can refer following sample when authoring `teamsapp.local.yml`</summary>

      ```yaml
      version: 1.0.0

      registerApp:
        - uses: teamsApp/create
          with:
            name: <your-preferred-teams-app-name> # Put your preferred app name here

      configureApp:
        - uses: teamsApp/validate
          with:
            manifestPath: ./templates/appPackage/manifest.template.json

        - uses: teamsApp/zipAppPackage
          with:
            manifestPath: ./templates/appPackage/manifest.template.json
            outputZipPath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip
            outputJsonPath: ./build/appPackage/manifest.${{TEAMSFX_ENV}}.json

        - uses: teamsApp/update
          with:
            appPackagePath: ./build/appPackage/appPackage.${{TEAMSFX_ENV}}.zip

      deploy:
        - uses: cli/runNpmCommand
          with:
            args: install --no-audit
            workingDirectory: ./SPFx
      ```
   </details>


## Migration
### MigrationAppPackageNotExist
The project to be migrated has no appPackage folder under templates which is required for migration. 
### ManifestTemplateNotExist
The project to be migrated has no manifest.template.json file under templates/appPackage which is required for migration. 
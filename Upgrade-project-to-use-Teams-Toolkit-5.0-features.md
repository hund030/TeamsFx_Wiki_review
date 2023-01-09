Teams Toolkit continuously evolves to offer more powerful features for developers. A group of new features has been available in latest VS Code Teams Toolkit preview release, such as customizing Teams Toolkit's lifecycle definition by yourselves. These new features will require an update to your existing project code structures. Teams Toolkit can automatically upgrade your original project when you opens an old project.

> Please note that the new features will be enabled by default in the future release of Teams Toolkit. We thrive to make Teams Toolkit compatible with existing projects but long time backward compatibility is not 100% guaranteed thus we strongly recommend you update your project configurations to continue using the latest Teams Toolkit.

## How to Upgrade

Teams Toolkit will automatically upgrade your original project folder structure and not change your custom code.

> We recommend you to leverage git to track the changes during upgrade it possible.

## File Structure Change

Once upgrade succeeds, your project file structure will be changed.

### File changes

1. Moved everything in `.fx` to `teamsfx` folder with new file format.
    * Created new `app.yml` and `app.local.yml` files in `teamsfx` folder
    * Moved contents in `state.{env}.json`, `config.{env}.json` and `{env}.userdata` to `.env.{env}` in `teamsfx` folder
2. Moved `templates/appPackage` to `appPackage` and updated placeholders in it per latest tooling's requirement.
3. Moved `templates/appPackage/aad.template.json` to `aad.manifest.template.json` and updated placeholders in it per latest tooling's requirement.
4. Moved `.fx/configs/azure.parameter.{env}.json` to `templates/azure` and updated placeholders in it per latest tooling's requirement.
5. Updated `.vscode/tasks.json` and `.vscode/launch.json`.
6. Updated `.gitignore` to ignore new environment files in `teamsfx` folder.

> All existing files will be moved into `teamsfx/.backup` folder for your reference. You can safely delete the `.backup` folder after you have compared and reviewed the changes.

## Changes to existing features in VS Code Teams Toolkit

If you're using VS Code Teams Toolkit, there're some changes to existing features you should be aware of:

### Environment management

1. All the environment files will be gitignored by default. You need to sync the environment variables in `.env.{env_name}` files under `teamsfx` folder by yourselves to other machines to operate corresponding environments.
2. When creating new environments, you need to fill customized fields in the new `.env.{env_name}` file. Usually you need to provide values for all environment values with `CONFIG__` prefix.
3. When creating new environments, you need to manually create `templates/azure/azure.parameters.{env_name}.json` as Azure provision parameters and fill the parameter values accordingly.

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

## Restore your project configuration

If you still want to restore your project configuration after the upgrade is successful and continue to use old version Teams Toolkit, you can copy every folder / file to your project root path and remove the new folders / files mentioned in [File Changes](#file-changes) section.

## Upgrade your projects manually

If you meet errors, you can follow these steps to initialize your project again to develop it with latest Teams Toolkit:

1. Run `npm install -g @microsoft/teamsfx-cli@alpha` to install latest TeamsFx CLI
> This step requires nodejs installed in your development machine.

2. Set environment variable `TEAMSFX_V3` to `true`

3. Run `teamsfx init debug` under your project root and follow the instructions to update your debug configurations

4. Run `teamsfx init infra` under your project root and follow the instructions to update your remote environment configurations

5. Update the placeholders in your Teams app manifest file, AAD app manifest file (if have) and azure.parameters.{env}.json (if have). The old placeholder `{{xxx}}` needs to be replaced with new format `${{xxx}}`. You need to change the name of placeholder to your preferred environment variable name and update `app.yml` and `app.local.yml` to generate these environment variables for these files.
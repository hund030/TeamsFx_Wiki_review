# Configure Outlook Add-in capability within your Teams app

## Introduction

Office Add-ins are web apps that extend the functionality of Outlook. Some of the things that can be done with an Outlook Add-in:

- Read and write the content of email messages and meeting invitations, as well as responses, cancellations, and appointments.
- Read properties of the user's mailbox.
- Respond automatically to events, such as the sending of an email.
- Integrate with external services including CRM and project management.
- Add custom ribbon buttons or menu items to perform specific tasks.

For more information, start with [Outlook Add-ins Overview](https://learn.microsoft.com/en-us/office/dev/add-ins/outlook/outlook-add-ins-overview).

## Prerequisites

To configure an Office Add-in as additional capability, please make sure you have a Microsoft 365 account to test the application.

## Overview

The following are the major steps to adding an Outlook Add-in to a Teams app. Details are in the sections below.

1. [Prepare the Teams app project](prepare-the-teams-app-project)
1. [Create an Office Add-in project](#create-an-outlook-add-in-project) that is initially separate from your Teams app.
1. [Merge the manifest](#merge-the-manifest) from the Outlook Add-in project into the Teams app manifest.
1. [Copy the Outlook Add-in files to the Teams app project](#copy-the-outlook-add-in-files-to-the-teams-app-project).
1. [Edit the tooling configuration files](#edit-the-tooling-configuration-files).
1. [Setup local debug environment](#setup-local-debug-environment).
1. [Move the application to Azure](#move-the-application-to-azure).

## Prepare the Teams app project

Begin by segregating the source code for the tab (or bot) into its own subfolder. These instructions assume that the project initially has the following structure: 

    ```
    |-- .vscode/
    |-- appPackage/
    |-- build\appPackage/
    |-- infra/
    |-- node_modules/
    |-- public/
    |-- src/
    |-- teamsfx/
    |-- gitignore
    |-- package-lock.json
    |-- package.json
    |-- tsconfig.json
    ```

1. Create a folder under the root named "tab" (or "bot").

    **NOTE**: For simplicity, the remainder of this article assumes that the existing Teams app is a tab. If you started with a bot instead, replace "tab" with "bot" in all of these instructions, including the content you add or edit in various files. 

1. Move the node_modules, public, and src folders into the new subfolder.
1. Move the package-lock.json, package.json, and tsconfig.json into the new subfolder. The project structure should now look like the following:

    ```
    |-- .vscode/
    |-- appPackage/
    |-- build\appPackage/
    |-- infra/
    |-- tab/
    |-- |-- public/
    |-- |-- src/
    |-- |-- package-lock.json
    |-- |-- package.json
    |-- |-- tsconfig.json
    |-- teamsfx/
    |-- gitignore
    ```

1. Create a new file named package.json in the root of the project and give it the following content:

    ```
    {
        "name": "CombinedTabAndAddin",
        "version": "0.0.1",
        "author": "Contoso",
        "scripts": {
            "test": "echo \"Error: no test specified\" && exit 1",
            "install": "cd tab && npm install",
            "install:tab": "cd tab && npm install",
            "start:tab": "cd tab && npm run start",
            "build:tab": "cd tab && npm run build",
        },
        "devDependencies": {
            "@microsoft/teamsfx-run-utils": "alpha"
        },
    }
    ```
1. Change the "name", "version", and "author" properties, as needed.
1. Open the package.json in the tabs subfolder and find the script for "dev:teamsfx". Add `cd .. &&` to the front of the value of the script. It should look like the following when you are done:

    ```
    "dev:teamsfx": "cd .. && node teamsfx/script/run.js . teamsfx/.env.local",
    ```

1. In the .vscode folder, open the tasks.json file and find the `Start frontend` task.
1. In the "options.cwd" property, add `/tab` to the end of the value. When you are done, it should look like the following:

   ```
    "options": {
        "cwd": "${workspaceFolder}/tab"
    },
   ```

1. Open the file teamsfx\script\run.js and find the line near the end that spawns a separate process. 
1. Change the string "start" in that line to "start:tab". When you are done the line should look like the following:

    ```
    cp.spawn(/^win/.test(process.platform) ? "npm.cmd" : "npm", ["run", "start:tab"], {
      stdio: "inherit",
    });
    ```

## Create an Outlook Add-in project

1. With Teams Toolkit open in Visual Studio Code, select **Create a new app** under **DEVELOPMENT**.
1. In the **Select an option** drop down, select **Create a new Office add-in**, and then select **Outlook Taskpane Add-in (preview)**.
1. Give a name to the project when prompted and Teams Toolkit will create the project with basic files and scaffolding.

## Merge the manifest

The Teams app's manifest is generated at debug-and-sideload time (or build time) from the template file, manifest.template.json in the \appPackage folder of the Teams project. Most of the markup is hardcoded into the template, but there are also some configuration files that contain data that gets injected into the final generated manifest. In the following steps, you do the following:

- Copy markup from the add-in's manifest to the Teams app's manifest template.
- Edit the configuration files. 

Unless specified otherwise, the file you change is \appPackage\manifest.template.json.

1. Copy the "$schema" and "manifestVersion" property values from the add-in's manifest to the corresponding properties of the Teams app's manifest.template.json file.
1. Adjust the "name.full", "description.short", and "description.full" property values as needed to take account of the fact that an Outlook add-in is part of the app. 
1. Leave the "name.short" with the placeholder value, `${{TEAMS_APP_NAME}}`,that it already has. To adjust this value, open the files .env.dev and .env.local in the \teamsfx folder. In both files, change the value of the `TEAMS_APP_NAME` variable. (If the value has spaces, enclose it in quotes.) 

    **Note**:
    It is a good practice to append "-local" onto the end of the name in the .env.local file.
    The "name.short" value appears in both the Teams tab app and the Outlook add-in. Examples: 

    - It is the label under the launch button of the tab app.
    - It is content of the title bar of the add-in's task pane.

    The following is an example:

    ```
    TEAMS_APP_NAME="FABRIKAM Tab and Add-in-local"
    ```

1. If there is no "authorization.permissions.resourceSpecific" array in the Teams manifest template, copy it from the add-in manifest as a top-level property. If there already is one in the Teams template, copy any objects from the array in the add-in manifest to the array in the Teams template. The following is an example:

    ```json
    "authorization": {
        "permissions": {
            "resourceSpecific": [
                {
                    "name": "MailboxItem.Read.User",
                    "type": "Delegated"
                }
            ]
        }
    },
    ```

1. In the .env.local file, find the lines that assign values to the `TAB_DOMAIN` and `TAB_ENDPOINT` variables. Add the following lines immediately below them:

    ```
    ADDIN_DOMAIN=localhost:53000
    ADDIN_ENDPOINT=https://localhost:53000
    ```

    **NOTE**: During the early preview phase, this variables are set to the same string as the two `TAB_...` variables. It is not possible at this time to host the add-in and the Teams app on distinct domains.

1. Add the following line to the end of the .env.dev file:

    ```
    ADDIN_ENDPOINT=
   ```
   
1. In the Teams manifest template, add the placeholder `${{ADDIN_DOMAIN}}` to the top of the "validDomains" array. The Teams Toolkit will replace this with a localhost domain when you are developing locally. When you deploy the finished Teams app to staging or production as described below in [Move the application to Azure](#move-the-application-to-azure), Teams Toolkit will replace the placeholder with the staging/production URI. The following is an example:

    ```json
    "validDomains": [
        "${{ADDIN_DOMAIN}}",
        
        // other domains or placeholders
    ],
    ```

1. Copy the entire "extensions" property from the add-in's manifest into the Teams app manifest as a top-level property.

## Copy the Outlook Add-in files to the Teams app project

1. Your Teams **tab** app starts with the following folder structure.

    ```
    |-- .vscode/
    |-- appPackage/
    |-- build\appPackage/
    |-- infra/
    |-- node_modules/
    |-- public/
    |-- src/            <!--your current source code-->
    |-- teamsfx/
    |-- package-lock.json
    |-- package.json
    |-- <!-- possibly other files, depending on how the project was created -->
    ```

1. Create a top-level folder called "add-ins" in the Teams app project.
1. Copy the following files and folders from the add-in project to the "add-ins" folder of the Teams app project.

    - /assets
    - /src
    - .eslintrc.json
    - babel.config.json
    - tsconfig.json

    **NOTE**: Do *not* copy over the manifest.json file.

    Your folder structure should now look like the following:

        ```
        |-- .vscode/
        |-- appPackage/
        |-- build\appPackage/
        |-- infra/
        |-- node_modules/
        |-- public/
        |-- src/
        |-- package-lock.json
        |-- package.json
        |-- <!-- possibly other files, depending on how the project was created -->
        |-- add-ins/
        |   |-- assets/
        |   |-- src/
        |   |   |-- commands/
        |   |   |-- taskpane/
        |   |-- .eslintrc.json
        |   |-- babel.config.json
        |   |-- tsconfig.json
        |-- teamsfx/
    ```

1. Copy the webpack.config.js file from the root of the add-in project to the root of the Teams project.

## Edit the tooling configuration files

1. Open the package.json file in the add-in project and the package.json file in the Teams project. In the next few steps you will copy markup from the former to the latter.
1. Copy the entire "config" property. Then change the "dev_server_port" value to `53000`.
1. Copy all of the properties in the "scripts" object to the destination "scripts". This will create redundant "build" and "start" scripts. Rename the ones you copied to "buildAddin" and "startAddin".
1. Just below the "startAddin" script are "start:desktop" and "start:web". Change these to "startAddin:desktop" and "startAddin:web", respectively.
1. Several of the scripts you copied in the last step have a `manifest.json` parameter. In each of these, change the parameter to `./build/AppPackage/manifest.local.json`. For example, 

    ```
    "startAddin:desktop": "office-addin-debugging start manifest.json desktop",
    ```

    should be changed to 

    ```
    "startAddin:desktop": "office-addin-debugging start ./build/AppPackage/manifest.local.json desktop",
    ```

1. Copy all of the properties in the "dependencies" object to the destination "dependencies".
1. Copy all of the properties in the "devDependencies" object to the destination "devDependencies". This will create redundant "typescript" properties. Delete the one with the lowest value. For example, if they are `"typescript": "^4.1.2"` and `"typescript": "^4.3.5"`, delete the first one.
1. In Visual Studio Code, open the **TERMINAL**. Be sure you at the root of the Teams project, then run the command `npm update`. 
1. In the webpack.config.js in the root of the Teams project, find all the URLs that begin with `"./src`, such as `template: "./src/taskpane/taskpane.html"`. Add `/add-ins` after the "." to each of them. For example, `"./src/taskpane/taskpane.html"` should be changed to `"./add-ins/src/taskpane/taskpane.html"`.
1. Near the end of the webpack.config.js file there is a line that assigns a port for the webpack dev server. Change the value from `3000` to `53000`.
 
1. In the Teams app project, open the teamsfx/app.local.yml file and find the `configureApp` section. Use the `#` character to comment out the lines that validate the manifest template. This is necessary because the Teams manifest validation system is not yet compatible with the changes you made to the manifest template. When you are done, the `configureApp` section should begin like the following:

    ```
    configureApp:
      - uses: file/updateEnv # Generate env to .env file
        with:
          envs:
            TAB_DOMAIN: localhost:53000
            TAB_ENDPOINT: https://localhost:53000
    #  - uses: teamsApp/validate
    #    with:
    #      manifestPath: ./appPackage/manifest.template.json # Path to manifest template
    
    # remainder of the section omitted
    
   ```

1. Open the .vscode\tasks.json file and add the following objects to the top of the "tasks" array. Note the following about this markup: 

    - It adds tasks similar to what is in the tasks.json of the add-in project to debug and stop debugging.
    - It also adds a "Start Add-in Locally" task that combines the tab app's "Create resources" task with the add-in's debugging task and specifies that they must run in that order.
    - The "Create resources" task generates the final manifest.
    - The "Debug: Outlook Desktop" calls the `startAddin:desktop` script that you added to the package.json in an earlier step. It also adds a command switch to that specifies that the add-in should be sideloaded in Outlook. 

    ```
    {
            "label": "Start Add-in Locally",
            "dependsOn": [
                "Create resources",
                "Debug: Outlook Desktop"
            ],
            "dependsOrder": "sequence"
         },
         {
            "label": "Debug: Outlook Desktop",
            "type": "npm",
            "script": "startAddin:desktop -- --app outlook",
            "presentation": {
              "clear": true,
              "panel": "dedicated",
            },
            "problemMatcher": []
          },
          {
            "label": "Stop Debug",
            "type": "npm",
            "script": "stop",
            "presentation": {
              "clear": true,
              "panel": "shared",
              "showReuseMessage": false
            },
            "problemMatcher": []
          },
          ```

1. Open the .vscode\launch.json file, which configures the **RUN AND DEBUG** UI in Visual Studio Code, and add the following object to the top of the "configurations" array.

    ```
    {
        "name": "Outlook Desktop (Edge Chromium)",
        "type": "msedge",
        "request": "attach",
        "port": 9229,
        "timeout": 600000,
        "webRoot": "${workspaceRoot}",
        "preLaunchTask": "Debug: Outlook Desktop",
        "postDebugTask": "Stop Debug"
    },
    ```

1. In the same file, add the following object to the top of the "compounds" array. This markup adds "Debug Add-in (Edge)" as the third option from the top in the **RUN AND DEBUG** drop down. It uses the configuration that you added in the previous step and it runs the task "Start Add-in Locally" that you added to tasks.json in an earlier step.

    ```
    {
        "name": "Debug Add-in (Edge)",
        "configurations": [
            "Outlook Desktop (Edge Chromium)"
        ],
        "preLaunchTask": "Start Add-in Locally",
        "presentation": {
            "group": "all",
            "order": 3
        },
        "stopAll": true
    },
    ```

EVERYTHING BELOW THIS POINT WAS COPIED FROM ANOTHER ARTICLE AND i DON'T KNOW IF IT IS ACCURATE OR NEEDED.

## Setup local debug environment

1. Generate debug profile with TeamsFx CLI.
    ```
    > teamsfx init debug
    ? Teams Toolkit: Select your development environment: Visual Studio Code (JS/TS)
    ? Teams Toolkit: Select the capability of your app: Tab
    ? Teams Toolkit: Are you developing with SPFx?: No
    ? Teams Toolkit: Teams Toolkit will generate the following files (existing files with duplicated names will be overwritten), would you like to proceed?
      teamsfx/
        - app.local.yml
        - .env.local
        - settings.json
        - run.js
      .vscode/
        - launch.json
        - settings.json
        - tasks.json
    : Yes
    ```
1. Manually merge the content in `.vscode` and `teamsfx` folder with yours. Update the `app.local.yml` and `run.js` to target your tab app code.
Here is an sample project for reference. [Hello World Bot with Tab](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-bot-with-tab).

1. Try local debug with Visual Studio Code.

## Move the application to Azure

1. Generate Bicep file for Azure infrastructure with TeamsFx CLI.
    ```
    > teamsfx init infra
    ? Teams Toolkit: Select your development environment: Visual Studio Code (JS/TS)
    ? Teams Toolkit: Select the capability of your app: Tab
    ? Teams Toolkit: Are you developing with SPFx?: No
    ? Teams Toolkit: Teams Toolkit will generate the following files (existing files with duplicated names will be overwritten), would you like to proceed?
      teamsfx/
        - app.yml
        - .env.dev
        - settings.json
      infra/
        - azure.bicep
        - azure.parameters.json
    : Yes
    ```
1. Manually merge the content in `infra` and `teamsfx` folder with yours.
    Here is a sample project for reference. [Hello World Bot with Tab](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-bot-with-tab).

1. Run `Teams: Provision in the cloud` command in Visual Studio Code to apply the bicep to Azure.

1. Run `Teams: Deploy to cloud` command in Visual Studio Code to deploy your Tab app code to Azure.

1. Open the `Run and Debug Activity Panel` and select `Launch Remote (Edge)` or `Launch Remote (Chrome)`. Press F5 to preview your Teams app.

## What’s next

There are other commonly suggested next steps, for example:

- [Add authentication and make a Graph API call](https://learn.microsoft.com/microsoftteams/platform/toolkit/add-single-sign-on?pivots=visual-studio-code)
- [Set up CI/CD pipelines](https://github.com/OfficeDev/TeamsFx/wiki/How-to-automate-cicd-pipelines)
- [Call a backend API](https://github.com/OfficeDev/TeamsFx/wiki/How-to-integrate-Azure-Functions-with-your-Teams-app)

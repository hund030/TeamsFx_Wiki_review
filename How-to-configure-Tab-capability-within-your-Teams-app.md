# Configure Tab capability within your Teams app

## Introduction

Tabs are Teams-aware webpages embedded in Microsoft Teams. They're simple HTML <iframe\> tags that point to domains declared in the app manifest and can be added as part of a channel inside a team, group chat, or personal app for an individual user. You can include custom tabs with your app to embed your own web content in Teams or add Teams-specific functionality to your web content. Learn more from [Build tabs for Teams
](https://learn.microsoft.com/microsoftteams/platform/tabs/what-are-tabs).

## Prerequisites

To configure tab as additional capability, please make sure:

- You have a Teams application and its manifest.
- You have a Microsoft 365 account to test the application.

Before going on, we strongly suggest you should create and go through a Tab app with Teams Toolkit.
Create a Tab app with Teams Toolkit(https://learn.microsoft.com/microsoftteams/platform/toolkit/create-new-project?pivots=visual-studio-code)

Following are the steps to add Tab capability:
1. [Configure Tab capability in Teams application manifest](#configure-tab-capability-in-teams-application-manifest).
1. [Setup local debug environment](#setup-local-debug-environment).
1. [Move the application to Azure](#move-the-application-to-azure).

## Configure Tab capability in Teams application manifest

1. You can configure your tab within group or channel, or personal scope in you Teams application manifest `appPackage/manifest.template.json`.
    Examples:
    ```
      "staticTabs": [
          {
              "entityId": "index",
              "name": "Personal Tab",
              "contentUrl": "${{TAB_ENDPOINT}}/index.html#/tab",
              "websiteUrl": "${{TAB_ENDPOINT}}/index.html#/tab",
              "scopes": [
                  "personal"
              ]
          }
      ],
    ```
    ```
      "configurableTabs": [
          {
              "configurationUrl": "${{TAB_ENDPOINT}}/index.html#/config",
              "canUpdateConfiguration": true,
              "scopes": [
                  "team",
                  "groupchat"
              ]
          }
      ],
    ```

1. Add your tab domain to the `validDomains` field.
    Example:
    ```
    "validDomains": [
        "${{TAB_DOMAIN}}"
    ],
    ```
    `TAB_ENDPOINT` and `TAB_DOMAIN` are built-in variables of Teams Toolkit. They will be replaced with the true endpoint in runtime based on your current environment(local, dev, etc.).

## Setup local debug environment

1. Bring your Tab app code into your project. If you don't have one, you can create a new Tab app project with Teams Toolkit and copy the source code to into your current project.
Your folder structure may be like:
    ```
    .
    |-- teasmfx/
    |-- infra/
    |-- appPackage/
    |-- tabs/           <!--tab app source code-->
    |   |-- src/
    |   |   |-- index.tsx
    |   |-- package.json
    |-- src/            <!--your current source code-->
    |   |-- index.ts
    |-- package.json
    ```

    It is suggested to re-organize the folder structure as:

    ```
    .
    |-- teasmfx/
    |-- infra/
    |-- appPackage/
    |-- tabs/           <!--tab app source code-->
    |   |-- src/
    |   |   |-- index.tsx
    |   |-- package.json
    |-- bot/            <!--move your current source code to a new sub folder-->
    |   |-- src/
    |   |   |-- index.ts
    |   |-- package.json
    ```

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
Here is an sample project for reference. [TODO]().

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
    Here is an sample project for reference. [TODO]().

1. Run `Teams: Provision in the cloud` command in Visual Studio Code to apply the bicep to Azure.

1. Run `Teams: Deploy to cloud` command in Visual Studio Code to deploy your Tab app code to Azure.

1. Open the `Run and Debug Activity Panel` and select `Launch Remote (Edge)` or `Launch Remote (Chrome)`. Press F5 to preview your Teams app.

## What’s next

There are other commonly suggested next steps, for example:

- [Add authentication and make a Graph API call](https://learn.microsoft.com/microsoftteams/platform/toolkit/add-single-sign-on?pivots=visual-studio-code)
- [Set up CI/CD pipelines](#How-to-add-CICD)
- [Call a backend API](#How-to-add-Azure-Functions)

# Configure Bot capability within your Teams app

## Introduction

A bot, chatbot, or conversational bot is an app that responds to simple commands sent in chat and replies in meaningful ways. Examples of bots in everyday use include: bots that notify about build failures, bots that provide information about the weather or bus schedules, or provide travel information. A bot interaction can be a quick question and answer, or it can be a complex conversation. Being a cloud application, a bot can provide valuable and secure access to cloud services and corporate resources. Learn more from [Build bots for Teams
](https://learn.microsoft.com/microsoftteams/platform/bots/what-are-bots).

## Prerequisites

To configure bot as additional capability, please make sure:

- You have a Teams application and its manifest.
- You have a Microsoft 365 account to test the application.

For adding bot to a tab Teams app, please go to: 
[Add bot to a tab Teams app](#add-bot-to-a-tab-teams-app).

For adding bot to a message extension Teams app, please go to: [Add bot to a message extension Teams app](#add-bot-to-a-message-extension-teams-app).

## Add bot to a tab Teams app:

Following are the steps to add Bot capability to a tab app:
1. [Create a bot Teams app using Teams Toolkit](#create-a-bot-app-using-teams-toolkit).
1. [Update manifest file](#configure-bot-capability-in-teams-application-manifest).
1. [Bring bot code to your project](#bring-bot-code-to-your-project).
1. [Setup local debug environment](#setup-local-debug-environment).
1. [Move the application to Azure](#move-the-application-to-azure).

### Create a bot app using Teams Toolkit

Please check the guide [Create a bot app with Teams Toolkit](https://learn.microsoft.com/microsoftteams/platform/toolkit/create-new-project?pivots=visual-studio-code)

### Configure bot capability in Teams application manifest

1. You can configure bot in `appPackage/manifest.template.json`. You can also refer to [bot schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#bots) if you want to customize.

    Example: 
    ```json
        "bots": [
            {
                "botId": "${{BOT_ID}}",
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
        ]
    ```

1. Add your bot domain to the `validDomains` field.
    Example:
    ```json
    "validDomains": [
        "${{BOT_DOMAIN}}"
    ],
    ```
    `BOT_ID` and `BOT_DOMAIN` are built-in variables of Teams Toolkit. They will be replaced with the true value in runtime based on your current environment(local, dev, etc.).

### Bring bot code to your project

1. Bring your own bot app code into your project. If you don't have one, you can use the bot app project previously created and copy the source code to into your current project. We suggest you to copy them into a `bot/` folder. Your folder structure will be like:
    ```
    |-- appPackage/
    |-- env/
    |-- infra/
    |-- bot/           <!--bot source code-->
    |   |-- index.ts
    |   |-- config.ts
    |   |-- teamsBot.ts
    |   |-- package.json
    |-- src/            <!--your current source code-->
    |   |-- index.ts
    |-- package.json
    |-- teamsapp.local.yml
    |-- teamsapp.yml
    ```
    We suggest you to re-organize the folder structure and create a root package.json as:
     ```
    |-- appPackage/
    |-- env/
    |-- infra/
    |-- bot/            <!--bot source code-->
    |   |-- index.ts
    |   |-- config.ts
    |   |-- teamsBot.ts
    |   |-- package.json
    |-- tab/           <!--move your current source code to a new sub folder-->
    |   |-- src/
    |   |   |-- index.tsx
    |   |-- package.json
    |-- package.json <!--root package.json-->
    |-- teamsapp.local.yml
    |-- teamsapp.yml
    ```
1. Add following to your root package.json:
    ```json
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
      "install:bot": "cd bot && npm install",
      "install:tab": "cd tab && npm install",
      "install": "concurrently \"npm run install:bot\" \"npm run install:tab\"",
      "dev:bot": "cd bot && npm run dev",
      "start:tab": "cd tab && npm run start",
      "build:tab": "cd tab && npm run build",
      "build:bot": "cd bot && npm run build",
      "build": "concurrently \"npm run build:tab\" \"npm run build:bot\""
    },
    "devDependencies": {
        "@microsoft/teamsfx-run-utils": "alpha"
    },
    "dependencies": {
        "concurrently": "^7.6.0"
    },
    ```

### Setup local debug environment

1. Manually merge the content in `.vscode/` and `teamsapp.local.yml` folders with yours. Update the `teamsapp.local.yml` and `run.js` to target your bot code.
Here is an [sample project](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-bot-with-tab) for reference.


1. Open the `Run and Debug Activity Panel` and select `Debug (Edge)` or `Debug (Chrome)`. Press F5 to preview your Teams app locally.

### Move the application to Azure

1. Manually merge the content in `infra/` and `teamsapp.yml` folder with yours.
Here is an [sample project](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-bot-with-tab) for reference.
1. Run `Teams: Provision in the cloud` command in Visual Studio Code to apply the bicep to Azure.

1. Run `Teams: Deploy to cloud` command in Visual Studio Code to deploy your app code to Azure.

1. Open the `Run and Debug Activity Panel` and select `Launch Remote (Edge)` or `Launch Remote (Chrome)`. Press F5 to preview your Teams app.

## Add bot to a message extension Teams app:

Since bot and message extension are both implemented on top of the Bot support architecture within Teams, adding bot to a message extension Teams app is simpler than adding to a tab Teams app.

Following are the steps to add bot capability to a message extension app:
1. [Create a bot Teams app using Teams Toolkit](#create-a-bot-app-using-teams-toolkit).
1. [Update manifest file](#configure-bot-capability-in-teams-application-manifest).
1. [Bring bot code to your project](#bring-bot-code-to-your-project).

### Create a bot app using Teams Toolkit

Please check the guide [Create a bot app with Teams Toolkit](https://learn.microsoft.com/microsoftteams/platform/toolkit/create-new-project?pivots=visual-studio-code)

### Configure bot capability in Teams application manifest

1. You can configure bot in `appPackage/manifest.template.json`. You can also refer to [bot schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#bots) if you want to customize.

    Example:  
    ```json
        "bots": [
            {
                "botId": "${{BOT_ID}}",
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
        ]
    ```

1. Add your bot domain to the `validDomains` field.
    Example:
    ```json
    "validDomains": [
        "${{BOT_DOMAIN}}"
    ],
    ```
    `BOT_ID` and `BOT_DOMAIN` are built-in variables of Teams Toolkit. They will be replaced with the true value in runtime based on your current environment(local, dev, etc.).

### Bring bot code to your project

1. If you are adding bot to a bot Teams app, then you should already have a class that extends `TeamsActivityHandler`. Bring your own bot code, or copy code from your previously created bot app to your own class. Below is an example if you copy code from Teams Toolkit created bot app:

    ```ts
      public class YourHandler extends TeamsActivityHandler{

        // bot code
        constructor(){
          super();
          this.likeCountObj = { likeCount: 0 };
          this.onMessage(async (context, next) => {});
          this.onMembersAdded(async (context, next) => {});
        }
        async onAdaptiveCardInvoke(context: TurnContext,
    invokeValue: AdaptiveCardInvokeValue):  Promise<AdaptiveCardInvokeResponse> {};

        /**
         * your own message extension code
        */
      }
    ```

## What’s next

There are other commonly suggested next steps, for example:

- [Set up CI/CD pipelines](#How-to-add-CICD)

## Introduction
Tabs are Teams-aware webpages embedded in Microsoft Teams. They're simple HTML <iframe\> tags that point to domains declared in the app manifest and can be added as part of a channel inside a team, group chat, or personal app for an individual user. You can include custom tabs with your app to embed your own web content in Teams or add Teams-specific functionality to your web content. Teams tab can be any web-based application, of course, your custom tabs can be a SharePoint web part.

You can extend Microsoft Teams with additional functionality by integrating your applications. This allows you expose functionality in the context of your users’ work helping them be more productive. By [building Microsoft Teams applications using SharePoint Framework](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/integrate-with-teams-introduction), you can save costs on hosting infrastructure and simplify the deployment and operation process.

## Step-by-step guide to configure additional SPFx tab in your Teams app 
Sometimes you will need to have multiple tabs for your Teams tab project. This document will introduce you how to add additional SPFx tab into your SPFx Teams project (assume you already have one, if not you can quickly create a SPFx Teams project using Teams Toolkit). This document will not include intructions about how to add SPFx tab into other type of Teams projects. e.g. Azure-hosted tab or bot. 

### Prerequisite
To add additional SPFx tab, please make sure: 
- You have a Teams application with SPFx tab capability and its manifest. 
- You have a Microsoft 365 account to test the application. 

Following are steps to add additional SPFx tab:
1. [Scaffold additional web part](#scaffold-additional-web-part)
2. [Configure SPFx tab capability in Teams application manifest](#configure-spfx-tab-capability-in-teams-application-manifest)
3. [Run the capability locally](#run-the-capability-locally)
4. [Move the application to cloud](#move-the-application-to-cloud)

### Scaffold additional web part
Teams Toolkit leverages Yeoman to scaffold SPFx solution and add web part. Before we start, the two packages are needed to run Yeoman:
- `yo`: CLI tool for running Yeoman generators
- `@microsoft/generator-sharepoint`: Yeoman generator for the SharePoint Framework

You can use packages that are globally installed by yourself or locally installed by Teams Toolkit to do scaffolding:

**Use Globally installed packages**
1. Install the two packages globally with `npm i -g yo @microsoft/generator-sharepoint` (Please make sure to install the same version of `@microsoft/generator-sharepoint` package as your existing SPFx project)
2. Execute `yo @microsoft/sharepoint` command under `src` directory

**Use locally installed packages by Teams Toolkit**
1. Check packages installed under `${HOME}\.fx\bin` (They should be installed under `yo` and `spGenerator` directory respectively if you have created SPFx project before)
2. Execute `${path_to_yo_executable}\yo ${HOME}\.fx\bin\spGenerator\node_modules\@microsoft\generator-sharepoint\lib\generators\app\index.js` command under `src` directory

### Configure SPFx tab capability in Teams application manifest
Teams app is represented by its manifest, where SPFx tab capability is defined. Follow these steps to add additional SPFx tab
1. Get the new web part id from web part manifest (located at `src\src\webparts\${webPartName}\${webPartName}.manifest.json`) `id` field. 
2. Configure your new web part in local environment by adding following static tab in `staticTabs` section in the `appPackage\manifest.local.json` file (Replace `${componentId}` and `${webPartName}` variable with actual value).
```
        {
            "entityId": "${componentId}",
            "name": "${webPartName}",
            "contentUrl": "https://{teamSiteDomain}/_layouts/15/TeamsLogon.aspx?SPFX=true&dest={teamSitePath}/_layouts/15/TeamsWorkBench.aspx%3FcomponentId=${componentId}%26teams%26personal%26forceLocale={locale}%26loadSPFX%3Dtrue%26debugManifestsFile%3Dhttps%3A%2F%2Flocalhost%3A4321%2Ftemp%2Fmanifests.js",
            "websiteUrl": "https://products.office.com/en-us/sharepoint/collaboration",
            "scopes": [
                "personal"
            ]
        }
```
3. Configure your new web part in other environments by adding following static tab in `staticTabs` section in the `appPackage\manifest.json` file (Replace `${componentId}` and `${webPartName}` variable with actual value).
```
        {
            "entityId": "${componentId}",
            "name": "${webPartName}",
            "contentUrl": "https://{teamSiteDomain}/_layouts/15/TeamsLogon.aspx?SPFX=true&dest=/_layouts/15/teamshostedapp.aspx%3Fteams%26personal%26componentId=${componentId}%26forceLocale={locale}",
            "websiteUrl": "https://products.office.com/en-us/sharepoint/collaboration",
            "scopes": [
                "personal"
            ]
        }
```

4. You can set environment variables within the manifest if you have multiple environments.

### Run the capability locally

To run SPFx project locally, you can `F5` to debug in Hosted workbench or Teams workbench like before.

### Move the application to cloud

To move your application to cloud, you can deploy and publish like before.

## Additional references
You can check the following links for more information: 
- [Create the URL of web parts used in Microsoft Teams apps](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/deployment-spfx-teams-solutions#dynamically-reference-the-underlying-sharepoint-site-urls)

- [Connect to SharePoint APIs](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/connect-to-sharepoint)
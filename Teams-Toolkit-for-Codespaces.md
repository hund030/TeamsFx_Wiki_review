# Using Teams Toolkit for App Development in Codespaces

## Introduction
[GitHub Codespaces](https://docs.github.com/en/codespaces/overview) is a cloud-based development environment for your GitHub projects that allows developers to develop and collaborate on code from anywhere, using a web browser or Visual Studio Code. With GitHub Codespaces, Teams Toolkit provides new "install-less" getting started experience for Teams app development, so you can quickly get started with a fully configured dev environment without spending time on local environment setup.

## Prerequisites
Before getting started building Teams app with Codespaces, please make sure: 
1. You have a GitHub account which will be used to create a Codespaces instance.
2. You have a M365 Account with sideloading permissions. You may find the setup instructions in the [Prepare your Microsoft 365 tenant](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/prepare-your-o365-tenant). If you don’t already have a Microsoft 365 account, you can register for a free one through the [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program).

## Getting started from codespaces-enabled sample
| Sample name | Entry point |
|-----|-----|
| Hello World Tab | [![Open hello-world tab in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=dol%2Fcodespaces&repo=348288141&machine=standardLinux32gb&devcontainer_path=.devcontainer%2Fhello-world-tab-codespaces%2Fdevcontainer.json&location=SouthEastAsia) |
| Notification Bot| [![Open notification app in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=dol%2Fcodespaces&repo=348288141&machine=basicLinux32gb&devcontainer_path=.devcontainer%2Fnotification-app%2Fdevcontainer.json&location=SouthEastAsia) |
| NPM Search Message Extension | Coming soon! |

## Enable Codespaces for Teams app
Besides getting started from our codespaces-enabled samples, you can enable codespaces configuration for your TeamsFx projects on GitHub by following the steps below:

### Add a dev container configuration
To set up your repository to use a custom dev container for building apps with Teams Toolkit, you'll need to create a `devcontainer.json` file and place it in the .devcontainer folder located in the root directory of your project. You can start from using the following sample `devcontainers.json`:

#### Sample `devcontainer.json` for JS/TS Tab App:
```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/javascript-node
{
  "name": "tab-codespaces",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:16",
  // Use 'forwardPorts' to make a list of ports inside the container available locally.
  "forwardPorts": [
    53000
  ],
  "portsAttributes": {
    "53000": {
      "label": "tab",
      "protocol": "https"
    }
  },  
  "remoteUser": "node",
  "customizations": {
    "vscode": {
      "extensions": [
        "TeamsDevApp.ms-teams-vscode-extension",
      ]
    }
  },
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {
      "version": "latest"
    }
  }
}
```

#### Sample `devcontainer.json` for JS/TS Bot App
```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/javascript-node
{
  "name": "bot-codespaces",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:16",
  // Use 'forwardPorts' to make a list of ports inside the container available locally.
  "forwardPorts": [
    3978
  ],
  "remoteUser": "node",
  "customizations": {
    "vscode": {
      "extensions": [
        "TeamsDevApp.ms-teams-vscode-extension",
      ]
    }
  },
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {
      "version": "latest"
    }
  }
}
```

### Update TeamsFx configuration for running app in Codespaces
1. Update `.vscode/tasks.json`
Add the following debug tasks in `.vscode/tasks.json`:
```json
{
    "version": "2.0.0",
    "tasks": [
        ...
        {
            "label": "Start Teams App in Codespaces",
            "dependsOn": [
                "Validate prerequisites",
                "Configure port visibility",
                "Provision",
                "Deploy",
                "Start application",
                "Open Teams Web Client"
            ],
            "dependsOrder": "sequence"
        },
        {
            "label": "Configure port visibility",
            "type": "shell",
            "command": "gh codespace ports visibility 53000:public -c $CODESPACE_NAME"
        },
        {
            // Launch Teams web client.
            // See https://aka.ms/teamsfx-deploy-task to know the details and how to customize the args.
            "label": "Open Teams Web Client",
            "type": "teamsfx",
            "command": "launch-web-client",
            "args": {
              "env": "local"
            }
        }
        ...
    ]
}
```

2. added launch configuration `.vscode/launch.json`:


3. Update `teamsapp.local.yml`

### Commit all the above code changes to your project repository


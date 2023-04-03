## Collaborating on TeamsFx Project
Previous version of Teams Toolkit is not easy for multiple users to develop the same project due to missing privilege to access Teams APP and AAD APP. If multiple developers want to share remote resources and work together, they need to manually handle permissions of Teams App and AAD APP which need deep understanding the low-level details about the TeamsFx project.

Teams Toolkit now natively support add other collaborators for TeamsFx project which is much easy and straightforward for collaborative development.

### Collaborating - Use VSCode

#### Usage

- To use collaboration feature, you need to **login M365 and Azure account**(Only needed for Tab project, and for SPFX project, it is not needed) and your **TeamsFx project should be provisioned first**

- In the Teams Toolkit panel, click ENVIRONMENT, and expand an environment name which you want to work with others, then you can see Manage Collaborator button:

  ![collaboration-buttons](https://user-images.githubusercontent.com/63089166/229402317-9211c1ea-d8c2-41da-aac6-e919f4f96ea6.png)


- Add App Owners:
  - Click Manage Collaborator button
  - Select `Add App Owners`
    ![collaboration-add-app-owners](https://user-images.githubusercontent.com/63089166/229402708-03d27794-bae3-49b3-8882-352239b72300.png)
  - Select the apps you want to add app owners for
    ![collaboration-select-app](https://user-images.githubusercontent.com/63089166/229402856-6af63850-03c3-45fa-a80b-05c5122713e4.png)
  - (Optional) Select and confirm Teams `manifest.json` file
    ![collaboration-select-manifest](https://user-images.githubusercontent.com/63089166/229403125-95ab5594-4cdd-4b03-a30f-2f5579017b33.png)
  - (Optional) Select and confirm Azure Active Directory app `aad.manifest.json` file
    ![collaboration-select-aad-manifest](https://user-images.githubusercontent.com/63089166/229403266-189fe064-cf01-4dbc-be8c-e25e31af2397.png)
  - Input the M365 account email address you want to add as app owner (this account should be in the same tenant with your M365 account)
    ![collaboration-input-collaborator](https://user-images.githubusercontent.com/63089166/229403537-647d9a1b-4443-4fe7-8b8a-109e2925fae0.png)

- List App Owners:
  - Click Manage Collaborator button
  - Select `List App Owners`
    ![collaboration-add-app-owners](https://user-images.githubusercontent.com/63089166/229402708-03d27794-bae3-49b3-8882-352239b72300.png)
  - Select the apps you want to add app owners for
    ![collaboration-select-app](https://user-images.githubusercontent.com/63089166/229402856-6af63850-03c3-45fa-a80b-05c5122713e4.png)
  - (Optional) Select and confirm Teams `manifest.json` file
    ![collaboration-select-manifest](https://user-images.githubusercontent.com/63089166/229403125-95ab5594-4cdd-4b03-a30f-2f5579017b33.png)
  - (Optional) Select and confirm Azure Active Directory app `aad.manifest.json` file
    ![collaboration-select-aad-manifest](https://user-images.githubusercontent.com/63089166/229403266-189fe064-cf01-4dbc-be8c-e25e31af2397.png)


- Share your project with the collaborator:
  - Commit your project to GitHub repository
  - Collaborator clone the project to his computer
  - If your project contains Bot capability, share secret file `env\.env.[Environment-Name].user` to the collaborator, and collaborator should copy the secret file to same place in the project

- Collaborator login to M365 account

- For Tab project, login Azure account which **at least has contributor permission** for all the Azure resources. Project owner can refer this [link](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal?tabs=current) to assign Azure roles using the Azure portal.

- For SPFX project, project owner needs to manually setup access policy via SharePoint admin center, please refer to this [link](https://docs.microsoft.com/en-us/sharepoint/manage-site-collection-administrators) for more details.

- Now collaborator can develop, provision and deploy the project

### Collaborating - Use CLI
Teams Toolkit CLI provides `teamsFx permission` Commands for collaboration scenario.

#### Commands
| `teamsFx permission` Commands | Descriptions |
|:------------------------------|-------------|
| `teamsfx permission grant` | Grant permission for collaborator's M365 account for the project |
| `teamsfx permission status` | Show permission status for the project | 

***

#### Parameters for `teamsfx permission grant`
- ##### `--env`
	**(Required)** Provide env name.

- ##### `--email`
	**(Required)** Provide collaborator's M365 email address. Note that the collaborators's account should be in the same tenant with creator.

- #### `--teams-app-manifest`
  Provide path of your Teams app `manifest.json` file.

- #### `--aad-app-manifest`
  Provide path of your Azure Active Directory `aad.manifest.json` file.

#### Parameters for `teamsfx permission status`
- ##### `--env`
	**(Required)** Provide env name.

- ##### `--list-all-collaborators`
	With this flag, Teams Toolkit CLI will print out all collaborators for this project.

- #### `--teams-app-manifest`
  Provide path of your Teams app `manifest.json` file.

- #### `--aad-app-manifest`
  Provide path of your Azure Active Directory `aad.manifest.json` file.

### Examples
Here are some examples for you to better handling permission for `TeamsFx` projects.

#### Grant Permission
```bash
teamsfx permission grant --env dev --email user-email@user-tenant.com --teams-app-manifest your-path-of-teams-app-manifest --aad-app-manifest your-path-of-aad-app-manifest
```

#### Show current user Permission Status
```bash
teamsfx permission status --env dev --teams-app-manifest your-path-of-teams-app-manifest --aad-app-manifest your-path-of-aad-app-manifest
```

#### List All Collaborators
```bash
teamsfx permission status --env dev --list-all-collaborators  --teams-app-manifest your-path-of-teams-app-manifest --aad-app-manifest your-path-of-aad-app-manifest
```

### Limitations of collaboration feature
- Azure related permissions should be handled manually by the Azure subscription administrator via Azure portal, different Azure account should at least have contributor role for the subscription so that developers can work together to provision and deploy TeamsFx project. For more information about how to assign Azure roles using the Azure portal, you can refer this [doc](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal?tabs=current).

- If your project is SPFX project, you need to manually setup access policy via SharePoint admin center, please refer to this [link](https://docs.microsoft.com/en-us/sharepoint/manage-site-collection-administrators) for more details.

- If the project contains Bot capability, creator of the project should share secret file `.fx\states\[Environment-Name].userdata` with the collaborator, otherwise collaborator will receive errors as below when provision: 

  ![403-error](https://user-images.githubusercontent.com/5545529/168981368-2e97b9df-0f37-4eaa-acd1-33e85492b4cb.png)


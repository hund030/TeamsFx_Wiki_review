Teams Toolkit helps you to access existing APIs for building Teams applications. These APIs are developed by your organization or third-party. 

# Requirement
Make sure that your project contains backend service, such as **Azure Function** or **Azure Bot Service** to host your API Connection.

# Steps to add API Connection
The following steps help you to add API connection manually:
- [Step 1: Add SDK to project](#step-1-add-sdk-to-project)
- [Step 2: Provide ApiClient for project](#step-2-provide-apiclient-for-project)
- [Step 3: Add Configuration for local debugging](#step-3-add-configuration-for-local-debugging)
- [Step 4: Add Configuration for Azure](#step-4-add-configuration-for-azure)

## Step 1: Add SDK to project
Add a reference to the `@microsoft/teamsfx` package to `package.json`.


## Step 2: Provide ApiClient for project
The TeamsFx SDK supports 5 different ways to connect to the API. You can create a module to connect your API and export the `apiClient` to provide for the project.

<details>
<summary><b>Basic Auth
</b></summary>

Sample code for Basic Auth
```javascript
const teamsfxSdk = require("@microsoft/teamsfx");

// Initialize a new axios instance to call your API
const authProvider = new teamsfxSdk.BasicAuthProvider(
  process.env.TEAMSFX_API_USERNAME,
  process.env.TEAMSFX_API_PASSWORD
);
const apiClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_ENDPOINT,
  authProvider
);
module.exports.apiClient = apiClient;
```
</details>
<details>
<summary><b>Certification
</b></summary>

Sample code for Certification
```javascript
const teamsfxSdk = require("@microsoft/teamsfx");

// Initialize a new axios instance to call your API
const authProvider = new teamsfxSdk.CertificateAuthProvider(
  // TODO: 
  // 1. Add code to read your certificate and private key.
  // 2. Replace "<your-cert>" and "<your-private-key>" with your actual certificate and private key values
  // If you have a .pfx certificate, you can use the `createPfxCertOption` function to initialize your certificate
  teamsfxSdk.createPemCertOption("<your-cert>", "<your-private-key>")
);
const apiClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_ENDPOINT,
  authProvider
);
module.exports.apiClient = apiClient;
```
</details>
<details>
<summary><b>Azure Active Directory
</b></summary>

There are 2 scenarios here, please choose one of them. 
- Scenario 1 is reusing the project AAD app, make sure your project contains an existing AAD app.
- Scenario 2 is using an existing AAD App.

```javascript
const teamsfxSdk = require("@microsoft/teamsfx");
// There are 2 scenarios here, please choose one of them. This sample uses the client credential flow to acquire a token for your API.
// Scenario 1. reuse the project AAD app.
const appAuthConfig: AppCredentialAuthConfig = {
  authorityHost: process.env.AAD_APP_OAUTH_AUTHORITY_HOST,
  clientId: process.env.TEAMSFX_API_CLIENT_ID,
  tenantId: process.env.TEAMSFX_API_TENANT_ID,
  clientSecret: process.env.TEAMSFX_API_CLIENT_SECRET,
};
// Scenario 2. use an existing AAD App.
const appAuthConfig: AppCredentialAuthConfig = {
  authorityHost: "https://login.microsoftonline.com",
  clientId: process.env.TEAMSFX_API_CLIENT_ID,
  tenantId: process.env.TEAMSFX_API_TENANT_ID,
  clientSecret: process.env.TEAMSFX_API_CLIENT_SECRET,
};
const appCredential = new AppCredential(appAuthConfig);
// Initialize a new axios instance to call your API
const authProvider = new teamsfxSdk.BearerTokenAuthProvider(
  // TODO: Replace '<your-api-scope>' with your required API scope
  async () => (await appCredential.getToken("<your-api-scope>")).token
);
const apiClient= teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_ENDPOINT,
  authProvider
);
module.exports.apiClient= apiClient;
```
</details>
<details>
<summary><b>API Key
</b></summary>

Sample code for API Key
```javascript
const teamsfxSdk = require("@microsoft/teamsfx");

// Initialize a new axios instance to call kudos, store API key in request header.
const authProvider = new teamsfxSdk.ApiKeyProvider(
  "{API-KEY-name}",
  process.env.TEAMSFX_API_API_KEY,
  teamsfxSdk.ApiKeyLocation.Header
);
// or store API key in request params.
const authProvider = new teamsfxSdk.ApiKeyProvider(
  "{API-KEY-name}",
  process.env.TEAMSFX_API_API_KEY,
  teamsfxSdk.ApiKeyLocation.QueryParams
);
const apiClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_ENDPOINT,
  authProvider
);
module.exports.apiClient = apiClient;
```
</details>
<details>
<summary><b>Custom Auth Implementation
</b></summary>

Sample code for Custom Auth Implementation
```javascript
const teamsfxSdk = require("@microsoft/teamsfx");

// A custom authProvider implements the `AuthProvider` interface.
// This sample authProvider implementation will set a custom property in the request header
class CustomAuthProvider {
  customProperty;
  customValue;

  constructor(customProperty, customValue) {
    this.customProperty = customProperty;
    this.customValue = customValue;
  }

  // Replace the sample code with your own logic.
  AddAuthenticationInfo = async (config) => {
    if (!config.headers) {
      config.headers = {};
    }
    config.headers[this.customProperty] = this.customValue;
    return config;
  };
}

const authProvider = new CustomAuthProvider(
  // You can also add configuration to the file `.env.teamsfx.local` and use `process.env.{setting_name}` to read the configuration. For example:
  //  process.env.TEAMSFX_API_CUSTOM_PROPERTY,
  //  process.env.TEAMSFX_API_CUSTOM_VALUE
  "customPropery",
  "customValue"
);
// Initialize a new axios instance to call your API
const apiClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_ENDPOINT,
  authProvider
);
module.exports.apiClient = apiClient;
```
</details>

You can import the `apiClient` (an Axios instance) in another file and call the APIs and authentication is now handled for you automatically.

Here is an example for a GET request to "relative_path_of_target_api":
```javascript
const { apiClient } = require("relative_path_to_this_file");
const response = await apiClient.get("relative_path_of_target_api");
// You only need to enter the relative path for your API.
// For example, if you want to call api https://my-api-endpoint/test and you configured https://my-api-endpoint as the API endpoint,
// your code will be: const response = await kudosClient.get("test");

const responseBody = response.data;
```


## Step 3: Add Configuration for local debugging
According to the `apiClient` created in Step 2, select the corresponding type to configure your API for local debugging.

<details>
<summary><b>Basic Auth
</b></summary>

Append your Api connection configuration to `env/.env.local`
```
...
// set up environment variables required by teamsfx
TEAMSFX_API_ENDPOINT =
TEAMSFX_API_USERNAME =
TEAMSFX_API_PASSWORD =
```
</details>
<details>
<summary><b>Certification
</b></summary>

Append your Api connection configuration to `env/.env.local`
```
...
// set up environment variables required by teamsfx
TEAMSFX_API_ENDPOINT =
```
</details>
<details>
<summary><b>Azure Active Directory
</b></summary>

There are 2 scenarios here, please choose one of them. 
- Scenario 1 is reusing the project AAD app, make sure your project contains an existing AAD app.
- Scenario 2 is using an existing AAD App.

Append your Api connection configuration to `env/.env.local`
```
...
// set up environment variables required by teamsfx
TEAMSFX_API_ENDPOINT =
// Scenario 1
TEAMSFX_API_TENANT_ID = 
TEAMSFX_API_CLIENT_ID = 
TEAMSFX_API_CLIENT_SECRET = 
AAD_APP_OAUTH_AUTHORITY_HOST = 
// Scenario 2
TEAMSFX_API_TENANT_ID =
TEAMSFX_API_CLIENT_ID =
TEAMSFX_API_CLIENT_SECRET =
```
</details>
<details>
<summary><b>API Key
</b></summary>

Append your Api connection configuration to `./env/.env.local`
```
...
// set up environment variables required by teamsfx
TEAMSFX_API_ENDPOINT =
TEAMSFX_API_API_KEY =
```
</details>
<details>
<summary><b>Custom Auth Implementation
</b></summary>

Append your Api connection configuration to `env/.env.local`
```
...
// set up environment variables required by teamsfx
TEAMSFX_API_ENDPOINT=
```
</details>


## Step 4: Add Configuration for Azure
Before provision, make sure the project has **Azure Function** or **Bot Service** to host your API connection. Then add the `apiClient` to the correspoing service according to your design.

If hosting in the **Azure Function**, please append following appSettings to `infra/azure.bicep`

If hosting in the **Bot Service**, please append following appSettings to `infra/botRegistration/azurebot.bicep`

<details>
<summary><b>Basic Auth
</b></summary>

- Host in the **Azure Function**, append following values to `infra/azure.bicep`
```bicep
...
// Azure Functions that hosts your function code
resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
  ...
      appSettings: [
        {
          name: 'TEAMSFX_API_ENDPOINT',
          value: ''
        }
        {
          name: 'TEAMSFX_API_USERNAME',
          value: ''
        }
        {
          name: 'TEAMSFX_API_USERNAME',
          value: ''
        }
        ...
```

- Host in the **Bot Service**, append following values to `infra/botRegistration/azurebot.bicep`
```bicep
...
// Register your web service as a bot with the Bot Framework
resource botService 'Microsoft.BotService/botServices@2021-03-01' = {
  ...
  properties: {
    TEAMSFX_API_ENDPOINT: ''
    TEAMSFX_API_USERNAME: ''
    TEAMSFX_API_USERNAME: ''
    ...
  }
```
</details>
<details>
<summary><b>Certification
</b></summary>

- Host in the **Azure Function**, append following values to `infra/azure.bicep`
```bicep
...
// Azure Functions that hosts your function code
resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
  ...
      appSettings: [
        {
          name: 'TEAMSFX_API_ENDPOINT',
          value: ''
        }
        ...
```

- Host in the **Bot Service**, append following values to `infra/botRegistration/azurebot.bicep`
```bicep
...
// Register your web service as a bot with the Bot Framework
resource botService 'Microsoft.BotService/botServices@2021-03-01' = {
  ...
  properties: {
    TEAMSFX_API_ENDPOINT: ''
    ...
  }
```
</details>
<details>
<summary><b>Azure Active Directory
</b></summary>

- Host in the **Azure Function**, append following values to `infra/azure.bicep`
```bicep
...
// Azure Functions that hosts your function code
resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
  ...
      appSettings: [
        {
          name: 'TEAMSFX_API_ENDPOINT',
          value: ''
        }
        // Scenario 1
        {
          name: 'TEAMSFX_API_TENANT_ID',
          value: ''
        }
        {
          name: 'TEAMSFX_API_CLIENT_ID',
          value: ''
        }
        {
          name: 'TEAMSFX_API_CLIENT_SECRET',
          value: ''
        }
        {
          name: 'AAD_APP_OAUTH_AUTHORITY_HOST',
          value: ''
        }

        // Scenario 2
        {
          name: 'TEAMSFX_API_TENANT_ID',
          value: ''
        }
        {
          name: 'TEAMSFX_API_CLIENT_ID'
          value: '' 
        }
        {
          name: 'TEAMSFX_API_CLIENT_SECRET',
          value: ''
        }
        ...
```

- Host in the **Bot Service**, append following values to `infra/botRegistration/azurebot.bicep`
```bicep
...
// Register your web service as a bot with the Bot Framework
resource botService 'Microsoft.BotService/botServices@2021-03-01' = {
  ...
  properties: {
    TEAMSFX_API_ENDPOINT: ''
    // Scenario 1
    TEAMSFX_API_TENANT_ID = 
    TEAMSFX_API_CLIENT_ID = 
    TEAMSFX_API_CLIENT_SECRET = 
    AAD_APP_OAUTH_AUTHORITY_HOST = 
    // Scenario 2
    TEAMSFX_API_TENANT_ID =
    TEAMSFX_API_CLIENT_ID =
    TEAMSFX_API_CLIENT_SECRET =
    ...
  }
```
</details>
<details>
<summary><b>API Key
</b></summary>

- Host in the **Azure Function**, append following values to `infra/azure.bicep`
```bicep
...
// Azure Functions that hosts your function code
resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
  ...
      appSettings: [
        {
          name: 'TEAMSFX_API_ENDPOINT',
          value: ''
        }
        {
          name: 'TEAMSFX_API_API_KEY',
          value: ''
        }
        ...
```

- Host in the **Bot Service**, append following values to `infra/botRegistration/azurebot.bicep`
```bicep
...
// Register your web service as a bot with the Bot Framework
resource botService 'Microsoft.BotService/botServices@2021-03-01' = {
  ...
  properties: {
    TEAMSFX_API_ENDPOINT: ''
    TEAMSFX_API_API_KEY: ''
    ...
  }
```
</details>
<details>
<summary><b>Custom Auth Implementation
</b></summary>

- Host in the **Azure Function**, append following values to `infra/azure.bicep`
```bicep
...
// Azure Functions that hosts your function code
resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
  ...
      appSettings: [
        {
          name: 'TEAMSFX_API_ENDPOINT',
          value: ''
        }
        ...
```

- Host in the **Bot Service**, append following values to `infra/botRegistration/azurebot.bicep`
```bicep
...
// Register your web service as a bot with the Bot Framework
resource botService 'Microsoft.BotService/botServices@2021-03-01' = {
  ...
  properties: {
    TEAMSFX_API_ENDPOINT: ''
    ...
  }
```
</details>
# Consent behaviours with knownClientApplications and preAuthorizedApplications attributes.

 1. [Overview](#overview)
 1. [Scenario](#scenario)
 1. [Contents](#contents)
 1. [Prerequisites](#prerequisites)
 1. [Setup](#setup)
 1. [Registration](#registration)
 1. [Running the sample](#running-the-sample)
 1. [Explore the sample](#explore-the-sample)
 1. [Consent Behaviour](#consent-behaviour)
 1. [More information](#more-information)
 1. [Community Help and Support](#community-help-and-support)
 1. [Contributing](#contributing)

## Overview

This sample consists of following projects.
- Front end Angular single-page application
- .NET core API-1
- .NET core API-2

(This sample is taken from the GitHub repo https://github.com/Azure-Samples/ms-identity-javascript-angular-tutorial/blob/main/7-AdvancedScenarios/1-call-api-obo/README.md and customized for current demostration)

Front end Angular app  lets a user authenticate and obtain an access token to call an ASP.NET Core web API 1 and ASP.NET Core web API 2, protected by [Azure Active Directory (Azure AD)](https://azure.microsoft.com/services/active-directory/). These two web APIs then calls the [Microsoft Graph API](https://developer.microsoft.com/graph) using the [OAuth 2.0 on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow). The web API's call to Microsoft Graph is made using the [Microsoft Graph SDK](https://docs.microsoft.com/graph/sdks/sdks-overview).Additionaly, Front end app has API permission to Azure batch service resource, API-1 has a API permission to Azure devops resource and API-2 has API permission to Azure Storage resource. The purpose of this repo is to demostrate the different consent behaviour with knownclient and preauthorizedclient attribute.If you are intrested to understand how code works or if you want explore how OBO flow work then please refer the sample https://github.com/Azure-Samples/ms-identity-javascript-angular-tutorial/blob/main/7-AdvancedScenarios/1-call-api-obo/README.md

## Scenario

- The sample implements an **onboarding** scenario where a profile is created for a new user whose fields are pre-populated by the available information about the user on Microsoft Graph.
- The **ProfileSPA** uses [MSAL Angular](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angular) to authenticate a user.
- Once the user authenticates, **ProfileSPA** obtains an [access token](https://docs.microsoft.com/azure/active-directory/develop/access-tokens) from Azure AD.
- The access token is then used to authorize the **ProfileAPI-1** to call MS Graph API **on user's behalf**. In order to call MS Graph API, **ProfileAP1-1** uses the [Microsoft Graph SDK](https://docs.microsoft.com/graph/sdks/sdks-overview).
- The access token is then used to authorize the **ProfileAPI-2** to call MS Graph API **on user's behalf**. In order to call MS Graph API, **ProfileAPI-2** uses the [Microsoft Graph SDK](https://docs.microsoft.com/graph/sdks/sdks-overview).
- To protect its endpoint and accept only the authorized calls, the ProfileAPI1 & ProfileAPI2 uses [Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web).

## Contents

| File/folder                         | Description                                                |
|-------------------------------------|------------------------------------------------------------|
| `ProfileSPA/src/app/auth-config.ts`        | Authentication parameters for SPA project reside here.     |
| `ProfileSPA/src/app/app.module.ts`         | MSAL Angular is initialized here.                          |
| `ProfileAPI-1/ProfileAPI-1/appsettings.json`   | Authentication parameters for API-1 project reside here.     |
| `ProfileAPI-1/ProfileAPI-1/Startup.cs`         | Microsoft.Identity.Web is initialized here.                |
| `ProfileAPI-2/ProfileAPI-2/appsettings.json`   | Authentication parameters for API-2 project reside here.     |
| `ProfileAPI-2/ProfileAPI-2/Startup.cs`         | Microsoft.Identity.Web is initialized here.                |


## Prerequisites

- An **Azure AD** tenant. For more information see: [How to get an Azure AD tenant](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant)
- A user account in your **Azure AD** tenant. This sample will not work with a **personal Microsoft account**. Therefore, if you signed in to the [Azure portal](https://portal.azure.com) with a personal account and have never created a user account in your directory before, you need to do that now.

## Setup

### Step 1. Clone or download this repository

```console
    git clone <<current repo path.git>
```

or download and extract the repository .zip file.

> :warning: To avoid path length limitations on Windows, we recommend cloning into a directory near the root of your drive.

### Step 2. Install .NET Core API dependencies

```console
    cd ProfileAPI-1/ProfileAPI-1
    dotnet restore
    cd ProfileAPI-2/ProfileAPI-2
    dotnet restore
```

### Step 3. Trust development certificates

```console
    dotnet dev-certs https --clean
    dotnet dev-certs https --trust
```

For more information and potential issues, see: [HTTPS in .NET Core](https://docs.microsoft.com/aspnet/core/security/enforcing-ssl).

### Step 4. Install Angular SPA dependencies

```console
    cd ../../
    cd ProfileSPA
    npm install
```

### Registration

There are three projects in this sample. Each needs to be separately registered in your Azure AD tenant. To register these projects, you can:

- follow the steps below for manually register your apps

### Choose the Azure AD tenant where you want to create your applications

As a first step you'll need to:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. If your account is present in more than one Azure AD tenant, select your profile at the top right corner in the menu on top of the page, and then **switch directory** to change your portal session to the desired Azure AD tenant.

### Register the API 1 (ProfileAPI-1)

1. Navigate to the [Azure portal](https://portal.azure.com) and select the **Azure AD** service.
1. Select the **App Registrations** blade on the left, then select **New registration**.
1. In the **Register an application page** that appears, enter your application's registration information:
   - In the **Name** section, enter a meaningful application name that will be displayed to users of the app, for example `ProfileAPI-1`.
   - Under **Supported account types**, select **Accounts in this organizational directory only**.
1. Select **Register** to create the application.
1. In the app's registration screen, find and note the **Application (client) ID**. You use this value in your app's configuration file(s) later in your code.
1. Select **Save** to save your changes.
1. In the app's registration screen, select the **Certificates & secrets** blade in the left to open the page where we can generate secrets and upload certificates.
1. In the **Client secrets** section, select **New client secret**:
   - Type a key description (for instance `app secret`),
   - Select one of the available key durations (**In 1 year**, **In 2 years**, or **Never Expires**) as per your security posture.
   - The generated key value will be displayed when you select the **Add** button. Copy the generated value for use in the steps later.
   - You'll need this key later in your code's configuration files. This key value will not be displayed again, and is not retrievable by any other means, so make sure to note it from the Azure portal before navigating to any other screen or blade.
1. In the app's registration screen, select the **API permissions** blade in the left to open the page where we add access to the APIs that your application needs.
   - Select the **Add a permission** button and then,
   - Ensure that the **Microsoft APIs** tab is selected.
   - In the *Commonly used Microsoft APIs* section, select **Microsoft Graph**
   - In the **Delegated permissions** section, select the **User.Read**, **offline_access** & **Files.Read.All** in the list. Use the search box if necessary.
   - Select the **Add permissions** button at the bottom.
1. In the app's registration screen, select the **API permissions** blade in the left to open the page where we add access to the APIs that your application needs.
   - Select the **Add a permission** button and then,
   - Ensure that the **Microsoft APIs** tab is selected.
   - Select *Azure Devops* section.
   - In the **Delegated permissions** section, select  **user_impersonation**. Use the search box if necessary.
   - Select the **Add permissions** button at the bottom.
1. In the app's registration screen, select the **Expose an API** blade to the left to open the page where you can declare the parameters to expose this app as an API for which client applications can obtain [access tokens](https://docs.microsoft.com/azure/active-directory/develop/access-tokens) for.
The first thing that we need to do is to declare the unique [resource](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow) URI that the clients will be using to obtain access tokens for this Api. To declare an resource URI, follow the following steps:
   - Select `Set` next to the **Application ID URI** to generate a URI that is unique for this app.
   - For this sample, accept the proposed Application ID URI (`api://{clientId}`) by selecting **Save**.
1. All APIs have to publish a minimum of one [scope](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow#request-an-authorization-code) for the client's to obtain an access token successfully. To publish a scope, follow the following steps:
   - Select **Add a scope** button open the **Add a scope** screen and Enter the values as indicated below:
    - For **Scope name**, use `access_ProfileAPI_1`.
    - Select **Admins and users** options for **Who can consent?**.
    - For **Admin consent display name** type `Access for user to call access_ProfileAPI_1 `.
    - For **Admin consent description** type `Allows the app to invoke access_ProfileAPI_1 which internally needs  1)Read all files that user can access 2) Full access to VSTS REST APIs 3) offline access to graph.`
    - For **User consent display name** type `Access for you to call access_ProfileAPI_1 permissions `.
    - For **User consent description** type `Allows the app to invoke access_ProfileAPI_1 which internally needs 1)Read all files that you have access 2) Full access to VSTS REST APIs 3) offline access to graph`
    - Keep **State** as **Enabled**.
    - Select the **Add scope** button on the bottom to save this scope.
   - Select **Add a scope** button open the **Add a scope** screen and Enter the values as indicated below:
    - For **Scope name**, use `access_ProfileAPI_1_withoutpreauthclient`.
    - Select **Admins and users** options for **Who can consent?**.
    - For **Admin consent display name** type `Access for user to call access_ProfileAPI_1_withoutpreauthclient permissions`.
    - For **Admin consent description** type `Allows the app to invoke access_ProfileAPI_1_withoutpreauthclient which internally needs  1)Read all files that you have access 2) Full access to VSTS REST APIs 3) offline access to graph`
    - For **User consent display name** type `Access for you to call access_ProfileAPI_1_withoutpreauthclient permissions `.
    - For **User consent description** type `Allows the app to invoke access_ProfileAPI_1_withoutpreauthclient which internally needs  1)Read all files that you have access 2) Full access to VSTS REST APIs 3) offline access to graph`
     - Keep **State** as **Enabled**.
     - Select the **Add scope** button on the bottom to save this scope.

#### Configure the service app (ProfileAPI-1) to use your app registration

Open the project in your IDE (like Visual Studio or Visual Studio Code) to configure the code.

> In the steps below, "ClientID" is the same as "Application ID" or "AppId".

1. Open the `ProfileAPI-1\ProfileAPI-1\appsettings.json` file.
1. Find the key `Domain` and replace the existing value with your Azure AD tenant name.
1. Find the key `ClientId` and replace the existing value with the application ID (clientId) of `ProfileAPI-1` app copied from the Azure portal.
1. Find the key `ClientSecret` and replace the existing value with the key you saved during the creation of `ProfileAPI-1` copied from the Azure portal.
1. Find the key `TenantId` and replace the existing value with your Azure AD tenant ID.

1. Open the `Controllers\ProfileController.cs` file.
1. Find the variable `scopeRequiredByApi` and replace its value with the name of the API scope that you have just exposed (by default `access_ProfileAPI_1`).

### Register the API 2 (ProfileAPI-2)

1. Navigate to the [Azure portal](https://portal.azure.com) and select the **Azure AD** service.
1. Select the **App Registrations** blade on the left, then select **New registration**.
1. In the **Register an application page** that appears, enter your application's registration information:
   - In the **Name** section, enter a meaningful application name that will be displayed to users of the app, for example `ProfileAPI-2`.
   - Under **Supported account types**, select **Accounts in this organizational directory only**.
1. Select **Register** to create the application.
1. In the app's registration screen, find and note the **Application (client) ID**. You use this value in your app's configuration file(s) later in your code.
1. Select **Save** to save your changes.
1. In the app's registration screen, select the **Certificates & secrets** blade in the left to open the page where we can generate secrets and upload certificates.
1. In the **Client secrets** section, select **New client secret**:
   - Type a key description (for instance `app secret`),
   - Select one of the available key durations (**In 1 year**, **In 2 years**, or **Never Expires**) as per your security posture.
   - The generated key value will be displayed when you select the **Add** button. Copy the generated value for use in the steps later.
   - You'll need this key later in your code's configuration files. This key value will not be displayed again, and is not retrievable by any other means, so make sure to note it from the Azure portal before navigating to any other screen or blade.
1. In the app's registration screen, select the **API permissions** blade in the left to open the page where we add access to the APIs that your application needs.
   - Select the **Add a permission** button and then,
   - Ensure that the **Microsoft APIs** tab is selected.
   - In the *Commonly used Microsoft APIs* section, select **Microsoft Graph**
   - In the **Delegated permissions** section, select the **User.Read**, **offline_access** & **Calendars.Read** in the list. Use the search box if necessary.
   - Select the **Add permissions** button at the bottom.
1. In the app's registration screen, select the **API permissions** blade in the left to open the page where we add access to the APIs that your application needs.
   - Select the **Add a permission** button and then,
   - Ensure that the **Microsoft APIs** tab is selected.
   - Select *Azure Storage* section.
   - In the **Delegated permissions** section, select  **user_impersonation**. Use the search box if necessary.
   - Select the **Add permissions** button at the bottom.
1. In the app's registration screen, select the **Expose an API** blade to the left to open the page where you can declare the parameters to expose this app as an API for which client applications can obtain [access tokens](https://docs.microsoft.com/azure/active-directory/develop/access-tokens) for.
The first thing that we need to do is to declare the unique [resource](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow) URI that the clients will be using to obtain access tokens for this Api. To declare an resource URI, follow the following steps:
   - Select `Set` next to the **Application ID URI** to generate a URI that is unique for this app.
   - For this sample, accept the proposed Application ID URI (`api://{clientId}`) by selecting **Save**.
1. All APIs have to publish a minimum of one [scope](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow#request-an-authorization-code) for the client's to obtain an access token successfully. To publish a scope, follow the following steps:
   - Select **Add a scope** button open the **Add a scope** screen and Enter the values as indicated below:
    - For **Scope name**, use `access_ProfileAPI_2`.
    - Select **Admins and users** options for **Who can consent?**.
    - For **Admin consent display name** type `Access for user to call access_ProfileAPI_2 permissions`.
    - For **Admin consent description** type `Allows the app to invoke access_ProfileAPI_2 which internally needs 1)Read user calendars 2) Full access to Azure Storage 3) offline access to graph`
    - For **User consent display name** type `Access for you to call access_ProfileAPI_2 permissions `.
    - For **User consent description** type `Allows the app to invoke access_ProfileAPI_2 which internally needs 1)Read user calendars 2) Full access to Azure Storage 3) offline access to graph`
     - Keep **State** as **Enabled**.
     - Select the **Add scope** button on the bottom to save this scope.
        
#### Configure the service app (ProfileAPI-2) to use your app registration

Open the project in your IDE (like Visual Studio or Visual Studio Code) to configure the code.

> In the steps below, "ClientID" is the same as "Application ID" or "AppId".

1. Open the `ProfileAPI-2\ProfileAPI-2\appsettings.json` file.
1. Find the key `Domain` and replace the existing value with your Azure AD tenant name.
1. Find the key `ClientId` and replace the existing value with the application ID (clientId) of `ProfileAPI-2` app copied from the Azure portal.
1. Find the key `ClientSecret` and replace the existing value with the key you saved during the creation of `ProfileAPI-2` copied from the Azure portal.
1. Find the key `TenantId` and replace the existing value with your Azure AD tenant ID.

1. Open the `Controllers\ProfileController.cs` file.
1. Find the variable `scopeRequiredByApi` and replace its value with the name of the API scope that you have just exposed (by default `access_ProfileAPI_2`).

### Register the client app (ProfileSPA)

1. Navigate to the [Azure portal](https://portal.azure.com) and select the **Azure AD** service.
1. Select the **App Registrations** blade on the left, then select **New registration**.
1. In the **Register an application page** that appears, enter your application's registration information:
   - In the **Name** section, enter a meaningful application name that will be displayed to users of the app, for example `ProfileSPA`.
   - Under **Supported account types**, select **Accounts in this organizational directory only**.
   - In the **Redirect URI (optional)** section, select **Single-page application** in the combo-box and enter the following redirect URI: `http://localhost:4200`.
1. Select **Register** to create the application.
1. In the app's registration screen, find and note the **Application (client) ID**. You use this value in your app's configuration file(s) later in your code.
1. Select **Save** to save your changes.
1. In the app's registration screen, select the **API permissions** blade in the left to open the page where we add access to the APIs that your application needs.
   - Select the **Add a permission** button and then:
    - Ensure that the **Microsoft APIs** tab is selected.
    - In the *Commonly used Microsoft APIs* section, select **Microsoft Graph**
    - In the **Delegated permissions** section, select the **User.Read** & **Device.Read** in the list. Use the search box if necessary.
    - Select the **Add permissions** button at the bottom.
    - Select the **Add a permission** button and then,
     - Ensure that the **Microsoft APIs** tab is selected.
     - Select *Azure Batch* section.
     - In the **Delegated permissions** section, select  **user_impersonation**. Use the search box if necessary.
     - Select the **Add permissions** button at the bottom.
   - Select the **Add a permission** button and then:
    - Ensure that the **My APIs** tab is selected.
    - In the list of APIs, select the API `ProfileAPI-1`.
    - In the **Delegated permissions** section, select the **access_ProfileAPI_1** & **access_ProfileAPI_1_withoutpreauthclient**  in the list. Use the search box if necessary.
     - Select the **Add permissions** button at the bottom.
   - Select the **Add a permission** button and then:
    - Ensure that the **My APIs** tab is selected.
    - In the list of APIs, select the API `ProfileAPI-2`.
    - In the **Delegated permissions** section, select the **access_ProfileAPI_2** in the list. Use the search box if necessary.
    - Select the **Add permissions** button at the bottom.
     
#### Configure the client app (ProfileSPA) to use your app registration

Open the project in your IDE (like Visual Studio or Visual Studio Code) to configure the code.

> In the steps below, "ClientID" is the same as "Application ID" or "AppId".

1. Open the `ProfileSPA\src\app\auth-config.ts` file.
1. Find the key `Enter_the_Application_Id_Here` and replace the existing value with the application ID (clientId) of `ProfileSPA` app copied from the Azure portal.
1. Find the key `Enter_the_Tenant_Info_Here` and replace the existing value with your Azure AD tenant ID.

## Running the sample

Using a command line interface such as VS Code integrated terminal, locate the application directory. Then:  

```console
    cd SPA
    npm start
```

In a separate console window, execute the following commands:

```console
    cd ProfileAPI-1
    dotnet run
    cd ProfileAPI-2
    dotnet run
```

## Explore the sample

1. Open your browser and navigate to `http://localhost:4200`.
2. Sign-in using the button on top-right corner.
3. If this is your first sign-in, you will be prompted with a consent prompt. 
![Screenshot](./ReadmeFiles/screenshot.png)

> :information_source: Did the sample not work for you as expected? Then please reach out to us using the [GitHub Issues](../../../../issues) page.

## Consent Behaviour

Consent behaviour depends on the scope that has been sent while making the call to authorize endpoint. Please follow the below steps to send different scopes from the ProfileSPA app so that, you can observe different consent behaviours.

- Open the `ProfileSPA\src\app\auth-config.ts` file.
- Find the key `Enter_the_Application_Id_of_Service_Here` in the and replace the existing value with the application ID (clientId). This will be passed as scope parameter during the login flow. 

### 1. Without preAuthorizedApplications & without knownClientApplications attributes ###

#### Specific scope scenarios ####

##### Scenario 1: Request the scope for GRAPH resource #####

   1. Pass client id of `Graph API` with `Device.Read` scope. 
      - Scope value in the code `scopes: ["https://graph.microsoft.com/Device.Read"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://graph.microsoft.com/Device.Read openid profile offline_access`

##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-1.1.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `View your list of devices` is shown in the consent promt since ProfileSPA has  **Delegated API permissions** for **Device.Read** for **Graph** resource and scope parameter to authorize endpoint contains `Device.read`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.

##### Scenario 2: Request the scope for Azure Batch resource #####

   1. Pass client id of `Azure Batch Service` with `user_impersonation` scope. 
     - Scope value in the code `scopes: ["https://batch.core.windows.net/user_impersonation"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://batch.core.windows.net/user_impersonation openid profile offline_access` 
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-1.2.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Full access to Azure Batch Service API` is shown in the consent promt since ProfileSPA has  **Delegated API permissions** to **user_impersonation** scope for **AzureBatch service** and scope parameter to authorize endpoint contains `https://batch.core.windows.net/user_impersonation`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.

##### Scenario 3: Request the scope for ProfileAPI-1 resource #####

   1. Pass client id of `ProfileAPI-1` with `access_ProfileAPI_1` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/access_ProfileAPI_1"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/access_ProfileAPI_1 openid profile offline_access`
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-1.3.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Access for you to call access_ProfileAPI_1 permissions (ProfileAPI-2)` is shown in the consent prompt since ProfileSPA has  **Delegated API permissions** to **access_ProfileAPI_1** scope for **ProfileAPI-1** and scope parameter to authorize endpoint contains `api://app_id_of_Profile_API_1/access_ProfileAPI_1`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.

##### Scenario 4: Request the scope for ProfileAPI-2 resource #####

   1. Pass client id of `ProfileAPI-2` with `access_ProfileAPI_2` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_2/access_ProfileAPI_2"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_2/access_ProfileAPI_2 openid profile offline_access`
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-1.4.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Access for you to call access_ProfileAPI_2 permissions (ProfileAPI-2)` is shown in the consent prompt since ProfileSPA has  **Delegated API permissions** to **access_ProfileAPI_2** scope for **ProfileAPI-2** and scope parameter to authorize endpoint contains `api://app_id_of_Profile_API_2/access_ProfileAPI_2`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.

#### Scenario 5: .default scope ####

##### You will observe a same consent prompt, for the below scope values being passed. #####

  1. Pass client id of `Profile API 1` with default scope. 
      - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/.default openid profile offline_access`
  1. Pass client id of `Profile API 2` with default scope. 
      - Scope value in the code `scopes: ["api://app_id_of_Profile_API_2/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_2/.default openid profile offline_access` 
  1. Pass client id of `Profile SPA` with default scope. 
      - Scope value in the code `scopes: ["app_id_of_Profile_SPA/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=app_id_of_Profile_SPA/.default openid profile offline_access`
  1. Pass client id of `Graph API` with default scope. 
     - Scope value in the code `scopes: ["https://graph.microsoft.com/.default"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `https://graph.microsoft.com/.default openid profile offline_access`
  1. Pass client id of `Azure Batch Service` with default scope. 
      - Scope value in the code `scopes: ["https://batch.core.windows.net/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://batch.core.windows.net/.default openid profile offline_access`
  1. Just pass `.default` with default scope. 
      - Scope value in the code `scopes: [".default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=.default openid profile offline_access`

##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-1.5.jpg)

##### Explanation for the consents that are shown in the prompt #####

 - The permissions in the black rectangle of the screenshot (`View your list of devices` & `Sign you in and read your profile`) in the consent prompt are shown since ProfileSPA has **Delegated API permissions** for **User.Read** & **Device.Read** from GRAPH resource. Please note that, **User.Read** scope combines the **profile** scope with it and hence you will not see a separte consent for `View your basic profile`
 - The permission in the red rectangle of the screenshot (`Access for you to call access_ProfileAPI_2 permissions (ProfileAPI-2)`) in the consent prompt, is shown since ProfileSPA has **Delegated API permissions** added to **access_ProfileAPI_2** for ProfileAPI-2 resource.  
 - The permission in the blue rectangle of the screenshot (`Full access to Azure Batch Service API`) in the consent prompt, is shown since ProfileSPA has **Delegated API permissions** added to **user_impersonation** for AzureBatch resource.
 - The permissions in the green rectangle of the screenshot (`Access for you to call access_ProfileAPI_1_withoutpreauthclient permissions (ProfileAPI-1)` & `Access for you to call access_ProfileAPI_1 permissions (ProfileAPI-1)`) in the consent prompt are shown since ProfileSPA has **Delegated API permissions** for **access_ProfileAPI_1** & **access_ProfileAPI_1_withoutpreauthclient** for ProfileAPI-1 resource.  
 
##### Conclusion: If scope parameter containes `.default` in it, then Azure AD prompts the consent for all the API permissions that are added in the app registration of the requester(In this case, front end SPA UI).The use of “.default” (e.g. “https://graph.microsoft.com/.default”, “https://middle-tier.example.com/.default”, or even just “.default”) instead of a specific delegated permission will trigger the combined consent prompt. ##### 

### 2. With preAuthorizedApplications & without knownClientApplications attributes ###

Visit the app regsitration page of **ProfileAPI-1** and navigate to **Expose an API** blade. There is an option to add an **AuthorizedClient application**, please click on the `Add a client application` and then copy the client id of **ProfileSPA** and paste it in the cliend id textbox on the page. Be sure to select the API **access_ProfileAPI_1**.Authorizing a client application indicates that this API trusts the application and users should not be asked to consent when the client calls this API. This setting is specific to API rather than application. We have added authorixed client only to **access_ProfileAPI_1** and not to **access_ProfileAPI_1_withoutpreauthclient**. 

#### Specific scopes scenarios ####

##### Scenario 1: Request the scope for GRAPH resource #####

   1. Pass client id of `Graph API` with `User.Read` scope. 
      - Scope value in the code `scopes: ["https://graph.microsoft.com/user.Read"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://graph.microsoft.com/User.Read openid profile offline_access`

##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-2.1.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Sign you in and read your profile` is shown in the consent promt since ProfileSPA has  **Delegated API permissions** for **User.Read** for **Graph** resource and scope parameter to authorize endpoint contains `User.read` & `profile`. Please note that, **User.Read** scope combines the **profile** scope along with it and hence you will not see a separte consent for `View your basic profile`
- The permission `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **offline.access** & **profile** are configured automatically for the apps signing into the applications in Azure AD as per the open id standards.

##### Scenario 2: Request the scope for Azure Batch resource #####

   1. Pass client id of `Azure Batch Service` with `user_impersonation` scope. 
     - Scope value in the code `scopes: ["https://batch.core.windows.net/user_impersonation"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://batch.core.windows.net/user_impersonation openid profile offline_access` 
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-2.2.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Full access to Azure Batch Service API` is shown in the consent promt since ProfileSPA has  **Delegated API permissions** to **user_impersonation** scope for **AzureBatch service** and scope parameter to authorize endpoint contains `https://batch.core.windows.net/user_impersonation`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.

##### Scenario 3: Request the scope for ProfileAPI-1 resource which is tagged against the preauthorizedclient attribute #####

   1. Pass client id of `ProfileAPI-1` with `access_ProfileAPI_1` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/access_ProfileAPI_1"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/access_ProfileAPI_1 openid profile offline_access`

##### Consent prompt screenshot #####

No consent prompt !!!!

##### Explanation for no consent prompt in this scenario #####

Since we added a client id of ProfileSPA against the API **access_ProfileAPI_1**  in the ProfileAPI-1, there is no consent prompt shown. Additionaly, an accesstoken is granted with the scope `"scp": "access_ProfileAPI_1"` after the SPA hit the token endpoint without any user consent being shown.

##### Scenario 4: Request the scope for ProfileAPI-1 resource  which is NOT tagged against the preauthorizedclient attribute  #####

   1. Pass client id of `ProfileAPI-1` with `access_ProfileAPI_1_withoutpreauthclient` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient openid profile offline_access`
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-2.4.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Access for you to call access_ProfileAPI_1_withoutpreauthclient permissions (ProfileAPI-1)` is shown in the consent prompt since ProfileSPA has  **Delegated API permissions** to **access_ProfileAPI_1_withoutpreauthclient** scope for **ProfileAPI-1** and scope parameter to authorize endpoint contains `api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.
- 
##### Scenario 5: Request the scope for ProfileAPI-2 resource #####

   1. Pass client id of `ProfileAPI-2` with `access_ProfileAPI_2` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_2/access_ProfileAPI_2"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_2/access_ProfileAPI_2 openid profile offline_access`
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-2.5.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Access for you to call access_ProfileAPI_2 permissions (ProfileAPI-2)` is shown in the consent prompt since ProfileSPA has  **Delegated API permissions** to **access_ProfileAPI_2** scope for **ProfileAPI-2** and scope parameter to authorize endpoint contains `api://app_id_of_Profile_API_2/access_ProfileAPI_2`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to`) shown in the consent prompt since scope parameter to authorize endpoint has `openid` `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope(To which scope is not sent in the request), **offline.access** **profile** are configured automatically for the apps signing into the applications in Azure AD.

#### .default scope scenarios ####

##### Sceanrio 6: #####
 
 You will observe a same consent prompt, for the below scope values being passed.
 
  1. Pass client id of `Profile API 2` with default scope. 
      - Scope value in the code `scopes: ["api://app_id_of_Profile_API_2/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_2/.default openid profile offline_access` 
  1. Pass client id of `Profile SPA` with default scope. 
      - Scope value in the code `scopes: ["app_id_of_Profile_SPA/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=app_id_of_Profile_SPA/.default openid profile offline_access`
   1. Pass client id of `Graph API` with default scope. 
     - Scope value in the code `scopes: ["https://graph.microsoft.com/.default"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `https://graph.microsoft.com/.default openid profile offline_access`
   1. Pass client id of `Azure Batch Service` with default scope. 
      - Scope value in the code `scopes: ["https://batch.core.windows.net/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=.default openid profile offline_access`
   1. Just pass `.default` with default scope. 
      - Scope value in the code `scopes: [".default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=.default openid profile offline_access`

##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-2.6.jpg)

##### Explanation for the consents that are shown in the prompt #####

 - The permissions in the black rectangle of the screenshot (`View your list of devices` & `Sign you in and read your profile`) in the consent prompt are shown since ProfileSPA has **Delegated API permissions** for **User.Read** & **Device.Read** from GRAPH resource.Please note that, **User.Read** scope combines the **profile** scope with it and hence you will not see a separte consent for `View your basic profile`
 - The permission in the red rectangle of the screenshot (`Access for you to call access_ProfileAPI_2 permissions (ProfileAPI-2)`) in the consent prompt, is shown since ProfileSPA has **Delegated API permissions** added to **access_ProfileAPI_2** from ProfileAPI-2 resource.  
 - The permission in the blue rectangle of the screenshot (`Full access to Azure Batch Service API`) in the consent prompt, is shown since ProfileSPA has **Delegated API permissions** added to **user_impersonation** from AzureBatch resource.
 - The permissions in the green rectangle of the screenshot (`Access for you to call access_ProfileAPI_1_withoutpreauthclient permissions (ProfileAPI-1)` & `Access for you to call access_ProfileAPI_1 permissions (ProfileAPI-1)`) in the consent prompt are shown since ProfileSPA has **Delegated API permissions** for **access_ProfileAPI_1** & **access_ProfileAPI_1_withoutpreauthclient** from ProfileAPI-1 resource.  

##### Scenario 7: With appending .default scope to the app id of the API in which preauthorizedclient attribute is configured  #####

1. Pass client id of `Profile API 1` with default scope. 
      - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/.default openid profile offline_access`

##### Consent prompt screenshot #####

No consent prompt !!!

##### Explanation for no consent prompt #####

We added a client id of ProfileSPA against the API **access_ProfileAPI_1**  in the ProfileAPI-1. While evaluating the .default for the ProfileAPI-1, AzureAd took consideration of the scope **access_ProfileAPI_1** against which preauthorizedclient attribute is added and hence there is no consent prompt shown. Additionaly, an accesstoken is granted with the scope `"scp": "access_ProfileAPI_1"` without user having to consent it.

##### Learning from the above test is that, if there is preauthorizedclient attribute added in the API for the client app and then if .default scope is passed appending with the app id of the API, then consent prompt is not shown. Be aware that, this behaviour may lead to confusion as issued access token may not have scopes for the resource requested for(.default is not honoured). For example scope for `access_ProfileAPI_1_withoutpreauthclient` is missing in the accesstoken. ##### 

### 3. With knownClientApplications attribute ###

#### Configure Known Client Applicationn for service (ProfileAPI-2)

In all the above scenarios, none of the scopes that are added in the API didn't come in the picture. For a middle-tier web API (`ProfileAPI-1` & `ProfileAPI-2`) to be able to call a downstream web API, the middle-tier app needs to be granted the required permissions as well. However, since the middle-tier cannot interact with the signed-in user, it needs to be explicitly bound to the client app in its **Azure AD** registration. This binding merges the permissions required by both the client and the middle tier Web Api and presents it to the end user in a single consent dialog. The user then consent to this combined set of permissions.

To achieve this, you need to add the **Application Id** of the client app, in the Manifest of the web API in the `knownClientApplications` property. Here's how:

1. In the [Azure portal](https://portal.azure.com), navigate to your `ProfileAPI-2` app registration, and select **Manifest** section.
1. In the manifest editor, change the `"knownClientApplications": []` line so that the array contains the Client ID of the client application (`ProfileSPA`) as an element of the array.

For instance:

   ```json
   "knownClientApplications": ["ca8dca8d-f828-4f08-82f5-325e1a1c6428"],
   ```

1. **Save** the changes to the manifest.

### Gaining consent for the middle-tier web API

The middle-tier application adds the client to the `knownClientApplications` list in its manifest, and then the client app can trigger a combined consent flow for both itself and the middle-tier application. On the Microsoft identity platform, this is done using the `/.default` scope. When triggering a consent screen using known client applications and `/.default`, the consent screen will show permissions for both the client to the middle-tier API, and also request whatever permissions are required by the middle-tier API. The user provides consent for both applications, and then the OBO flow works.

> :information_source: **KnownClientApplications** is an attribute in **application manifest**. It is used for bundling consent if you have a solution that contains two (or more) parts: a client app and a custom web API. If you enter the `appID` (clientID) of the client app into this array, the user will have to consent only once to the client app. Azure AD will know that consenting to the client means implicitly consenting to the web API. It will automatically provision service principals for both the client and web API at the same time. Both the client and the web API app must be registered in the same tenant.

#### Specific scopes sceanrios ####

##### Scenario 1: Request the scope for GRAPH resource #####

   1. Pass client id of `Graph API` with `User.Read` scope. 
      - Scope value in the code `scopes: ["https://graph.microsoft.com/user.Read"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://graph.microsoft.com/User.Read openid profile offline_access`

##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-3.1.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Sign you in and read your profile` is shown in the consent promt since ProfileSPA has  **Delegated API permissions** for **User.Read** for **Graph** resource and scope parameter to authorize endpoint contains `User.read` & `profile`. Please note that, **User.Read** scope combines the **profile** scope along with it and hence you will not see a separte consent for `View your basic profile`
- The permission `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **offline.access** & **profile** are configured automatically for the apps signing into the applications in Azure AD as per the open id standards.

##### Scenario 2: Request the scope for Azure Batch resource #####

   1. Pass client id of `Azure Batch Service` with `user_impersonation` scope. 
     - Scope value in the code `scopes: ["https://batch.core.windows.net/user_impersonation"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://batch.core.windows.net/user_impersonation openid profile offline_access` 
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-3.2.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Full access to Azure Batch Service API` is shown in the consent promt since ProfileSPA has  **Delegated API permissions** to **user_impersonation** scope for **AzureBatch service** and scope parameter to authorize endpoint contains `https://batch.core.windows.net/user_impersonation`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.

##### Scenario 3: Request the scope for ProfileAPI-1 resource which is tagged against the preauthorizedclient attribute #####

   1. Pass client id of `ProfileAPI-1` with `access_ProfileAPI_1` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/access_ProfileAPI_1"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/access_ProfileAPI_1 openid profile offline_access`

##### Consent prompt screenshot #####

No consent prompt !!!!

##### Explanation for no consent prompt in this scenario #####

Since we added a client id of ProfileSPA against the API **access_ProfileAPI_1**  in the ProfileAPI-1, there is no consent prompt shown. Additionaly, an accesstoken is granted with the scope `"scp": "access_ProfileAPI_1"` after the SPA hit the token endpoint without any user consent being shown.

##### Scenario 4: Request the scope for ProfileAPI-1 resource  which is NOT tagged against the preauthorizedclient attribute  #####

   1. Pass client id of `ProfileAPI-1` with `access_ProfileAPI_1_withoutpreauthclient` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient openid profile offline_access`
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-3.4.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Access for you to call access_ProfileAPI_1_withoutpreauthclient permissions (ProfileAPI-1)` is shown in the consent prompt since ProfileSPA has  **Delegated API permissions** to **access_ProfileAPI_1_withoutpreauthclient** scope for **ProfileAPI-1** and scope parameter to authorize endpoint contains `api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.
- 
##### Scenario 5: Request the scope for ProfileAPI-2 resource #####

   1. Pass client id of `ProfileAPI-2` with `access_ProfileAPI_2` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_2/access_ProfileAPI_2"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_2/access_ProfileAPI_2 openid profile offline_access`
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-3.5.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Access for you to call access_ProfileAPI_2 permissions (ProfileAPI-2)` is shown in the consent prompt since ProfileSPA has  **Delegated API permissions** to **access_ProfileAPI_2** scope for **ProfileAPI-2** and scope parameter to authorize endpoint contains `api://app_id_of_Profile_API_2/access_ProfileAPI_2`. 
- The permission `View your basic profile` & `Maintain access to data you have given it access to`) shown in the consent prompt since scope parameter to authorize endpoint has `openid` `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope(To which scope is not sent in the request), **offline.access** **profile** are configured automatically for the apps signing into the applications in Azure AD.

#### .default scope scenarios ####

##### Scenario 6: #####
 
 You will observe a same consent prompt, for the below scope values being passed.
 
  1. Pass client id of `Profile API 2` with default scope. 
      - Scope value in the code `scopes: ["api://app_id_of_Profile_API_2/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_2/.default openid profile offline_access` 
  1. Pass client id of `Profile SPA` with default scope. 
      - Scope value in the code `scopes: ["app_id_of_Profile_SPA/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=app_id_of_Profile_SPA/.default openid profile offline_access`
   1. Pass client id of `Graph API` with default scope. 
     - Scope value in the code `scopes: ["https://graph.microsoft.com/.default"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `https://graph.microsoft.com/.default openid profile offline_access`
   1. Pass client id of `Azure Batch Service` with default scope. 
      - Scope value in the code `scopes: ["https://batch.core.windows.net/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=.default openid profile offline_access`
   1. Just pass `.default` with default scope. 
      - Scope value in the code `scopes: [".default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=.default openid profile offline_access`

##### Explanation for the consents that are shown in the prompt #####

 - The permissions in the black rectangle of the screenshot (`View your list of devices` & `Sign you in and read your profile`) in the consent prompt are shown since ProfileSPA has **Delegated API permissions** for **User.Read** & **Device.Read** from GRAPH resource.  Plus, permission(`Full access to Azure Batch Service API`) is shown since, it also has **Delegated API permissions** added to **user_impersonation** from AzureBatch resource.
  - The permissions in the red rectangle of the screenshot (`Access for you to call access_ProfileAPI_1_withoutpreauthclient permissions (ProfileAPI-1)` & `Access for you to call access_ProfileAPI_1 permissions (ProfileAPI-1)`) in the consent prompt are shown since ProfileSPA has **Delegated API permissions** for **access_ProfileAPI_1** & **access_ProfileAPI_1_withoutpreauthclient** from ProfileAPI-1 resource.  
 - The permission in the blue rectangle of the screenshot (`Access Azure Storage As the Signed-in User`, `Maintain access to data you have given it access to` & `
Read your calendars`) in the consent prompt, is shown since ProfileAPI-2 has **Delegated API permissions** added to **Calendars.Read** & **offline_access** for GRAPH resource and **user_impersonation** scope for Azure storage resource. Please note that, ProfileSPA had an API permission **access_ProfileAPI_2** added for the resource ProfileAPI-2 and also, client id of the ProfileSPA is updated in the KnownclientApplications attribute of the ProfileAPI-2 manifest. As a result of which, consent agregated and combined the permissions which are added in the ProfileAPI-2. 

##### Scenario 7:  With appending .default scope to the app id of the API in which preauthorizedclient attribute is configured #####

1. Pass client id of `Profile API 1` with default scope. 
      - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/.default openid profile offline_access`

##### Consent prompt screenshot #####

No consent prompt !!!

##### Explanation for no consent prompt #####

We added a client id of ProfileSPA against the API **access_ProfileAPI_1**  in the ProfileAPI-1. While evaluating the .default for the ProfileAPI-1, AzureAd took consideration of the scope **access_ProfileAPI_1** against which preauthorizedclient attribute is added and hence there is no consent prompt shown. Additionaly, an accesstoken is granted with the scope `"scp": "access_ProfileAPI_1"` without user having to consent it.

##### Learning from the above scenarios is that, if there is a knownclientapplication attribute added in the API for the client app, then it aggregates scope present in the API.Additionaly, knownclientapplicatin attribute works only with .default scope. ####

### 4. With knownClientApplications & with preAuthorizedApplications combined ###

In the previous sceanrios, we have already configured preAuthorizedApplications for the `ProfileAPI-1`. Now let us add client id of the ProfileSPA in the knownClientApplications attribute manifest of `ProfileAPI-1` to test combined behaviour of knownClientApplications & preAuthorizedApplications. We have already configured the knownclientapplivation attribute for the ProfileAPI-2, so let us now do it for the ProfileAPI-1.

To achieve this, you need to add the **Application Id** of the client app, in the Manifest of the web API in the `knownClientApplications` property. Here's how:

1. In the [Azure portal](https://portal.azure.com), navigate to your `ProfileAPI-1` app registration, and select **Manifest** section.
1. In the manifest editor, change the `"knownClientApplications": []` line so that the array contains the Client ID of the client application (`ProfileSPA`) as an element of the array.

For instance:

   ```json
   "knownClientApplications": ["ca8dca8d-f828-4f08-82f5-325e1a1c6428"],
   ```

1. **Save** the changes to the manifest.

#### Specific scopes sceanrios ####


##### Scenario 1: Request the scope for GRAPH resource #####

   1. Pass client id of `Graph API` with `Device.Read` scope. 
      - Scope value in the code `scopes: ["https://graph.microsoft.com/Device.Read"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://graph.microsoft.com/Device.Read openid profile offline_access`

##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-4.1.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `View your list of devices` is shown in the consent promt since ProfileSPA has  **Delegated API permissions** for **Device.Read** for **Graph** resource and scope parameter to authorize endpoint contains `Device.read`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.

##### Scenario 2: Request the scope for Azure Batch resource #####

   1. Pass client id of `Azure Batch Service` with `user_impersonation` scope. 
     - Scope value in the code `scopes: ["https://batch.core.windows.net/user_impersonation"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=https://batch.core.windows.net/user_impersonation openid profile offline_access` 
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-4.2.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Full access to Azure Batch Service API` is shown in the consent promt since ProfileSPA has  **Delegated API permissions** to **user_impersonation** scope for **AzureBatch service** and scope parameter to authorize endpoint contains `https://batch.core.windows.net/user_impersonation`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.

##### Scenario 3: Request the scope for ProfileAPI-1 resource which is tagged against the preauthorizedclient attribute #####

   1. Pass client id of `ProfileAPI-1` with `access_ProfileAPI_1` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/access_ProfileAPI_1"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/access_ProfileAPI_1 openid profile offline_access`

##### Consent prompt screenshot #####

No consent prompt !!!!

##### Explanation for no consent prompt in this scenario #####

Since we added a client id of ProfileSPA against the API **access_ProfileAPI_1**  in the ProfileAPI-1, there is no consent prompt shown. Additionaly, an accesstoken is granted with the scope `"scp": "access_ProfileAPI_1"` after the SPA hit the token endpoint without any user consent being shown.

##### Scenario 4: Request the scope for ProfileAPI-1 resource  which is NOT tagged against the preauthorizedclient attribute  #####

   1. Pass client id of `ProfileAPI-1` with `access_ProfileAPI_1_withoutpreauthclient` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient openid profile offline_access`
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-4.4.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Access for you to call access_ProfileAPI_1_withoutpreauthclient permissions (ProfileAPI-1)` is shown in the consent prompt since ProfileSPA has  **Delegated API permissions** to **access_ProfileAPI_1_withoutpreauthclient** scope for **ProfileAPI-1** and scope parameter to authorize endpoint contains `api://app_id_of_Profile_API_1/access_ProfileAPI_1_withoutpreauthclient`.
- The permission `View your basic profile` & `Maintain access to data you have given it access to` shown in the consent prompt since scope parameter to authorize endpoint has `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope, **profile** & **offline.access** scopes will be configured automatically for the apps signing into the applications in Azure AD as per the open id connect.
- 
##### Scenario 5: Request the scope for ProfileAPI-2 resource #####

   1. Pass client id of `ProfileAPI-2` with `access_ProfileAPI_2` scope. 
     - Scope value in the code `scopes: ["api://app_id_of_Profile_API_2/access_ProfileAPI_2"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_2/access_ProfileAPI_2 openid profile offline_access`
     
##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-4.5.jpg)

##### Explanation for the consents that are shown in the prompt #####
- The permission `Access for you to call access_ProfileAPI_2 permissions (ProfileAPI-2)` is shown in the consent prompt since ProfileSPA has  **Delegated API permissions** to **access_ProfileAPI_2** scope for **ProfileAPI-2** and scope parameter to authorize endpoint contains `api://app_id_of_Profile_API_2/access_ProfileAPI_2`. 
- The permission `View your basic profile` & `Maintain access to data you have given it access to`) shown in the consent prompt since scope parameter to authorize endpoint has `openid` `profile` & `offline_access`. Though you may only see ProfileSPA having **Delegated API permissions** for **User.Read** scope(To which scope is not sent in the request), **offline.access** **profile** are configured automatically for the apps signing into the applications in Azure AD.

#### .default scope scenarios ####

##### Scenario 6: #####
 
 You will observe a same consent prompt, for the below scope values being passed.
 
  1. Pass client id of `Profile API 2` with default scope. 
      - Scope value in the code `scopes: ["api://app_id_of_Profile_API_2/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_2/.default openid profile offline_access` 
  1. Pass client id of `Profile SPA` with default scope. 
      - Scope value in the code `scopes: ["app_id_of_Profile_SPA/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=app_id_of_Profile_SPA/.default openid profile offline_access`
   1. Pass client id of `Graph API` with default scope. 
     - Scope value in the code `scopes: ["https://graph.microsoft.com/.default"]`
     - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `https://graph.microsoft.com/.default openid profile offline_access`
   1. Pass client id of `Azure Batch Service` with default scope. 
      - Scope value in the code `scopes: ["https://batch.core.windows.net/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=.default openid profile offline_access`
   1. Just pass `.default` with default scope. 
      - Scope value in the code `scopes: [".default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=.default openid profile offline_access`

##### Consent prompt screenshot #####

![Screenshot](./ConsentScreenshot/Scenario-4.6.jpg)

##### Explanation for the consents that are shown in the prompt #####

 - The permissions in the black rectangle of the screenshot (`View your list of devices` & `Sign you in and read your profile`) in the consent prompt are shown since ProfileSPA has **Delegated API permissions** for **User.Read** & **Device.Read** from GRAPH resource.  Plus, permission(`Full access to Azure Batch Service API`) is shown since, it also has **Delegated API permissions** added to **user_impersonation** from AzureBatch resource.
  - The permissions in the red rectangle of the screenshot (`Access Azure Storage As the Signed-in User`, `Maintain access to data you have given it access to` & `Read your calendars`) in the consent prompt are shown since ProfileSPA has **Delegated API permissions** for **access_ProfileAPI_2** for ProfileAPI-1 resource. Then, ProfileAPI-2 ressource and then ProfileAPI-2 inturn has **Delegated API permissions** to **Calendars.Read** & **offline_access** for GRAPH resource and **user_impersonation** scope for Azure storage resource. As a result of a knownclientapplication attribute, instead of showing a consent to `access_ProfileAPI_2`, it combined the API permissions for ProfileAPI-2 and aggregated to 3 permissions in the consent.
 - The permission in the blue rectangle of the screenshot (`Have full access to Visual Studio Team Services REST APIs` & `Read all files that you have access to`) in the consent prompt, is shown since ProfileAPI-1 has **Delegated API permissions** added to **Calendars.Read** for GRAPH resource and **user_impersonation** scope for Azure devops resource. Please note that, ProfileSPA had an API permission **access_ProfileAPI_1** & **access_ProfileAPI_1_withoutpreauthclient** added for the resource ProfileAPI-1 and also, client id of the ProfileSPA is updated in the KnownclientApplications attribute of the ProfileAPI-1 manifest. As a result of which, consent agregated and combined the permissions which are present in the ProfileAPI-1. 

##### Scenario 7:  With appending .default scope to the app id of the API in which preauthorizedclient attribute is configure #####

1. Pass client id of `Profile API 1` with default scope. 
      - Scope value in the code `scopes: ["api://app_id_of_Profile_API_1/.default"]`
      - Final scope to authorize endpoint after appending the default graph scopes by MSAL - `scope=api://app_id_of_Profile_API_1/.default openid profile offline_access`

##### Consent prompt screenshot #####

No consent prompt !!!!

##### Explanation for no consent prompt #####

Since we added client id of ProfileSPA in the preauthorizedclient attribute against **access_ProfileAPI_1** scope of ProfileAPI-1, there is no consent prompt shown. Additionaly, an accesstoken is granted with the scope `"scp": "access_ProfileAPI_1"` without any user consent. Refer previous example under preauthorizedclient for more details.

##### Learning from the above scenarios is that, if there is a knownclientapplication attribute added in the API for the client app + plus preauthorizedclient attribuet configured to one of the scopes in the same API, then knownclientapplication attribute will not get into effect as preauthorizedclient behaviour overrides it. #####

## We'd love your feedback!

Were we successful in addressing your learning objective? Consider taking a moment to [share your experience with us](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR73pcsbpbxNJuZCMKN0lURpUOU5PNlM4MzRRV0lETkk2ODBPT0NBTEY5MCQlQCN0PWcu).

## More information

- [Microsoft identity platform (Azure Active Directory for developers)](https://docs.microsoft.com/azure/active-directory/develop/)
- [Overview of Microsoft Authentication Library (MSAL)](https://docs.microsoft.com/azure/active-directory/develop/msal-overview)
- [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app)
- [Quickstart: Configure a client application to access web APIs](https://docs.microsoft.com/azure/active-directory/develop/quickstart-configure-app-access-web-apis)
- [Understanding Azure AD application consent experiences](https://docs.microsoft.com/azure/active-directory/develop/application-consent-experience)
- [Understand user and admin consent](https://docs.microsoft.com/azure/active-directory/develop/howto-convert-app-to-be-multi-tenant#understand-user-and-admin-consent)
- [Application and service principal objects in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)
- [National Clouds](https://docs.microsoft.com/azure/active-directory/develop/authentication-national-cloud#app-registration-endpoints)
- [MSAL code samples](https://docs.microsoft.com/azure/active-directory/develop/sample-v2-code)

For more information about how OAuth 2.0 protocols work in this scenario and other scenarios, see [Authentication Scenarios for Azure AD](https://docs.microsoft.com/azure/active-directory/develop/authentication-flows-app-scenarios).

## Community Help and Support

Use [Stack Overflow](http://stackoverflow.com/questions/tagged/msal) to get support from the community.
Ask your questions on Stack Overflow first and browse existing issues to see if someone has asked your question before.
Make sure that your questions or comments are tagged with [`azure-active-directory` `dotnet` `ms-identity` `adal` `msal`].

If you find a bug in the sample, raise the issue on [GitHub Issues](../../../../issues).

To provide feedback on or suggest features for Azure Active Directory, visit [User Voice page](https://feedback.azure.com/forums/169401-azure-active-directory).

## Contributing

If you'd like to contribute to this sample, see [CONTRIBUTING.MD](/CONTRIBUTING.md).

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

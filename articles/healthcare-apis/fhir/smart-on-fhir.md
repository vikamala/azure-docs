---
title: SMART on FHIR - Azure Health Data Services
description: This tutorial describes how to use a proxy to enable SMART on FHIR applications with the FHIR service.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.author: kesheth
author: expekesheth
ms.date: 11/10/2022
---

# SMART on FHIR

Substitutable Medical Applications and Reusable Technologies [SMART on FHIR](https://docs.smarthealthit.org/) is a healthcare standard through which applications can access clinical information through a data store. It adds a security layer based on open standards including OAuth2 and OpenID Connect, to FHIR interfaces to enable integration with EHR systems. Using SMART on FHIR provides at least three important benefits:
- Applications have a known method for obtaining authentication/authorization to a FHIR repository
- Users accessing a FHIR repository with SMART on FHIR are restricted to resources associated with the user, rather than having access to all data in the repository
- Users have the ability to grant applications access to a further limited set of their data by using SMART clinical scopes.

<!---SMART Implementation Guide v1.0.0 is supported by Azure Health Data Services and Azure API Management (APIM). This is our recommended approach, as it enabled Health IT developers to comply with 21st Century Act Criterion §170.315(g)(10) Standardized API for patient and population services. 

Sample demonstrates and list steps that can be referenced to pass ONC G(10) with Inferno test suite.

--->

One of the main purposes of the specifications is to describe how an application should discover authentication endpoints for an FHIR server and start an authentication sequence. SMART on FHIR uses parameter naming conventions that aren’t immediately compatible with Azure Active Directory (Azure AD), the Azure Health Data Services (FHIR Service) has a built-in Azure AD SMART on FHIR proxy that enables a subset of the SMART on FHIR launch sequences. Specifically, the proxy enables the [EHR launch sequence](https://hl7.org/fhir/smart-app-launch/#ehr-launch-sequence).

Below tutorial describes steps to enable SMART on FHIR applications with FHIR Service.

## Prerequisites

- An instance of the FHIR Service
- .NET SDK 6.0
- [Enable cross-origin resource sharing (CORS)](configure-cross-origin-resource-sharing.md)
- [Register public client application in Azure AD]([https://learn.microsoft.com/azure/healthcare-apis/azure-api-for-fhir/register-public-azure-ad-client-app]
     - After registering the application, make note of the applicationId for client application.
     
<!--- Tutorial : To enable SMART on FHIR using APIM, follow below steps 
Step 1 : Set up FHIR SMART user role 
Follow the steps listed under section [Manage Users: Assign Users to Role](https://learn.microsoft.com/azure/active-directory/fundamentals/active-directory-users-assign-role-azure-portal). Any user added to this role will be able to access the FHIR Service if their requests comply with the SMART on FHIR implementation Guide, such as request having access token which includes a fhirUser claim and a clinical scopes claim.  The access granted to the users in this role will then be limited by the resources associated to their fhirUser compartment and the restrictions in the clinical scopes.

Step 2 : Deploy the necessary components to set up the FHIR server integrated with APIM in production. Follow ReadMe
Step 3 : Load US Core profiles 
Step 4 : Create Azure AD custom policy using this README  ---> 

Lets go over individual steps to enable SMART on FHIR 
## Step 1 : Set admin consent for your client application

To use SMART on FHIR, you must first authenticate and authorize the app. The first time you use SMART on FHIR, you also must get administrative consent to let the app access your FHIR resources.

If you don't have an ownership role in the app, contact the app owner and ask them to grant admin consent for you in the app. 

If you do have administrative privileges, complete the following steps to grant admin consent to yourself directly. (You also can grant admin consent to yourself later when you're prompted in the app.) You can complete the same steps to add other users as owners, so they can view and edit this app registration.

To add yourself or another user as owner of an app:

1. In the Azure portal, go to Azure Active Directory.
2. In the left menu, select **App Registration**.
3. Search for the app registration you created, and then select it.
4. In the left menu, under **Manage**, select **Owners**.
5. Select **Add owners**, and then add yourself or the user you want to have admin consent.
6. Select **Save**
   

## Step 2 : Configure Azure AD registrations

SMART on FHIR requires that `Audience` has an identifier URI equal to the URI of the FHIR service. The standard configuration of the FHIR service uses an `Audience` value of `https://azurehealthcareapis.com`. However, you can also set a value matching the specific URL of your FHIR service (for example `https://MYFHIRAPI.fhir.azurehealthcareapis.com`). This is required when working with the SMART on FHIR proxy.

## Step 3:  Enable the SMART on FHIR proxy

Enable the SMART on FHIR proxy in the **Authentication** settings for your FHIR instance by selecting the **SMART on FHIR proxy** check box.

The SMART on FHIR proxy acts as an intermediary between the SMART on FHIR app and Azure AD. The authentication reply (the authentication code) must go to the SMART on FHIR proxy instead of the app itself. The proxy then forwards the reply to the app. 

Because of this two-step relay of the authentication code, you need to set the reply URL (callback) for your Azure AD client application to a URL that is a combination of the reply URL for the SMART on FHIR proxy and the reply URL for the SMART on FHIR app. The combined reply URL takes this form:

```http
https://MYFHIRAPI.azurehealthcareapis.com/AadSmartOnFhirProxy/callback/aHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMS9zYW1wbGVhcHAvaW5kZXguaHRtbA
```

In that reply, `aHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMS9zYW1wbGVhcHAvaW5kZXguaHRtbA` is a URL-safe, base64-encoded version of the reply URL for the SMART on FHIR app. For the SMART on FHIR app launcher, when the app is running locally, the reply URL is `https://localhost:5001/sampleapp/index.html`. 

You can generate the combined reply URL by using a script like this:

```PowerShell
$replyUrl = "https://localhost:5001/sampleapp/index.html"
$fhirServerUrl = "https://MYFHIRAPI.azurewebsites.net"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($ReplyUrl)
$encodedText = [Convert]::ToBase64String($bytes)
$encodedText = $encodedText.TrimEnd('=');
$encodedText = $encodedText.Replace('/','_');
$encodedText = $encodedText.Replace('+','-');

$newReplyUrl = $FhirServerUrl.TrimEnd('/') + "/AadSmartOnFhirProxy/callback/" + $encodedText
```

Add the reply URL to the public client application that you created earlier for Azure AD

<!---![Reply URL configured for the public client](media/tutorial-smart-on-fhir/configure-reply-url.png)--->


## Step 4 :  Get a test patient

To test the FHIR service and the SMART on FHIR proxy, you'll need to have at least one patient in the database. If you've not interacted with the API yet, and you don't have data in the database, see [Access the FHIR service using Postman](./../fhir/use-postman.md) to load a patient. Make a note of the ID of a specific patient.

## Step 5 :  Download the SMART on FHIR app launcher

The open-source [FHIR Server for Azure repository](https://github.com/Microsoft/fhir-server) includes a simple SMART on FHIR app launcher and a sample SMART on FHIR app. In this tutorial, use this SMART on FHIR launcher locally to test the setup.

You can clone the GitHub repository and go to the application by using these commands:

```PowerShell
git clone https://github.com/Microsoft/fhir-server
cd fhir-server/samples/apps/SmartLauncher
```

The application needs a few configuration settings, which you can set in `appsettings.json`:

```json
{
    "FhirServerUrl": "https://MYFHIRAPI.fhir.azurehealthcareapis.com",
    "ClientId": "APP-ID",
    "DefaultSmartAppUrl": "/sampleapp/launch.html"
}
```

We recommend that you use the `dotnet user-secrets` feature:

```PowerShell
dotnet user-secrets set FhirServerUrl https://MYFHIRAPI.fhir.azurehealthcareapis.com
dotnet user-secrets set ClientId <APP-ID>
```

Use this command to run the application:

```PowerShell
dotnet run
```

## Step 6 : Test the SMART on FHIR proxy

After you start the SMART on FHIR app launcher, you can point your browser to `https://localhost:5001`, where you should see the following screen:

![Screenshot showing SMART on FHIR app launcher.](media/smart-on-fhir/smart-on-fhir-app-launcher.png)

When you enter **Patient**, **Encounter**, or **Practitioner** information, you'll notice that the **Launch context** is updated. When you're using the FHIR service, the launch context is simply a JSON document that contains information about patient, practitioner, and more. This launch context is base64 encoded and passed to the SMART on FHIR app as the `launch` query parameter. According to the SMART on FHIR specification, this variable is opaque to the SMART on FHIR app and passed on to the identity provider. 

The SMART on FHIR proxy uses this information to populate fields in the token response. The SMART on FHIR app *can* use these fields to control which patient it requests data for and how it renders the application on the screen. The SMART on FHIR proxy supports the following fields:

* `patient`
* `encounter`
* `practitioner`
* `need_patient_banner`
* `smart_style_url`

These fields are meant to provide guidance to the app, but they don't convey any security information. A SMART on FHIR application can ignore them.

Notice that the SMART on FHIR app launcher updates the **Launch URL** information at the bottom of the page. Select **Launch** to start the sample app, and you should see something like this sample:

![Screenshot showing SMART on FHIR app.](media/smart-on-fhir/smart-on-fhir-app.png)

Inspect the token response to see how the launch context fields are passed on to the app.


FHIR&#174; is a registered trademark of [HL7](https://hl7.org/fhir/) and is used with the permission of HL7.

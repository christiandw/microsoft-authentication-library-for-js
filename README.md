
Microsoft Authentication Library Preview for JavaScript (MSAL.js)
=========================================================

| [Getting Started](https://docs.microsoft.com/en-us/azure/active-directory/develop/guidedsetups/active-directory-javascriptspa)| [Docs](https://aka.ms/aaddevv2) | [Library Reference](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/docs/classes/_useragentapplication_.useragentapplication.html) | [Support](README.md#community-help-and-support) | [Samples](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Samples)
| --- | --- | --- | --- | --- |


The MSAL library preview for JavaScript enables client-side JavaScript web applications, running in a web browser, to authenticate users using [Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-overview) work and school accounts (AAD), Microsoft personal accounts (MSA) and social identity providers like Facebook, Google, LinkedIn, Microsoft accounts, etc. through [Azure AD B2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-overview#identity-providers) service. It also enables your app to get tokens to access [Microsoft Cloud](https://cloud.microsoft.com) services such as [Microsoft Graph](https://graph.microsoft.io).

[![Build Status](https://travis-ci.org/AzureAD/microsoft-authentication-library-for-js.png?branch=dev)](https://travis-ci.org/AzureAD/microsoft-authentication-library-for-js)[![npm version](https://img.shields.io/npm/v/msal.svg?style=flat)](https://www.npmjs.com/package/msal)[![npm version](https://img.shields.io/npm/dm/msal.svg)](https://nodei.co/npm/msal/)


## Installation
Via NPM:

    npm install msal

Via CDN:

    <!-- Latest compiled and minified JavaScript -->
    <script src="https://secure.aadcdn.microsoftonline-p.com/lib/<version>/js/msal.js"></script>
    <script src="https://secure.aadcdn.microsoftonline-p.com/lib/<version>/js/msal.min.js"></script>

Note that msal.js is built for ES5, therefore enabling support for Internet Explorer 11. If you want to target Internet Explorer, you'll need to add a reference to promises polyfill.

    <!-- IE support: add promises polyfill before msal.js  -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/bluebird/3.3.4/bluebird.min.js" class="pre"></script>

See here for more details on [supported browsers](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/FAQs#q4-what-browsers-is-msaljs-supported-on).

## Usage
The example snippets below show how to acquire a token for Microsoft Graph.

#### Prerequisite

Before using MSAL.js, [register an application in Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app) to get your clientID.

#### 1. Instantiate the UserAgentApplication

Instantiate the UserAgentApplication with a minimal required configuration of clientId.

You must register a callback handler if you are using redirect flow(loginRedirect and acquireTokenRedirect). This callback function is called after the authentication request is completed either successfully or with a failure. This is not required for the popup flows since they return promises.

UserAgentApplication has other optional parameters like redirectUri which can be assigned. Please refer to the [Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/MSAL-basics#configuration-options) to see the full list and their default values.

```JavaScript
    var applicationConfig = {
        auth: {
            clientId: 'your_client_id'
        }
    };

    var msalInstance = new Msal.UserAgentApplication(applicationConfig);

    msalInstance.handleRedirectCallbacks(response => {
        // handle redirect response
    }), err => {
       // handle error
    });

```

#### 2. Login the user

Your app must login the user with either loginPopup or the loginRedirect method to establish user context. When the login methods are called and the authentication of the user is completed by the Azure AD service, an [id token](https://docs.microsoft.com/en-us/azure/active-directory/develop/id-tokens) is returned which is used to identify the user with some basic information.

```JavaScript
   var loginRequest = {
       scopes: ["user.read", "mail.send"] // optional
   };

    msalInstance.loginPopup(loginRequest)
        .then(response => {
            // handle response
        })
        .catch(err => {
            // handle error
        });

```
> Note: The scopes passed to the login method are optional. In this example, the graphScopes are passed in the login method to obtain consent upfront from the user for your app to access certain Graph API scopes. The idtoken returned here does not contain the scopes. In the next step, you can see how to get an access token which will contain the consented scopes.

#### 3. Get an access token to call an API

In MSAL, you can get access tokens for the APIs your app needs to call using the acquireTokenSilent method which makes a silent request(without prompting the user) to Azure AD to obtain an access token. The Azure AD service then returns an [access token](https://docs.microsoft.com/en-us/azure/active-directory/develop/access-tokens) containing the user consented scopes to allow your app to securely call the API.

If the silent token acquisition fails for some reasons such as an expired token or password change, you will need to invoke an interactive method to acquire tokens such as acquireTokenPopup or acquireTokenRedirect.

 ```JavaScript

    var tokenRequest = {
        scopes: ["user.read", "mail.send"]
    };

    msalInstance.loginPopup(loginRequest)
        .then(response => msalInstance.acquireTokenSilent(graphScopes)
        .then(response {
            // get access token from response
            // response.accessToken
        })
        .catch(err => {
            // if the error requires user consent
            if (err.errorCode.indexOf("consent_required") !== -1 ||err.errorCode.indexOf("interaction_required") !== -1 ||err.errorCode.indexOf("login_required") !== -1 ) {
                return msalnstance.acquireTokenPopup(tokenRequest)
                    .then(response => {
                        // get access token from response
                        // response.accessToken
                    })
                    .catch(err => {
                        // handle error
                    })
            } else {
                // handle error
            }
        });


```

#### 4. Use the token as a bearer in an HTTP request to call the Microsoft Graph or a Web API

```JavaScript
    var headers = new Headers();
    var bearer = "Bearer " + token;
    headers.append("Authorization", bearer);
    var options = {
         method: "GET",
         headers: headers
    };
    var graphEndpoint = "https://graph.microsoft.com/v1.0/me";

    fetch(graphEndpoint, options)
        .then(resp => {
             //do something with response
        });
```

You can learn further details about MSAL.js functionality documented in the [MSAL Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki) and find complete [code samples](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Samples).

## Wrapper libraries

We provide preview versions of the following wrapper libraries as separate packages.

- [Microsoft Authentication Library for Angular Preview](lib/msal-angular/README.md) :
A wrapper of the core library for apps using Angular framework.

- [Microsoft Authentication Library for AngularJS Preview](lib/msal-angularjs/README.md) :
A wrapper of the core library for apps using AngularJS framework.

Please check the [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap) for details on release plans.

## Community Help and Support

- [FAQs](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/FAQs) for access to our frequently asked questions

- [Stack Overflow](http://stackoverflow.com/questions/tagged/msal) using "msal" and "msal.js" tag.

We highly recommend you ask your questions on Stack Overflow first and browse existing issues to see if someone has asked your question before.

- [GitHub Issues](../../issues) for reporting a bug or feature requests

- [User Voice page](https://feedback.azure.com/forums/169401-azure-active-directory) to provide recommendations and/or feedback

## Contribute

We enthusiastically welcome contributions and feedback. Please read the [contributing guide](contributing.md) before you begin.

## Security Library

This library controls how users sign-in and access services. We recommend you always take the latest version of our library in your app when possible. We use [semantic versioning](http://semver.org) so you can control the risk associated with updating your app. As an example, always downloading the latest minor version number (e.g. x.*y*.x) ensures you get the latest security and feature enhanements but our API surface remains the same. You can always see the latest version and release notes under the Releases tab of GitHub.

## Security Reporting

If you find a security issue with our libraries or services please report it to [secure@microsoft.com](mailto:secure@microsoft.com) with as much detail as possible. Your submission may be eligible for a bounty through the [Microsoft Bounty](http://aka.ms/bugbounty) program. Please do not post security issues to GitHub Issues or any other public site. We will contact you shortly upon receiving the information. We encourage you to get notifications of when security incidents occur by visiting [this page](https://technet.microsoft.com/en-us/security/dd252948) and subscribing to Security Advisory Alerts.

## License

Copyright (c) Microsoft Corporation.  All rights reserved. Licensed under the MIT License (the "License");

## We Value and Adhere to the Microsoft Open Source Code of Conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

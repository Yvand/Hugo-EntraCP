---
title: "Introduction"
description: ""
lead: ""
date: 2025-05-26T08:14:13Z
lastmod: 2025-08-21
draft: false
images: []
weight: 100
toc: true
---

EntraCP (formerly AzureCP) is a claims provider that runs in your SharePoint Server farm, to connect it to your Microsoft Entra ID tenant.  
It is useful in federated authentication (either with [WS-Federation](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/sharepoint-on-premises-tutorial) or [OpenID Connect](https://docs.microsoft.com/en-us/sharepoint/security-for-sharepoint-server/oidc-1-0-authentication)), to improve the user experience and fill some gaps in this scenario.

## Prerequisites

- SharePoint Subscription Edition, SharePoint 2019 or SharePoint 2016.
- [.NET Framework 4.8](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net48) or newer on all SharePoint servers.
- **All** the harePoint servers must be able to connect to Entra ID and Microsoft Graph.
- An [app registration in your Microsoft Entra ID tenant]({{< relref "register-application" >}}), with the appropriate permissions.

## Features

- Searches users and groups based on the people picker's input.
- Gets group membership of Entra ID users (augmentation).
- Queries multiple Entra ID tenants in parallel.
- Populates the metadata (e.g. email, display name) of the entities.
- Easy to configure through PowerShell or administration pages.
- No dependency on any SharePoint service application.

## Customization

EntraCP is highly customizable to adapt to your requirements:

- Securely connects to your Entra ID tenant using either a client secret or a client certificate.
- Customizes the display of the results in the people picker.
- Customizes the claim types and their mapping with Azure AD objects.
- Enables/disables augmentation.
- Enables/disables connection to your tenant, to keep EntraCP running with limited functionality if connectivity with your tenant is lost.

## Limitations

EntraCP cannot be used if:

- SharePoint servers have no network access to Entra ID or Microsoft Graph.
- Cmdlet `New-SPTrustedIdentityTokenIssuer` was run with the switch `-UseDefaultConfiguration`.
- It is already associated with an **SPTrustedIdentityTokenIssuer**, and you want to associate it with a new one.

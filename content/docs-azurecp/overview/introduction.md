---
title: "Introduction"
description: ""
lead: ""
date: 2023-09-18
lastmod: 2023-09-18
draft: false
images: []
weight: 10
toc: true
---

{{< callout context="danger" title="Danger" icon="outline/alert-octagon" >}}
AzureCP is outdated and no longer maintained. Follow [this guide](/docs-azurecp/guides/upgrade-to-entracp/) to upgrade to [EntraCP](/docs/overview/introduction).
{{< /callout >}}

## Prerequisites

- AzureCP 18+ requires at least [.NET Framework 4.7.2](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472) on all SharePoint servers.
- AzureCP 17 requires at least [.NET Framework 4.6.1](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net461) on all SharePoint servers.
- **All** SharePoint servers need to be able to connect to Internet. [Read this]({{< ref "/docs-azurecp/usage/configuration#configure-proxy-for-internet-access" >}}) if you need to configure a proxy.
- An [app registration in your Microsoft Entra ID tenant]({{< relref "register-application" >}}) with Microsoft Graph permissions `GroupMember.Read.All` and `User.Read.All`.

## Features

- Search users and groups based on the the people picker's input.
- Get group membership of Entra ID users (augmentation).
- Query multiple Entra ID tenants in parallel.
- Populate the metadata (e.g. email, display name) of the entities.
- Easy to configure through PowerShell or administration pages.
- No dependency on any SharePoint service application.

## Customization

AzureCP is highly customizable to adapt to your requirements:

- Securely connect to your Entra ID tenant using either a client secret or a client certificate.
- Customize the display of the results in the people picker.
- Customize the claim types and their mapping with Azure AD objects.
- Enable/disable augmentation.
- Enable/disable connection to Azure AD, to keep AzureCP running with limited functionality if connectivity with Azure AD is lost.

It queries your Microsoft Entra ID tenant(s) to search and validate users and groups:

![Image](images/people-picker-EntraCP.png)

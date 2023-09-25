---
title: "Overview"
description: ""
lead: ""
date: 2023-09-18
lastmod: 2023-09-18
draft: false
images: []
menu: 
  docs:
    parent: ""
weight: 10
toc: true
---

This is the documentation for EntraCP.

## Prerequisites

- SharePoint Subscription Edition, SharePoint 2019 or SharePoint 2016.
- [.NET Framework 4.8](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net48) or newer on all SharePoint servers.
- **All** SharePoint servers need to be able to connect to Internet.
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

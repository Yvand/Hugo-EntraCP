---
title: "Introduction"
description: ""
lead: ""
date: 2021-05-26T08:14:13Z
lastmod: 2021-08-06T11:15:29Z
draft: false
images: []
menu: 
  overview:
    parent: ""
weight: 100
toc: true
---

EntraCP (formerly AzureCP) is a claims provider that runs in your SharePoint Server farm, to connect it to your Microsoft Entra ID tenant.  
It is useful in federated authentication (either with [WS-Federation](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/sharepoint-on-premises-tutorial) or [OpenID Connect](https://docs.microsoft.com/en-us/sharepoint/security-for-sharepoint-server/oidc-1-0-authentication)), to improve the user experience and fill some gaps in this scenario.

## EntraCP or AzureCP?

### EntraCP

This is the [new version]({{< relref "/blog/announcing-entracp" >}}) that should be used on all the supported versions of SharePoint Server: SharePoint Subscription, SharePoint 2019 and SharePoint 2016.  
[Go to documentation]({{< relref "/docs/overview" >}}).

### AzureCP

This is the legacy, deprecated version. You should use it only if you have SharePoint 2013, and [migrate otherwise]({{< relref "introduction#migrate-from-azurecp-to-entracp" >}}).  
[Go to documentation]({{< relref "/docs-azurecp/overview" >}}).

## Should I migrate from AzureCP to EntraCP?

Yes. Here are a few reasons why:

- AzureCP is deprecated and no longer maintained.
- EntraCP uses the SharePoint solution type `Application` (instead of `Web Front End`), which makes administrative operations (install / update / delete) much easier and safer by design.
- EntraCP has a much better design and uses the latest versions of the Nuget packages [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) and [Azure.Identity](https://www.nuget.org/packages/Azure.Identity), while AzureCP uses outdated, no longer maintained dependencies.
- EntraCP has a single location to configure the proxy for all the SharePoint processes / servers in the farm.

## Migrate from AzureCP to EntraCP

There is no migration so to speak, because they are 2 completely different SharePoint solutions.  
Below are the possible strategies to migrage to EntraCP.  

### Uninstall AzureCP, then install EntraCP

This strategy can be used if you are confident about all the steps involved.

### Install EntraCP side by side with AzureCP

It is perfectly possible to have both claims providers installed on the same farm.  
To switch from one claims provider to the other, all you need to do is update the property `ClaimProviderName` using the cmdlet `Get-SPTrustedIdentityTokenIssuer`.  

At some point, you may decide to uninstall AzureCP, and this is where you need to be careful:  
Some assemblies are used by both. Although it should not happen, it was reported a few times that uninstalling AzureCP removed the assemblies also used by EntraCP.  
If this happens, simply retract and redeploy EntraCP.wsp.  
Unlike AzureCP, this operation is safe with EntraCP, and faster than finding out which assemblies got removed, and adding them back manually on each server.

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

This is the [new claims provider]({{< relref "/blog/announcing-entracp" >}}) to use on all the supported versions of SharePoint Server.  
[Go to documentation]({{< relref "/docs/overview" >}}).

### AzureCP

This is the legacy, deprecated claims provider, to use only with SharePoint 2013.  
[Go to documentation]({{< relref "/docs-azurecp/overview" >}}).

## Should I migrate from AzureCP to EntraCP?

Yes, you should:

- AzureCP is no longer maintained and uses outdated, deprecated 3rd party libraries.
- Administrative operations (install / update / delete) are much easier and safer in EntraCP, thanks to using the SharePoint solution type `Application` (instead of `Web Front End` in AzureCP).
- EntraCP uses modern and up-to-date packages [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) and [Azure.Identity](https://www.nuget.org/packages/Azure.Identity).
- EntraCP stores the proxy configuration directly, and it uses it in all SharePoint processes, in all the servers of the farm.

## Migrate from AzureCP to EntraCP

There is no migration so to speak, because they are 2 completely different SharePoint solutions.  
There are 2 possible strategies discussed here, but be mindful that a claims provider is a critical component in a SharePoint farm, and any approach should be tested in a non-production environment first.

### Uninstall AzureCP, then install EntraCP

This strategy can be used if you are confident about all the steps involved.

### Install EntraCP side by side with AzureCP

It is perfectly possible to have both claims providers installed on the same farm.  
Once both are installed, to switch from one to another, simply update the property `ClaimProviderName` from cmdlet `Get-SPTrustedIdentityTokenIssuer`.  

At some point, you may decide to uninstall AzureCP, and this is where you need to be careful:  
Some assemblies are used by both. Although it should not happen, it was reported a few times that uninstalling AzureCP removed assemblies also used by EntraCP.  
If this happens, simply retract and redeploy EntraCP.wsp.  
Unlike AzureCP, this operation is safe with EntraCP, and faster than finding out which assemblies got removed, and adding them back manually on each server.

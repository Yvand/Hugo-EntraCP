---
title: "Upgrade to EntraCP"
description: ""
lead: ""
date: 2021-05-26T08:14:13Z
lastmod: 2025-08-21
draft: false
images: []
weight: 100
toc: true
---

As a first step, make sure your SharePoint farm meets [the requirements](/docs/overview/introduction/) to run EntraCP.

There is no migration so to speak, because they are 2 completely different SharePoint solutions.  
There are 2 possible strategies discussed here, but be mindful that a claims provider is a critical component in a SharePoint farm, and any approach should be tested in a non-production environment first.

## Uninstall AzureCP, then install EntraCP

This strategy can be used if you are confident about all the steps involved.

## Install EntraCP side by side with AzureCP

It is perfectly possible to have both claims providers installed on the same farm.  
Once both are installed, to switch from one to another, simply update the property `ClaimProviderName` from cmdlet `Get-SPTrustedIdentityTokenIssuer`.

At some point, you may decide to uninstall AzureCP, and this is where you need to be careful:  
Some assemblies are used by both. Although it should not happen, it was reported a few times that uninstalling AzureCP removed assemblies also used by EntraCP.  
If this happens, simply retract and redeploy EntraCP.wsp.  
Unlike AzureCP, this operation is safe with EntraCP, and faster than finding out which assemblies got removed, and adding them back manually on each server.

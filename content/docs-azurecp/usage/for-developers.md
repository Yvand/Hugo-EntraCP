---
title: "For developers"
description: "Possibilities of customization for developers"
lead: "Possibilities of customization for developers"
date: 2021-05-20T10:45:52Z
lastmod: 2021-08-06T11:15:29Z
draft: false
images: []
menu:
  docs-azurecp:
    parent: "usage"
    identifier: "for-developers"
weight: 150
toc: true
---

## When AzureCP may need to be customized

Project has evolved a lot since it started, and now most of the customizations can be made out of the box.  
However, there are still some scenarios where a developer may want to customize AzureCP:

- Use AzureCP with multiple SPTrustedIdentityTokenIssuer.
- Have full control on the entities (permissions) created by AzureCP.

## How to proceed

For that, the class AzureCP can be inherited to create a class that will be a new, unique claims provider in SharePoint.  
[Each release](https://github.com/Yvand/AzureCP/releases) comes with a Visual Studio sample project "AzureCP.Developers.zip". It contains several sample classes that demonstrate various customizations capabilities.

## Things to know

- Each class inheriting AzureCP is a new claims provider, uniquely defined by the name of its class.
- Sample project has 1 feature event receiver, and it can manage only 1 claims provider.
- Both AzureCP.wsp and the custom version need azurecp.dll and all its dependent assemblies. Be aware that updating / removing one package will affect these assemblies (also used by the other package).
- To avoid deployment issues, always deactivate the farm feature (which manages the claims provider) before retracting the solution.

If something goes wrong during solution deployment, [check this page]({{< relref "../help/fix-setup-issues" >}}) to resolve problems.  

---
title: "Update"
description: "Update AzureCP"
lead: "Update AzureCP in your SharePoint farm"
date: 2021-05-20T10:45:52Z
lastmod: 2023-12-27
draft: false
images: []
menu:
  docs-azurecp:
    parent: "usage"
    identifier: "update"
weight: 150
toc: true
---

## Update the solution

{{< callout context="caution" title="Important" icon="alert-triangle" >}} Always start a new PowerShell process to ensure using up to date persisted objects and avoid nasty errors.<br>Bear in mind that additional steps are required on SharePoint servers which do not run the service 'Microsoft SharePoint Foundation Web Application'. {{< /callout >}}

On the server running the central administration:

1. Start a SharePoint management shell and run `Update-SPSolution`:

  ```powershell
  # This will start a timer job that will deploy the update on SharePoint servers. Central administration will restart during the process
  Update-SPSolution -GACDeployment -Identity "AzureCP.wsp" -LiteralPath "F:\Data\Dev\AzureCP.wsp"
  ```

1. Visit central administration > System Settings > Manage farm solutions: Wait until solution status shows "Deployed".

  {{< callout context="caution" title="Important" icon="alert-triangle" >}} Be patient, cmdlet Update-SPSolution triggers a one-time timer job on the SharePoint servers and this may take a minute or 2. {{< /callout >}}
  > If status shows "Error", restart the SharePoint timer service on servers where depployment failed, start a new PowerShell process and run Update-SPSolution again.

## Finalize the update

`Update-SPSolution` updates the bits only on the servers running the service "Microsoft SharePoint Foundation Web Application", but AzureCP must be updated on all SharePoint servers.  
To complete the update, follow the steps described in the section ["Finalize the installation" in the install page]({{< relref "install.md#finalize-the-installation" >}}).

## Breaking changes

Some versions have breaking changes and some settings will be reset to their default values during the upgrade:

- Upgrade to v12:
  - Claim type configuration list will be reset. This is due to the complete rewriting of this list.
  - Groups permissions are now created using the Id instead of the DisplayName. See below for more information.
- Upgrade to v13: Claim type configuration list will be reset. This is due to the changes made to handle Guest users.

## Group identifier change

Starting with v12, AzureCP creates group entities (permissions) using the Id instead of the DisplayName. There are 2 reasons for this change:

- Group Id is unique, DisplayName is not
- With group Id, AzureCP can get nested group membership of users during augmentation, which is not possible with the DisplayName

As a consequence of this change, permissions granted to Azure AD groups before v12 will stop working, because the group value in the SAML token of AAD users (set with the Id) won't match the group value of group permission in the sites (set with the DisplayName).

To fix this, group permissions must be migrated to change their value with the Id. This can be done by calling method [SPFarm.MigrateGroup()](https://msdn.microsoft.com/en-us/library/office/microsoft.sharepoint.administration.spfarm.migrategroup.aspx) for each group to migrate:

```powershell
# SPFarm.MigrateGroup() will migrate group "c:0-.t|contoso.local|AzureGroupDisplayName" to "c:0-.t|contoso.local|a5e76528-a305-4345-8481-af345ea56032" in the whole farm
$oldlogin = "c:0-.t|aadtrust|AzureGroupDisplayName";
$newlogin = "c:0-.t|aadtrust|a5e76528-a305-4345-8481-af345ea56032";
[Microsoft.SharePoint.Administration.SPFarm]::Local.MigrateGroup($oldlogin, $newlogin);
```

{{< callout context="caution" title="Important" icon="alert-triangle" >}} This operation is farm wide and must be tested carefully before it is applied in production environments. {{< /callout >}}

Alternatively, administrators can configure AzureCP to use the property DisplayName for the groups, instead of the Id:

```powershell
Add-Type -AssemblyName "AzureCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [azurecp.AzureCPConfig]::GetConfiguration("AzureCPConfig")
# Get the ClaimTypeConfig used for groups and set property DirectoryObjectProperty to DisplayName
$ctConfig = $config.ClaimTypes| ?{$_.ClaimType -eq "http://schemas.microsoft.com/ws/2008/06/identity/claims/role"}
$ctConfig.DirectoryObjectProperty = [azurecp.AzureADObjectProperty]::DisplayName
$config.Update()
```

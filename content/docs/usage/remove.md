---
title: "Remove"
description: "This article describes the steps to complete in order to successfully update EntraCP in your SharePoint farm."
lead: ""
date: 2021-05-20T10:45:52Z
lastmod: 2026-03-05
draft: false
images: []
weight: 130
toc: true
---

This article describes the steps to remove in order to successfully update EntraCP in your SharePoint farm.

{{< details "Is it safe to (un)install or update EntraCP in my SharePoint farm?" >}}
Yes, as long as you follow the steps in the documentation.  
Since EntraCP solution uses the deployment type `ApplicationServer`:

- Deploying/retracting the solution only copies/removes the files on disk (it does not installs/removes the EntraCP features).
- The features are installed/removed with a specific step, which prevents conflicts.
- The files are deployed on truly all SharePoint servers.

The biggest risk is a misconfiguration in the assembly bindings, which could prevent SharePoint to run.
{{< /details >}}

{{< callout context="caution" title="Do not just retract the solution" icon="outline/alert-triangle" >}} EntraCP features must be deactivated and uninstalled **before** the solution is retracted. {{< /callout >}}

## Update the trust configuration

Unfortunately, the only supported way to clear the property `ClaimProviderName` is to remove and recreate the `SPTrustedIdentityTokenIssuer` object.  
Alternatively, this property can be reset using .NET reflection, but this is not supported and you do this at your own risks:

```powershell
# Set private member m_ClaimProviderName to null. Note that using .NET reflection on SharePoint objects is not supported and you do this at your own risks
$trust = Get-SPTrustedIdentityTokenIssuer "<SPTRUST_NAME>"
$trust.GetType().GetField("m_ClaimProviderName", "NonPublic, Instance").SetValue($trust, $null)
$trust.Update()
```

## Uninstall EntraCP

{{< tabs "uninstall-entracp-type" >}}
{{< tab "Automated uninstall" >}}

Run the following script on the server running the central administration, to completely uninstall EntraCP from the SharePoint farm:

```powershell
<#
.SYNOPSIS
    Uninstalls "EntraCP.wsp" in a simple, automated way
.DESCRIPTION
    Run this script ONLY on the server running the central administration, in a new PowerShell process.
    The script does not require any modification.
.LINK
    https://entracp.yvand.net/docs/usage/remove/
#>

$product = "EntraCP"
$featureNamePrefix = "Yvand"
Write-Host "Disables the features present in $product.wsp solution"
Disable-SPFeature -Identity "$featureNamePrefix.$product" -Confirm:$false # This will remove the claims provider from the farm
Disable-SPFeature -Identity "$featureNamePrefix.$product.Administration" -Url $([Microsoft.SharePoint.Administration.SPAdministrationWebApplication]::Local.Url) -Confirm:$false

Write-Host "Uninstalls the features present in $product.wsp solution"
Uninstall-SPFeature -Identity "$featureNamePrefix.$product" -Confirm:$false
Uninstall-SPFeature -Identity "$featureNamePrefix.$product.Administration" -Confirm:$false

# Sanity check: The solution should not be retracted until its features are properly removed, because it cannot be done afterward
if ($null -eq (Get-SPFeature | ?{$_.DisplayName -like "$featureNamePrefix.$product*"}) -and
    $null -eq (Get-SPClaimProvider -Identity "$product" -ErrorAction SilentlyContinue)) {

    Write-Host "Retracts the $product.wsp solution"
    Uninstall-SPSolution -Identity "$product.wsp" -Confirm:$false

    $count = 0
    $maxAttempts = 10
    do
    {
        Write-Host "Waiting for the solution $product.wsp to be retracted..."
        Start-Sleep -Seconds 10
        $solution = Get-SPSolution -Identity "$product.wsp"
        $count++
    }
    while ($count -lt $maxAttempts -and $false -ne $solution.Deployed -and $false -ne $solution.JobExists)

    if ($false -eq $solution.Deployed -and $false -eq $solution.JobExists) {
        Write-Host "Removes the $product.wsp solution"
        Remove-SPSolution -Identity "$product.wsp" -Confirm:$false
    }
}
```

{{< /tab >}}
{{< tab "Manual uninstall" >}}

This script does the minimum work required in PowerShell, before you can safely retract the solution from the central administration.

1. Run the following script on the server running the central administration:

   ```powershell
   # Disables the features present in EntraCP.wsp solution
   Disable-SPFeature -Identity "Yvand.EntraCP" # This will remove the claims provider from the farm
   Disable-SPFeature -Identity "Yvand.EntraCP.Administration" -Url $([Microsoft.SharePoint.Administration.SPAdministrationWebApplication]::Local.Url)

   # Uninstalls the features present in EntraCP.wsp solution
   Uninstall-SPFeature -Identity "Yvand.EntraCP"
   Uninstall-SPFeature -Identity "Yvand.EntraCP.Administration"

   # Sanity check
   if ($null -eq (Get-SPFeature | ?{$_.DisplayName -like "Yvand.EntraCP*"}) -and
       $null -eq (Get-SPClaimProvider -Identity "EntraCP" -ErrorAction SilentlyContinue)) {
       Write-Host "You can now safely retract EntraCP.wsp"
   } else {
       Write-Warning "You should not retract EntraCP.wsp until all its features and its claims provider are properly removed. You won't be able to do it after you retracted the solution, unless you redeploy the solution"
   }
   ```

1. Browse to the central administration > System Settings > Manage farm solutions > EntraCP.wsp > Retract Solution
1. Once the solution is retracted, click again on EntraCP.wsp > Remove Solution

{{< /tab >}}
{{< /tabs >}}

## Remove the assembly bindings

{{< callout context="caution" title="Steps order" icon="outline/alert-triangle" >}} This step must be completed on **all** SharePoint servers, **after** the solution was uninstalled. {{< /callout >}}

1. Open `%systemroot%\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config` in a text editor.
2. Locate the XML node `<runtime>`.
3. Delete the entire node with its children and replace it with `<runtime />`.
4. Save the file `machine.config`.

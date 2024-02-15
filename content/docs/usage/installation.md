---
title: "Installation"
description: "This article describes the steps required to install EntraCP in your SharePoint farm."
lead: ""
date: 2021-05-20T10:45:52Z
lastmod: 2024-02-15
draft: false
images: []
menu:
  docs:
    parent: "usage"
    identifier: "installation"
weight: 100
toc: true
---

This article describes the steps required to install EntraCP in your SharePoint farm.

{{< details "About the installation" >}}
Installing EntraCP is much easier and safer than AzureCP because it uses the deployment type `ApplicationServer`, which implies that:

- Its features are installed with a specific, additional step, preventing conflicts.
- Its assemblies are deployed on truly all SharePoint servers.
{{< /details >}}

## Download the required assets

Browse to the [latest release](https://github.com/Yvand/EntraCP/releases/) and download the assets `assembly-bindings.config` and `EntraCP.wsp`.

## Set the assembly bindings

{{< details "Why this is needed" >}}
EntraCP uses NuGet packages [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) and [Azure.Identity](https://www.nuget.org/packages/Azure.Identity), which both require assembly bindings to work with .NET Framework 4.8 ([more info](https://nickcraver.com/blog/2020/02/11/binding-redirects/)).  
Since SharePoint runs in many processes (w3wp.exe, owstimer.exe, powershell.exe, etc...), the only file that can propagate the bindings to all is the `machine.config`.
{{< /details >}}

{{< callout context="caution" title="Important" icon="alert-triangle" >}} The steps below must be completed on all the SharePoint servers, before the solution is deployed. {{< /callout >}}

1. Open the `machine.config` file (`%systemroot%\Microsoft.NET\Framework64\v4.0.30319\Config\Machine.config`) in a text editor.
1. Locate the node `<runtime />`.
1. Replace it with the entire node `<runtime>` in the file `assembly-bindings.config` you downloaded.
1. Save the file.

## Install EntraCP

{{< tabs "install-entracp-type" >}}
{{< tab "Automated install" >}}

Run the following script on the server running the central administration, in a new PowerShell process:

```powershell {title="Automated installation script for EntraCP" lineNos=true}
<#
.SYNOPSIS
    Deploys the SharePoint solution EntraCP.wsp, created with the deployment mode "Application"
.DESCRIPTION
    Run this script ONLY on the server running the central administration, in a new PowerShell process.
    The script does not require any modification, except to update the path in $packagefullpath.

    EntraCP.wsp uses deployment mode "Application", which by design makes its deployment much more secure than AzureCP.
    Because contrary to AzureCP, running Install-SPSolution does NOT install the features in the farm, which prevents conflicts.
.LINK
    https://entracp.yvand.net/docs/usage/installation/
#>

$product = "EntraCP"
$packagefullpath = "C:\YvanData\$product.wsp" # Only update the path here

# Add the solution if it's not already present in the farm
if ($null -eq (Get-SPSolution -Identity "$product.wsp" -ErrorAction SilentlyContinue)) {
    Write-Host "Adding solution $product.wsp to the farm..."
    Add-SPSolution -LiteralPath $packagefullpath
}

$count = 0
while (($count -lt 20) -and ($null -eq $solution))
{
    Write-Host "Waiting for the solution $product.wsp to be available..."
    Start-Sleep -Seconds 5
    $solution = Get-SPSolution -Identity "$product.wsp"
    $count++
}

if ($null -eq $solution) {
    Write-Error "Solution $product.wsp could not be found in the farm."
    throw ("Solution $product.wsp could not be found in the farm.")
}

Write-Host "Deploying solution $product.wsp globally..."
Install-SPSolution -Identity "$product.wsp" -GACDeployment

$solution = Get-SPSolution -Identity "$product.wsp"
$count = 0
while (($count -lt 20) -and ($false -eq $solution.Deployed))
{
    Write-Host "Waiting for the solution $product.wsp to be deployed..."
    Start-Sleep -Seconds 10
    $solution = Get-SPSolution -Identity "$product.wsp"
    $count++
}

if ($null -ne (Get-SPFeature| Where-Object{$_.SolutionId -eq $solution.SolutionId}) -or
    $null -ne (Get-SPClaimProvider -Identity "$product" -ErrorAction SilentlyContinue)) {
  Write-Warning "The claims provider and/or the features are already installed, skip Install-SPFeature"
} else {
  Write-Host "Installing the features in the solution $product.wsp..."
  Install-SPFeature -SolutionId $solution.Id -AllExistingFeatures
}
Write-Host "Finished."
```

{{< /tab >}}
{{< tab "Manual install" >}}

Do the following on the server running the central administration:

1. Add the solution to the farm:

    ```powershell
    Add-SPSolution -LiteralPath "C:\YvanData\dev\EntraCP.wsp"
    ```

1. Navigate to the central administration > Security > Manage farm solutions > click on "entracp.wsp" > Deploy solution.
1. Monitor the deployment of the solution and wait for it to be fully deployed.
1. Install the features present in the solution:

    ```powershell
    Install-SPFeature -SolutionId "dd03bdd7-0645-475e-a852-f180b8bc8191" -AllExistingFeatures
    ```

{{< /tab >}}
{{< /tabs >}}

## Finalize the installation

On each SharePoint server, restart the IIS and the SharePoint timer services:

```powershell
Restart-Service -Name @("W3SVC", "SPTimerV4")
```

## Enable the claims provider

To be enabled, EntraCP must be associated with the SPTrustedLoginProvider created when the federation was configured.  
Execute this script on the server running the central administration:

```powershell
$trust = Get-SPTrustedIdentityTokenIssuer "YOUR_SPTRUST_NAME"
$trust.ClaimProviderName = "EntraCP"
$trust.Update()
```

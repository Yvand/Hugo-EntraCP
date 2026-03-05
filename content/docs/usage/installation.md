---
title: "Installation"
description: "This article describes the steps to complete in order to successfully install EntraCP in your SharePoint farm."
lead: ""
date: 2021-05-20T10:45:52Z
lastmod: 2026-03-05
draft: false
images: []
weight: 110
toc: true
---

This article describes the steps to complete in order to successfully install EntraCP in your SharePoint farm.

{{< details "Is it safe to (un)install or update EntraCP in my SharePoint farm?" >}}
Yes, as long as you follow the steps in the documentation.  
Since EntraCP solution uses the deployment type `ApplicationServer`:

- Deploying/retracting the solution only copies/removes the files on disk (it does not installs/removes the EntraCP features).
- The features are installed/removed with a specific step, which prevents conflicts.
- The files are deployed on truly all SharePoint servers.

The biggest risk is a misconfiguration in the assembly bindings, which could prevent SharePoint to run.
{{< /details >}}

{{< callout context="caution" title="Updated steps order" icon="outline/alert-triangle" >}} Ensure to [deploy the solution](#deploy-the-solution) **before** [setting the assembly bindings](#set-the-assembly-bindings). {{< /callout >}}


## Download the required assets

Browse to the [latest release](https://github.com/Yvand/EntraCP/releases/) and download the assets `assembly-bindings.config` and `EntraCP.wsp`.

## Deploy the solution

{{< tabs "install-entracp-type" >}}
{{< tab "Automated install" >}}

Run the following script on the server running the central administration, in a **new** PowerShell process:

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

1. Navigate to the central administration > System Settings > Manage farm solutions > click on "entracp.wsp" > Deploy solution.
1. Monitor the deployment of the solution and wait for it to be fully deployed.
1. Install the features present in the solution:

   ```powershell
   Install-SPFeature -SolutionId "dd03bdd7-0645-475e-a852-f180b8bc8191" -AllExistingFeatures
   ```

{{< /tab >}}
{{< /tabs >}}

## Set the assembly bindings

In this step, you set assembly bindings in the `machine.config` file using the content in file `assembly-bindings.config`, to ensure EntraCP can load its dependencies.

{{< details "Why are those bindings needed?" >}}
EntraCP uses NuGet packages [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) and [Azure.Identity](https://www.nuget.org/packages/Azure.Identity), which both require assembly bindings to work with .NET Framework 4.8 ([more info](https://nickcraver.com/blog/2020/02/11/binding-redirects/)).
{{< /details >}}
{{< details "Why setting them in the machine.config?" >}}
Since SharePoint runs in many processes (w3wp.exe, owstimer.exe, powershell.exe, etc...), the only config file that can propagate the bindings to all is the `machine.config`.
{{< /details >}}

{{< callout context="caution" title="Steps order" icon="outline/alert-triangle" >}} This step must be completed on **all** SharePoint servers, **after** the solution was deployed. {{< /callout >}}

{{< callout context="caution" title="Assembly bindings depend on the EntraCP version" icon="outline/alert-triangle" >}} Make sure to use the `assembly-bindings.config` corresponding to your version of EntraCP, as each release has unique assembly bindings. {{< /callout >}}

1. Open `%systemroot%\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config` in a text editor.
2. Locate the XML node `runtime` (search `<runtime />` or `<runtime>`).
3. Replace it with the content in the file `assembly-bindings.config`.
4. Save the file `machine.config`.

<!-- ## Restart the services

On each SharePoint server, restart the IIS and the SharePoint timer services:

```powershell
Restart-Service -Name @("W3SVC", "SPTimerV4")
``` -->

## Validate the setup

EntraCP includes special page **TroubleshootEntraCP.aspx**, that helps to validate the install/update was completed successfully, and the [prerequisites]({{< relref "../overview/introduction#prerequisites" >}}) are met.  
It is standalone (it does NOT use your EntraCP configuration) and can be found in the central administration > Security.  
[More info]({{< relref "../help/troubleshooting#use-the-built-in-troubleshooting-page" >}}) about this page.

## Enable the claims provider

To be enabled, EntraCP must be associated with the SPTrustedLoginProvider created when the federation was configured.  
Execute this script on the server running the central administration:

```powershell
$trust = Get-SPTrustedIdentityTokenIssuer "YOUR_SPTRUST_NAME"
$trust.ClaimProviderName = "EntraCP"
$trust.Update()
```

---
title: "Update"
description: "This article describes the steps to complete in order to successfully update EntraCP in your SharePoint farm."
lead: ""
date: 2021-05-20T10:45:52Z
lastmod: 2026-03-05
draft: false
images: []
weight: 120
toc: true
---

This article describes the steps to complete in order to successfully update EntraCP in your SharePoint farm.

{{< details "Is it safe to (un)install or update EntraCP in my SharePoint farm?" >}}
Yes, as long as you follow the steps in the documentation, as the solution's deployment type (`ApplicationServer`) means that:

- Deploying/retracting the solution only copies/removes the files on disk (it does not installs/removes the EntraCP features).
- The features are installed/removed with a specific step, which prevents conflicts.
- The files are deployed on truly all SharePoint servers.

The biggest risk is a misconfiguration in the assembly bindings, which could prevent SharePoint to run.
{{< /details >}}

{{< callout context="caution" title="Updated steps order" icon="outline/alert-triangle" >}} Make sure to [update the solution](#update-the-solution) **before** [setting the assembly bindings](#set-the-assembly-bindings). {{< /callout >}}

## Download the required assets

Browse to the [latest release](https://github.com/Yvand/EntraCP/releases/) and download the assets `assembly-bindings.config` and `EntraCP.wsp`.

## Update the solution

Complete the steps below on the server running the central administration:

1. On the server running the central administration, start a new SharePoint management shell and run this command:

   ```powershell
   # This will start a timer job that will deploy the update on SharePoint servers. Central administration will restart during the process
   Update-SPSolution -GACDeployment -Identity "EntraCP.wsp" -LiteralPath "C:\YvanData\EntraCP.wsp"
   ```

1. Visit central administration > System Settings > Manage farm solutions: Wait until solution status shows "Deployed".
   {{< callout context="note" title="Note" icon="outline/info-circle" >}} `Update-SPSolution` triggers a one-time timer job, which may take a few minutes to run on all SharePoint servers. {{< /callout >}}
   > If status shows "Error", restart the SharePoint timer service on the servers where the depployment failed, start a new PowerShell process and run `Update-SPSolution` again.

## Set the assembly bindings

In this step, you set the assembly bindings in the `machine.config` file using the content in file `assembly-bindings.config`, to ensure EntraCP can load its dependencies.

{{< details "Why are those bindings needed?" >}}
EntraCP uses NuGet packages [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) and [Azure.Identity](https://www.nuget.org/packages/Azure.Identity), which both require assembly bindings to work with .NET Framework 4.8 ([more info](https://nickcraver.com/blog/2020/02/11/binding-redirects/)).
{{< /details >}}
{{< details "Why setting them in the machine.config?" >}}
Since SharePoint runs in many processes (w3wp.exe, owstimer.exe, powershell.exe, etc...), the only config file that can propagate the bindings to all is the `machine.config`.
{{< /details >}}

{{< callout context="caution" title="Steps order" icon="outline/alert-triangle" >}} This step must be completed on **all** SharePoint servers, **after** the solution was deployed. {{< /callout >}}

{{< callout context="caution" title="Assembly bindings depend on the EntraCP version" icon="outline/alert-triangle" >}} Make sure to use the `assembly-bindings.config` corresponding to your version of EntraCP, as each release has unique assembly bindings. {{< /callout >}}

1. Open file `%systemroot%\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config` in a text editor.
2. Locate the XML node `runtime` (search `<runtime />` or `<runtime>`).
3. Replace it with the content in the file `assembly-bindings.config`.
4. Save the file `machine.config`.

## Validate the setup

EntraCP includes special page **TroubleshootEntraCP.aspx**, that helps to validate the install/update was completed successfully, and the [prerequisites]({{< relref "../overview/introduction#prerequisites" >}}) are met.  
It is standalone (it does NOT use your EntraCP configuration) and can be found in the central administration > Security.  
[More info]({{< relref "../help/troubleshooting#use-the-built-in-troubleshooting-page" >}}) about this page.

<!-- ## Finalize the installation

On each SharePoint server, restart the IIS and the SharePoint timer services:

```powershell
Restart-Service -Name @("W3SVC", "SPTimerV4")
``` -->

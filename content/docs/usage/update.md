---
title: "Update"
description: "This article describes the steps required to update EntraCP in your SharePoint farm."
lead: ""
date: 2021-05-20T10:45:52Z
lastmod: 2021-08-06T11:15:29Z
draft: false
images: []
menu:
  docs:
    parent: "usage"
    identifier: "update"
weight: 120
toc: true
---

This article describes the steps required to update EntraCP in your SharePoint farm.

## Download the required assets

Browse to the [latest release](https://github.com/Yvand/EntraCP/releases/) and download the assets `assembly-bindings.config` and `EntraCP.wsp`.

## Update the assembly bindings

{{< callout context="caution" title="Important" icon="alert-triangle" >}} Each new release of EntraCP may have unique assembly bindings, depending on the updates made on the dependencies it is using. {{< /callout >}}

Complete the steps below on **all the SharePoint servers**:

1. Browse to the [latest release](https://github.com/Yvand/EntraCP/releases/) and download the assets `assembly-bindings.config` and `EntraCP.wsp`.
1. Open the `machine.config` file (`%systemroot%\Microsoft.NET\Framework64\v4.0.30319\Config\Machine.config`) and locate the XML node `<runtime>`.
1. If needed, update the `machine.config` to reflect the changes in `assembly-bindings.config`.

## Update the solution

Complete the steps below on the server running the central administration:

1. On the server running the central administration, start a new SharePoint management shell and run this command:

   ```powershell
   # This will start a timer job that will deploy the update on SharePoint servers. Central administration will restart during the process
   Update-SPSolution -GACDeployment -Identity "EntraCP.wsp" -LiteralPath "C:\YvanData\EntraCP.wsp"
   ```

1. Visit central administration > System Settings > Manage farm solutions: Wait until solution status shows "Deployed".
   {{< callout context="caution" title="Important" icon="alert-triangle" >}} Be patient, cmdlet Update-SPSolution triggers a one-time timer job on the SharePoint servers and this may take a minute or 2. {{< /callout >}}
   > If status shows "Error", restart the SharePoint timer service on the servers where the depployment failed, start a new PowerShell process and run `Update-SPSolution` again.

## Finalize the installation

On each SharePoint server, restart the IIS and the SharePoint timer services:

```powershell
Restart-Service -Name @("W3SVC", "SPTimerV4")
```

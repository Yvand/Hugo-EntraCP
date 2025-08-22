---
title: "Set the credentials"
description: ""
lead: ""
date: 2023-11-30
lastmod: 2025-08-21
draft: false
images: []
toc: true
---

Follow this article to set the credentials used by EntraCP to connect to [your Entra ID tenant](https://entra.microsoft.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType//sourceType/Microsoft_AAD_IAM).

## Prerequisites

- An app registration [created for EntraCP]({{< relref "register-application" >}}) in your Entra ID tenant.
- The app registration has either a valid client secret or client certificate.

## Add new credentials to a tenant

The tenant can be added from the EntraCP global configuration page in the central administration, or using PowerShell:

{{< tabs "add-credentials" >}}
{{< tab "Central administration" >}}

Follow the steps below to add a connection to your Entra ID tenant using the Central Administration:

- Navigate to the SharePoint Central Administration > Security > EntraCP Global configuration.
- In the section **Register a new Microsoft Entra ID tenant**, fill the fields to add a connection to your tenant.
- Click on **Add tenant** to save your changes.

{{< /tab >}}
{{< tab "PowerShell" >}}

Run the script below to add a connection to your Entra ID tenant using Powershell:

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings

# Add a tenant using a client secret
$tenant = New-Object "Yvand.EntraClaimsProvider.Configuration.EntraIDTenant"
$tenant.Name = "TENANTNAME.onmicrosoft.com"
$tenant.SetCredentials("clientId", "clientSecret")
$settings.EntraIDTenants.Add($tenant)
$config.ApplySettings($settings, $true)

# Add a tenant using a client certificate
$tenant = New-Object "Yvand.EntraClaimsProvider.Configuration.EntraIDTenant"
$tenant.Name = "TENANTNAME.onmicrosoft.com"
$certificateFilePath = "pfxCertificateFilePath"
$certificatePassword = "pfxCertificatePassword"
$tenant.SetCredentials("clientId", $certificateFilePath, $certificatePassword)
$settings.EntraIDTenants.Add($tenant)
$config.ApplySettings($settings, $true)

# Commit the configuration to the database
$config.Update($true)
```

{{< /tab >}}
{{< /tabs >}}

## Update the credentials of a tenant

{{< callout context="note" title="Note" icon="outline/info-circle" >}}
You can use the central administration only if you want to update a client secret. To update a client certificate, use PowerShell
{{< /callout >}}

{{< tabs "update-credentials" >}}
{{< tab "Central administration" >}}

Follow the steps below to edit a connection to your Entra ID tenant using the Central Administration:

- Navigate to the SharePoint Central Administration > Security > EntraCP Global configuration.
- In the section **Registered Microsoft Entra ID tenants**, click on **Edit** next to the tenant you want to update.
- Make your necessary changes.
- Click on **Update** to save them.

{{< /tab >}}
{{< tab "PowerShell" >}}

Run the scripts below to update a connection to your Entra ID tenant using Powershell:

### Set a new client secret

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$config.UpdateTenantCredentials("tenantNameToUpdate", "newClientSecret")
$config.Update($true)
```

### Set a new client certificate

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$newCertificatePath = "C:\YvanData\config\certs\azurecp.aadcert.pfx"
$newCertificatePassword = "certicicatePassword"
$config.UpdateTenantCredentials("tenantNameToUpdate", $newCertificatePath, $newCertificatePassword)
$config.Update($true)
```

{{< /tab >}}
{{< /tabs >}}

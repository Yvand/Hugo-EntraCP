---
title: "Set the credentials"
description: ""
lead: ""
date: 2023-11-30
lastmod: 2026-01-30
draft: false
images: []
toc: true
---

Follow this article to set the credentials EntraCP will use to connect to [your Entra ID tenant](https://entra.microsoft.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType//sourceType/Microsoft_AAD_IAM).

## Prerequisites

- An app registration [created for EntraCP]({{< relref "register-application" >}}) in your Entra ID tenant.
- The app registration has either a valid client secret or client certificate.

## Add credentials for a tenant

{{< tabs "add-credentials" >}}
{{< tab "Central administration" >}}

Follow the steps below to add a connection to your Entra ID tenant using the Central Administration:

- Navigate to the SharePoint Central Administration > Security > EntraCP Global configuration.
- In the section **Register a new Microsoft Entra ID tenant**, fill the required fields.
- Click on **Add tenant** to save your changes.

{{< /tab >}}
{{< tab "PowerShell" >}}

The Powershell script below adds Entra ID tenants using both a client secret and certificate:

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

## Update credentials for a tenant

{{< callout context="note" title="Note" icon="outline/info-circle" >}}
The new credentials will replace the existing ones and will be used immediately by EntraCP.  
You can use the central administration only to set a new client secret. To set a new client certificate, use PowerShell.
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

The Powershell scripts below update the credentials for an existing Entra ID tenant:

### Set a new client secret

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$config.SetTenantSecret("TENANTNAME.onmicrosoft.com", "newClientSecret")
$config.Update($true)
```

### Set a new client certificate

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$newCertificatePath = "C:\YvanData\config\certs\azurecp.aadcert.pfx"
$newCertificatePassword = "certicicatePassword"
$config.SetTenantCertificate("TENANTNAME.onmicrosoft.com", $newCertificatePath, $newCertificatePassword)
$config.Update($true)
```

{{< /tab >}}
{{< /tabs >}}

---
title: "Register your application"
description: ""
lead: ""
date: 2021-05-20T10:45:06Z
lastmod: 2026-01-30
draft: false
images: []
toc: true
---

You need to create an app registration in your Entra ID tenant, to allow EntraCP to query it.  
This article will guide you through the steps to create it.

## Permissions required

EntraCP connects to your tenant to search for users, groups, and to get the group membership of the users.  
To achieve this, it needs the application permissions (not delegated) `GroupMember.Read.All` and `User.Read.All`.

{{< callout context="note" title="Note" icon="outline/info-circle" >}}
The less privileged permission [`User.ReadBasic.All`](https://learn.microsoft.com/en-us/graph/permissions-reference#userreadbasicall) cannot be used because it prevents using the property `UserType`, which is essential to determine if the user is a member or a guest. More information [here](https://github.com/Yvand/EntraCP/issues/265).
{{< /callout >}}

## Create the app registration

{{< tabs "create-app-registration" >}}
{{< tab "Entra ID portal" >}}

1. [Connect](https://entra.microsoft.com/) to your Entra ID tenant.
1. Under **Entra ID**, expand **Applications** and click **App Registrations** > **New registration** > Type the following information:
   - Name: EntraCP
   - Supported account types: **Accounts in this organizational directory only (Single tenant)**
1. Click **Register**
1. Click **API permissions**
   - Remove the default permission.
   - Click **Add a permission** > Select `Microsoft Graph` > **Application permissions**, and select `GroupMember.Read.All` and `User.Read.All`.
   - Click **Grant admin consent for TenantName** > **Yes**.
1. Click on **Certificates & secrets**: EntraCP supports both a certificate and a secret, choose either option depending on your needs.

{{< /tab >}}
{{< tab "m365 cli" >}}

[m365 cli](https://pnp.github.io/cli-microsoft365/) makes the registration very simple: It takes a single command to create the application, create a secret, set the permissions and grant the admin consent:

```bash
m365 login
# The command will print all the information that EntraCP needs to connect.
m365 entra app add --name "EntraCP" --withSecret --apisApplication 'https://graph.microsoft.com/User.Read.All,https://graph.microsoft.com/GroupMember.Read.All' --grantAdminConsent
```

{{< /tab >}}
{{< tab "bash" >}}

This bash script creates the application, adds a secret, sets the permissions and grants the admin consent.  
It can be used in Azure cloud shell or in a local shell:

```bash
#!/bin/bash
set -euo pipefail

# Sign-in to your Entra ID tenant. Use --allow-no-subscriptions if it doesn't have a subscription
az login #--allow-no-subscriptions

appName="EntraCP"
echo "Creating application '$appName'..."
appId=$(az ad app create --display-name "$appName" --query 'id' -o tsv)

echo "Creating service principal for application id '$appId', and getting its objectId (stored in field id)..."
spObjectId=$(az ad sp create --id $appId --query 'id' -o tsv)
echo "Application '$appName' was created with client id '$appId', and its service principal with objectId '$spObjectId'"

echo "Creating client secret for the app id '$appId'..."
appSecret=$(az ad app credential reset --id $appId --query 'password' --only-show-errors -o tsv)

# Retrieve the id of each permission to add
microsoftGraphId="00000003-0000-0000-c000-000000000000"
userPermName="User.Read.All"
groupPermName="GroupMember.Read.All"
userPermId=$(az ad sp show --id "$microsoftGraphId" --query "appRoles[?value=='$userPermName'].id" --output tsv)
groupPermId=$(az ad sp show --id "$microsoftGraphId" --query "appRoles[?value=='$groupPermName'].id" --output tsv)
msGraphResourceId=$(az ad sp show --id "$microsoftGraphId" --query "id" --output tsv)

# Add those permissions to the application
echo "Adding permissions '$userPermName' and '$groupPermName' to the app id '$appId'..."
az ad app update --id $appId --required-resource-accesses "[{
        \"resourceAppId\": \"$microsoftGraphId\",
        \"resourceAccess\": [{
                        \"id\": \"$userPermId\",
                        \"type\": \"Role\"
                },
                {
                        \"id\": \"$groupPermId\",
                        \"type\": \"Role\"
                }
        ]
}]"

# Grant admin consent by granting permissions to the service principal - https://learn.microsoft.com/en-us/graph/api/serviceprincipal-post-approleassignments
echo "Granting admin consent on the permissions $userPermName (id '$userPermId') and $groupPermName (id '$groupPermId') for the service principal '$spObjectId'..."
az rest --method POST \
        --uri "https://graph.microsoft.com/v1.0/servicePrincipals/$spObjectId/appRoleAssignments" \
        --body "{
        \"principalId\": \"$spObjectId\",
        \"resourceId\": \"$msGraphResourceId\",
        \"appRoleId\": \"$userPermId\"
        }" 1> /dev/null

az rest --method POST \
        --uri "https://graph.microsoft.com/v1.0/servicePrincipals/$spObjectId/appRoleAssignments" \
        --body "{
        \"principalId\": \"$spObjectId\",
        \"resourceId\": \"$msGraphResourceId\",
        \"appRoleId\": \"$groupPermId\"
        }" 1> /dev/null

echo "Application '$appName' was created successfully with client id '$appId' and client secret '$appSecret'."
echo "App-only permissions '$userPermName' and '$groupPermName' were added, and admin consent granted."
```

{{< /tab >}}
{{< /tabs >}}

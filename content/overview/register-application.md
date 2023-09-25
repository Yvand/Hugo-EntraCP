---
title: "Grant permissions to your tenant"
description: ""
lead: ""
date: 2021-05-20T10:45:06Z
lastmod: 2021-08-06T11:15:29Z
draft: false
images: []
menu:
  overview:
    parent: ""
    identifier: "register-app"
weight: 110
toc: true
---

Follow this article to create an app registration in your Microsoft Entra ID tenant, and grant the permissions that EntraCP needs.

## Permissions required

EntraCP requires permissions `GroupMember.Read.All` and `User.Read.All`, of type application (not delegated):

![Image](assets/images/aad-entracp-permissions.png "At the end of the configuration, the permissions should be exactly like this.")

## Create the app registration

You can register the application using either:

- The [Azure portal]({{< relref "#using-the-azure-portal" >}}).
- [m3655 cli]({{< relref "#using-m365-cli" >}}).
- [az cli]({{< relref "#using-az-cli" >}}).

### Using the Azure portal

1. Sign-in to your [Microsoft Entra ID tenant](https://entra.microsoft.com/).
1. Under "Identity", expand "Applications" and click "App Registrations" > "New registration" > Type the following information:
    * Name: EntraCP
    * Supported account types: "Accounts in this organizational directory only (Single tenant)"
1. Click "Register"
1. Click "API permissions"
    * Remove the default permission.
    * Click "Add a permission" > Select `Microsoft Graph` > "Application permissions", and select `GroupMember.Read.All` and `User.Read.All`.
    * Click "Grant admin consent for TenantName" > Yes
1. Click on "Certificates & secrets": EntraCP supports both a certificate and a secret, choose either option depending on your needs.

### Using m365 cli

[M365 cli](https://pnp.github.io/cli-microsoft365/) makes the registration very simple: It takes a single command to create the application, create a secret, set the permissions and grant the admin consent:

```bash
m365 login
# The command will print all the information that EntraCP needs to connect.
m365 aad app add --name "EntraCP" --withSecret --apisApplication 'https://graph.microsoft.com/User.Read.All,https://graph.microsoft.com/GroupMember.Read.All' --grantAdminConsent
```

### Using az cli

This bash script creates the application, adds a secret, sets the permissions and grants the admin consent.  
It can be used in Azure cloud shell or in a local shell:

```bash
# Sign-in to Azure AD tenant. Use --allow-no-subscriptions if it doesn't have a subscription
az login --allow-no-subscriptions

appName="EntraCP"

echo "Creating application '$appName'..."
az ad app create --display-name "$appName" --key-type Password --credential-description 'client secret'
appId=$(az ad app list --display-name "$appName" --query '[].[appId]' -o tsv)

echo "Creating service principal for application id '$appId'..."
az ad sp create --id $appId
# Get the objectId of the service principal, needed to grant the required permissions
spObjectId=$(az ad sp list --filter "appId eq '$appId'" --query '[].[appId, objectId, appDisplayName]' -o tsv | cut -f2)
echo "Application '$appName' was created with client id '$appId', and its service principal with objectId '$spObjectId'"

# Create a secret
appSecret=$(az ad app credential reset --id $appId -o tsv | cut -f3)

# Retrieve the id of the permissions to grant
userPermId=$(az ad sp show --id 00000003-0000-0000-c000-000000000000 --query "appRoles[?value=='User.Read.All'].id" --output tsv)
groupPermId=$(az ad sp show --id 00000003-0000-0000-c000-000000000000 --query "appRoles[?value=='GroupMember.Read.All'].id" --output tsv)
msGraphResourceId=$(az ad sp show --id 00000003-0000-0000-c000-000000000000 --query "objectId" --output tsv)

# Add the permissions required to the definition of the application (optional as it is just a declaration of the permissions needed)
az ad app update --id $appId --required-resource-accesses "[{
        \"resourceAppId\": \"00000003-0000-0000-c000-000000000000\",
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

echo "Grant admin consent to Microsoft Graph permissions User.Read.All (id '$userPermId') and GroupMember.Read.All (id '$groupPermId') for service principal '$spObjectId'..."
# Wait before granting the permissions to avoid error "Request_ResourceNotFound" on the service principal just created
sleep 20
# Grant permissions to the service principal of the application
az rest --method POST \
        --uri "https://graph.microsoft.com/v1.0/servicePrincipals/$spObjectId/appRoleAssignments" \
        --body "{
        \"principalId\": \"$spObjectId\",
        \"resourceId\": \"$msGraphResourceId\",
        \"appRoleId\": \"$userPermId\"
        }"

az rest --method POST \
        --uri "https://graph.microsoft.com/v1.0/servicePrincipals/$spObjectId/appRoleAssignments" \
        --body "{
        \"principalId\": \"$spObjectId\",
        \"resourceId\": \"$msGraphResourceId\",
        \"appRoleId\": \"$groupPermId\"
        }"

echo "Application $appName was created with client id '$appId' and client secret '$appSecret'"
```

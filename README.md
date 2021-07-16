# Logic App auto invite

Logic app to automatically invite Azure AD synchronized users to B2B collaboration.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fcblomart%2FLogicApp-Invite%2Fmaster%2Fazuredeploy.json)
[![Visualize](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/visualizebutton.svg?sanitize=true)](http://armviz.io/#/?load=https://raw.githubusercontent.com/cblomart/LogicApp-Invite/master/azuredeploy.json)


## Before Deployement

To deploy the template you will need the service principal id of 'Microsoft Graph Change Tracking'

```powershell
# Connect to Azure AD
Connect-AzureAD

# Get the ObjectId of 'Microsoft Graph Change Tracking'
Get-AzureADServicePrincipal -Filter "AppId eq '0bf30f3b-4a52-48df-9a82-234910c4a086'" | Select-Object -ExpandProperty ObjectId
```

## Deployement

Deploy the solution via the Azure Resource Manager (ARM) template.

The solution is composed of different components:
* EventHub: to recieve user change notifications from Azure AD
* KeyVault: to hold the reference of the eventhub for the Azure AD notification
* A managed identity: this identity will be assigned to the logic app to give them appropriate rights to Azure AD
* Subscription Logic App: this logic app is scheduled every day to renew the notification subscription
* Invitation Logic App: this logic app does the invitation of accounts that needs to be invited

The logic apps will be disabled from the start. Before enabling them you will neede to provide the necessary rights (bellow).

## After Deployement

### Managed identity permissions

The logic Apps will run under a common managed identity. This managed identity will need to be assigned the guest inviter role.
The id of the managed identity is given by the deployement.


```powershell
# Connect to Azure AD
Connect-AzureAD

# Get the Guest Inviter role
$role =  Get-AzureADMSRoleDefinition -Filter "DisplayNAme eq 'Guest Inviter'"

# Set the role to the managedIdentity ID recovered in the deployement
New-AzureADMSRoleAssignment -RoleDefinitionId $role.Id -PrincipalId $managedIdentityId -ResourceScope "/"
```

### Enable the workflows !

For the workflows to work you will need to enable them.
1. Enable the subscription workflow so that changes are sent to event hub
2. Enable the invite workflow to invite users
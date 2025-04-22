# Powershell Remoting

### ypes <a href="#types" id="types"></a>

* one to one -> Runs in a new process (wsmprovhost) its state full
* One to many

### One-To-One <a href="#one-to-one" id="one-to-one"></a>

```powershell
New-PSSession

$sess = New-PSSession -ComputerName

$sess = New-PSSession -ComputerName -Credentials 

Enter-PSSession -Session $sess

Enter-PSSession
Enter-PSSession -ComputerName

$password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("test", $password)
Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
```

### One-To-Many <a href="#one-to-many" id="one-to-many"></a>

```powershell
//One Computer
Invoke-Command -ComputerName {} -ScriptBlock{whoami;hostname}

localfunction to remote computer
Invoke-Command -ComputerName {} -ScriptBlock ${function: }

//Multiple Computer
Invoke-Command -ComputerName (Get-content <txt file> ) -ScriptBlock{whoami;hostname}

//Important
Invoke-Command -ComputerName  -FilePath C:\script\Get-PassHashes.ps1
```


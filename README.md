# Nmap

#### Nmap <a href="#nmap" id="nmap"></a>

```bash
# Get an initial idea of the scenario
sudo nmap -sCV -oN nmap/initial -v $IP

# All port scan again with max retries 0 and min rate 5000
sudo nmap -p- -T4 --min-rate 5000 --max-retries 0 -v $IP -oN nmap/ports

# All port scan with 5 threads to list the ports
sudo nmap -p- -T5 -v $IP -oN nmap/ports2

# Once we have to ports do both an aggressive scan and a verbose service scan
ports=$(cat nmap/{initial,ports,ports2} | grep 'open' | cut -d '/' -f 1 | sort -u |sed -z 's/\n/,/g;s/,$/\n/')

sudo nmap -p $ports -A -v $IP -oN nmap/all-ports
sudo nmap -p $ports -sCV -O -oN nmap/all-ports-service -v $IP

# UDP Portscan
sudo nmap -p $ports -sU -A $IP
```

> Reference: https://github.com/21y4d/nmapAutomator

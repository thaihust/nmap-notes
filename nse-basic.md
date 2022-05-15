## Ref
https://nmap.org/book/nse-tutorial.html
https://nmap.org/book/nse-example-scripts.html

## Usecases
- Network Discovery
  - Perform whois lookups, perform additional protocol queries, and act as a client for the listening service to collect information such as available network shares.
https://materials.rangeforce.com/tutorial/2019/12/24/Network-Discovery-Nmap/
https://www.redhat.com/sysadmin/enumerating-network-nmap
https://medium.com/purple-team/network-discovery-and-security-auditing-with-nmap-323432e54ee6

- Version Detection
  - Perform complex version probes and attempt service brute-force cracking.

- Vulnerability Detection
  - Execute probes to check for specific vulnerabilities.

- Malware Detection
  - Execute probes to discover Trojan and worm backdoors.

- Vulnerability Exploitation
  - Execute scripts to exploit a detected vulnerability.

## Examples
1. Troubleshoot network protocol
1.1. DHCP

```sh
nmap -sU -p 67 --script=dhcp-discover 192.168.1.0/24
nmap --script broadcast-dhcp-discover
```

2. Discover OS info via SMB
2.1. Determine the operating system, computer name, domain, workgroup, and current time over the SMB protocol (ports 445 or 139)
- Discover info toàn bộ các host trong 1 mạng:

```sh
nmap --script smb-os-discovery.nse -p445 192.168.1.0/24
nmap --script smb-os-discovery.nse -p139 192.168.1.0/24
```

2.2.  Discover all user accounts that exist on a remote system

```sh
nmap --script smb-enum-users -p445 192.168.192.0/24
nmap -sU -sS --script smb-enum-users -p U:137,T:139 192.168.192.0/24
```

3. Footprinting
- Cho biết ssh auth-method supported

```sh
nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=ubuntu" 125.212.204.217
```

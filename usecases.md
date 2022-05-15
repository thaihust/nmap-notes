
# Nmap Enterprise usecases

## Usecase 1. Using Nmap for Compliance Testing (Network Discovery+Version Detection)

Testing for compliance can be one of the most important detective security controls you perform in a enterprise infrastructure. The purpose of compliance testing is to measure the critical components of the organization to the policies and controls that govern them. Normally this function falls to either an internal or external audit team. An internal team is generally comprised of employees of the organization and perhaps some long-term contractors, while an external team is often part of a managed services or consulting package. The audit team is responsible for conducting compliance testing against controls they have developed that are specific to meeting regulatory and legal requirements. These requirements vary based on the type of business your organization is in (the vertical market), in addition to where your organization is located or does business. International, state and local laws all come into play. It is the audit team's responsibility to stay on top of the latest requirements and also to ensure that compliance testing is done in both an orderly and timely fashion. Much like designing and maintaining the policies themselves, compliance testing requires persistent and ongoing attention. There are many different types of compliance testing where Nmap could be
utilized as part of the solution. Some examples:

- Testing for open ports on the interfaces of a firewall.
- Performing scans across workstation IP address ranges to determine if any unauthorized networking applications are installed.
- Determining if the correct version of web service is installed in your De-Militarized Zone (DMZ).
- Locating systems with open file sharing ports.
- Locating unauthorized File Transfer Protocol (FTP) servers, printers or operating systems.
- Any number of needs specifi c to the controls written around your organization's policies.

Let’s take the example of determining what version of web service is running on the server located in your DMZ. We’ll pull out our trusty Nmap application and use
the Version Scan, –sV, setting:

```sh
nmap –sV host.example.com

Starting Nmap 4.50 (http://insecure.org) at 2007-12-13 19:41 Central
Standard Time
Interesting ports on host.example.com (192.168.10.10):
Not shown: 1686 closed ports
PORT STATE SERVICE VERSION
21/tcp open tcpwrapped
80/tcp open http Microsoft IIS webserver 5.0
135/tcp open msrpc Microsoft Windows RPC
443/tcp open https?
445/tcp open microsoft-ds Microsoft Windows 2000 microsoft-ds
1025/tcp open msrpc Microsoft Windows RPC
1027/tcp open msrpc Microsoft Windows RPC
1433/tcp open ms-sql-s?
2301/tcp open http Compaq Diagnostis httpd (CompaqHTTPServer 4.2)
3389/tcp open ms-term-serv?
49400/tcp open http Compaq Diagnostis httpd (CompaqHTTPServer 4.2)
Service Info: OS: Windows
```

In this example, we see that Nmap believes the server to be running Microsoft IIS 5.0. You can also see a lot of other port information that isn’t really specific to our current question. We’ll discuss how to narrow down our Nmap query in order to facilitate the scan. First though let’s telnet to port 80 on the server and see if Nmap has given us the correct information.

```sh
telnet host.example.com 80
GET/HTTP/1.0
HTTP/1.1 200 OK
Server: Microsoft-IIS/5.0
Date: Wed, 13 Dec 2007 21:24:22 GMT
X-Powered-By: ASP.NET
X-AspNet-Version: 2.0.50727
Cache-Control: private
Content-Type: text/html; charset=utf-8
Content-Length: 9578
```

Keep in mind that it is very easy to mask this information at the server, but if you are checking organization owned assets for version compliance, most likely you have found an outdated system. Now, if you wanted to narrow down your Nmap scan to only check ports 80 and 443 (or any other ports you know your organization might
be using for web-based applications), it is fairly easy to scan specific ports with the -p command.

The most important point to keep in mind when scanning for policy compliance is that you should have an established set of controls that map back to and describe
the particular piece of policy you are checking. As an example, let’s say your organization has a policy mandating the usage of AV (anti-virus) software on all desktops. Depending on the type of anti-virus application that is deployed, you might find that you have an open port on each system running the AV client. By creating a control that describes this port and the fact that it should be present on systems in your Desktop VLANs, you can then utilize Nmap to locate active systems and subsequently query for this specifi c port. The beauty of Nmap and its various output capabilities is that you can script this entire process and end up with a small report of online systems having this AV port. One thing to keep in mind (and this goes for any discovery process) is that an end-user’s workstation could make it onto the "has AV installed" list and not be running the AV client. This happens when users inadvertently or purposely reassign ports to other networked applications. This author once came across the elite port of 31337 (default port for the Back Orifice Trojan) during a scheduled port scan of a small intranet and then discovered that a programmer was beta-testing a new application and had chosen this port because it was "fun to use infamous ports"! Needless to say, the programmer was asked to change the default port setting of the application.

## Usecase 2. Using Nmap for Inventory and Asset Management (Network Discovery+Version Detection)

There are many commercial applications designed to track assets, manage inventory
counts, relay information about installed services, and monitor system uptime. Luckily
for non-commercial application owners, this is another area where Nmap’s ease of use
pays off with succinct results. In a matter of minutes, an administrator can generate a
scan request for a range of IP addresses, an entire subnet, or even re-scan pre-identified
systems. The options for identifying services and Operating System (OS) type come in
handy when you are trying to identify existing desktops or servers in the infrastructure.
Let’s assume you have been tasked with identifying any outdated OS in your
network. Step one is to use Nmap to identify up systems. This will help us narrow
down the number of IP addresses that we have to scan more in-depth. Step two is to
use Nmap to query those systems to determine what OS is installed. We’ll do this in
an Nmap 2-step process first to get used to the idea:
nmap –n -sP 10.0.0.1-10 (ok, it’s a small network)
Starting Nmap 4.50 (http://insecure.org) at 2007-12-13 19:52 Central Standard Time
Host 10.0.0.1 appears to be up.
MAC Address: 00:0F:B5:6C:DE:E0 (Netgear)
Host 10.0.0.2 appears to be up.
MAC Address: 00:02:E3:13:36:4B (Lite-on Communications)
Host 10.0.0.3 appears to be up.
MAC Address: 00:19:C5:D5:70:EA (Unknown)
Host 10.0.0.4 appears to be up.
Host 10.0.0.5 appears to be up.
MAC Address: 00:14:A5:13:17:75 (Gemtek Technology Co.) 
Host 10.0.0.6 appears to be up.
MAC Address: 00:10:A4:7C:22:AF (Xircom)
Host 10.0.0.7 appears to be up.
MAC Address: 00:0C:29:E9:43:0A (VMware)
Nmap finished: 10 IP addresses (7 hosts up) scanned in 1.000 seconds

Here we utilized the –sP parameter to perform a ping scan and determine which
hosts are up on this small ten host network. We also used the –n option to disable
DNS lookups of the IP addresses. This is a common practice to help speed up the
performance of the network mapping scan (although Nmap is extremely efficient,
even when performing DNS lookups). Notice that the 10.0.0.4 host did not report
a MAC address. This is because the scan was performed from this system.
Now let’s use the –oN parameter to write our results to a normal output file, to try
and make it easier to perform step two:
nmap -n -oN up-systems -sP 10.0.0.1-10

If we open the up-systems file in Wordpad (or whatever your text viewer of
choice might be), we find the following (see Figure 2.4)

While this is a great format for viewing the results off-line or at a later point
in time, this does not easily lend itself to our step two. In order to submit a list of
online hosts to Nmap, we need to have just a listing of hosts without any extraneous
information. If you try to submit this list, Nmap will complain that it is unable to
determine what the hosts are:
nmap -sV -iL up-systems
Starting Nmap 4.50 (http://insecure.org) at 2007-12-13 20:47 Central Standard Time
Invalid target host specification: #
QUITTING!
What we need is a nice, well-ordered list that we can work with for our step two
submission to Nmap. Let’s try a different output option to see what impact it has.
In this example, we’ll use the –oG or ‘grepable’ format. This format has been deprecated
but is still very popular for this very reason: It is simple to create a file that can later
be searched and manipulated.
nmap -sP -oG up-systems2 10.0.0.1-10

This produces a report with output that is very easy to read:
### Nmap 4.50 scan initiated Thur Dec 13 22:03:28 2007 as: nmap -sP -oG up-systems2 10.0.0.1-10

Host: 10.0.0.1 () Status: Up
Host: 10.0.0.2 () Status: Up
Host: 10.0.0.3 () Status: Up
Host: 10.0.0.4 () Status: Up
Host: 10.0.0.5 () Status: Up
Host: 10.0.0.6 () Status: Up
Host: 10.0.0.7 () Status: Up
### Nmap run completed at Thur Dec 13 22:03:29 2007 –- 10 IP addresses

(7 hosts up) scanned in 0.922 seconds
At this point, we can simply delete the top and bottom status lines and then use a
combination of cut and tr to cull the IP addresses from our resulting file and create a
new file of only active IP addresses that can be fed into Nmap for our OS scan. As an
example for this file, we can use cut to create a list with only our active IP addresses
in it (see Figure 2.5).
cut -b7-15 up-systems2 > IPs-only

As our final prep step, we’ll use the tr command to delete the carriage returns and
prep our IP address list so that it is ready to be fed into our Nmap OS scan:
tr -d ‘\r’ < IPs-only > Nmap-ready_IPs
If you take a peek into the Nmap-ready_IPs file, you will see the IP addresses are
all on one line, each separated by a space. It’s not very easy to manually read, but this
is the perfect format for Nmap:
10.0.0.1 10.0.0.2 10.0.0.3 10.0.0.4 10.0.0.5 10.0.0.6 10.0.0.7
As another alternative, this single command line will create a CR delimited list of
IP addresses that Nmap can use as an input file:
cat up-systems2 | grep Host | awk ‘{print $2}’ > Nmap-ready_IPs
Now we are ready for our second Nmap step: Let’s run this Nmap-ready_IPs file
as an input file to an Nmap –A scan to detect service and OS versions of these live
hosts. We’ll output the data to a file named OS-Svc-info and then peek into the
contents of the resulting fi le (edited for length) to get our OS info:
Nmap –A –iL Nmap-ready_IPs > OS-Svc-info
Starting Nmap 4.50 (http://insecure.org) at 2007-12-13 23:48 Central
Standard Time
Insufficient responses for TCP sequencing (1), OS detection may be less accurate
Interesting ports on 10.0.0.1:
Not shown: 1694 filtered ports
PORT STATE SERVICE VERSION
23/tcp open telnet?
80/tcp open tcpwrapped
1723/tcp closed pptp
MAC Address: 00:0F:B5:6C:AB:E4 (Netgear)
Device type: remote management|firewall|media device
Running: Compaq embedded, Enterasys embedded, Phillips embedded
OS details: Compaq Inside Management Board, Enterasys XSR-1805 Security Route,
Phillips ReplayTV 5000 DVR
Network Distance: 1 hop
<Author’s Note: This host is a Netgear 54Mps Wireless Router WGR614 v5>
Interesting ports on 10.0.0.2:
Not shown: 1694 closed ports
PORT STATE SERVICE VERSION
135/tcp open msrpc Microsoft Windows RPC
139/tcp open netbios-ssn
1026/tcp open mstask Microsoft mstask (task server - c:\winnt\system32\Mstask.exe)
MAC Address: 00:02:E3:13:47:6B (Lite-on Communications)
Device type: general purpose|firewall|VoIP adapter|specialized
Running (JUST GUESSING) : Microsoft Windows NT/2K/XP|95/98/ME|2003/.NET|PocketPC/
CE (97%), NetBSD (92%), IBM OS/400 V5 (92%), Secure Computing embedded (92%),
Cisco embedded (91%), Ixia embedded (90%), Apple Mac OS X 10.2.X (90%)
Aggressive OS guesses: Microsoft Windows 2000 Professional SP2 (97%), Microsoft
Windows XP Pro SP1/SP2 or 2000 SP4 (95%), Microsoft Windows Millennium Edition
(Me), Windows 2000 Professional or Advanced Server, or Windows XP (94%), Microsoft
Windows 2003 Server or XP SP2 (93%), Microsoft Windows 2000 Professional RC1 or
Windows 2000 Advanced Server Beta3 (93%), Microsoft Windows 2003 Server Enterprise
Edition (93%), NetBSD 1.6.2 (alpha) (92%), IBM AS/400 running OS/400 5.1 (92%),
Microsoft Windows NT 3.51 SP5, NT 4.0 or 95/98/98SE (92%), Secure Computing
Sidewinder firewall 5.2.1.06 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Windows
<Author’s Note: This host is running Windows 2000, SP4>
Warning: OS detection for 10.0.0.3 will be MUCH less reliable because we did not
find at least 1 open and 1 closed TCP port
All 1697 scanned ports on 10.0.0.3 are closed
MAC Address: 00:19:C5:D5:68:EO (Unknown)
Device type: general purpose
Running: NetBSD
OS details: NetBSD 4.99.4 (x86)
Network Distance: 1 hop
<Author’s Note: This is actually a Playstation 3, v. 2.01 on a wireless
connection>
Skipping SYN Stealth Scan against 10.0.0.4 because Windows does not support
scanning your own machine (localhost) this way.
Skipping OS Scan against 10.0.0.4 because it doesn’t work against your own machine
(localhost)
All 0 scanned ports on 10.0.0.4 are
Insufficient responses for TCP sequencing (0), OS detection may be less accurate
<Author’s Note: This is my scanning system and it is a Windows XP SP2 box>
Interesting ports on 10.0.0.5:
Not shown: 1695 closed ports
PORT STATE SERVICE VERSION
135/tcp open msrpc?
912/tcp open ftp vsftpd or WU-FTPD
MAC Address: 00:14:A5:13:23:46 (Gemtek Technology Co.)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop
<Author’s Note: This host is running XP SP2 and connecting wirelessly using an
internal Broadcom 802.11b/g WLAN adapter>
Interesting ports on 10.0.0.6:
Not shown: 1693 closed ports
PORT STATE SERVICE VERSION
135/tcp open msrpc?
139/tcp open netbios-ssn
445/tcp open microsoft-ds Microsoft Windows XP microsoft-ds
1025/tcp open NFS-or-IIS?
MAC Address: 00:10:A4:7C:33:DF (Xircom)
Device type: general purpose|firewall|VoIP adapter|specialized
Running (JUST GUESSING) : Microsoft Windows NT/2K/XP|95/98/ME|2003/.NET|PocketPC/
CE (97%), NetBSD (92%), IBM OS/400 V5 (92%), Secure Computing embedded (92%),
Cisco embedded (91%), Ixia embedded (90%), Apple Mac OS X 10.2.X (90%)
Aggressive OS guesses: Microsoft Windows 2000 Professional SP2 (97%), Microsoft
Windows XP Pro SP1/SP2 or 2000 SP4 (95%), Microsoft Windows Millennium Edition
(Me), Windows 2000 Professional or Advanced Server, or Windows XP (94%), Microsoft
Windows 2003 Server or XP SP2 (93%), Microsoft Windows 2000 Professional RC1 or
Windows 2000 Advanced Server Beta3 (93%), Microsoft Windows 2003 Server Enterprise
Edition (93%), NetBSD 1.6.2 (alpha) (92%), IBM AS/400 running OS/400 5.1 (92%),
Microsoft Windows NT 3.51 SP5, NT 4.0 or 95/98/98SE (92%), Secure Computing
Sidewinder firewall 5.2.1.06 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Windows
<Author’s Note: This is another Windows 2000 SP4 system>
Interesting ports on 10.0.0.7:
Not shown: 1694 closed ports
PORT STATE SERVICE VERSION
22/tcp open tcpwrapped
111/tcp open rpcbind?
631/tcp open ipp?
MAC Address: 00:0C:29:E9:59:DE (VMware)
Device type: general purpose
Running: Linux 2.4.X
OS details: Linux 2.4.22-ck2 (x86) w/grsecurity.org and HZ=1000 patches
Network Distance: 1 hop
<Author’s Note: This is a Vmware box running SuSe Linux 10.0 with a 2.6.13-15kernel>
OS and Service detection performed. Please report any incorrect results at http://
insecure.org/nmap/submit/.
Nmap finished: 6 IP addresses (6 hosts up) scanned in 223.859 seconds

Now you are probably saying “That definitely was not a quick, easy method” and
since our test environment is really just a small, home network, this really is overkill.
However, once you start scanning class C and larger networks, it is often very handy

to have a separate fi le that contains just live host information. This is true both from
an ongoing live hosts comparison perspective and also from the proficiency angle
when you start firing up service and OS scans

## Usecase 3. Using Nmap for Security Auditing (Vulnerability Detection)

Security auditing can be defined as creating a set of controls specifi c to the technology
or infrastructure being reviewed and then applying those controls, like a filter, to your
environment. Any gaps in or outside that fi lter become audit points and could negatively
impact the audit’s overall assessment of your security framework.
Nmap can assist with such audit needs as:
- Auditing firewalls by verifying the firewall filters are operating properly.
- Searching for open ports on perimeter devices (perimeter being anything from Internet-edge, to extranet or intranet boundary lines).
- Performing reconnaissance for certain versions of services.
- Utilizing the OS detection feature to pin-point outdated or unauthorized systems on your networks.
- Discovering unauthorized applications and services.

## Usecase 4. Using Nmap for System Administration

Although it is normally seen as a go-to application for security professionals, its
wide-range of port scanning, service and OS identifi cation capabilities make it perfect
for the system administrator. If you decide to make Nmap available to administrators
outside IT Security, keep in mind that this could increase unwanted scanning activity
in your network. This is a perfect lead-in to our next subject–important security
facets of employing Nmap.

### Securing Nmap

Nmap is a security tool, but it must also be utilized in your infrastructure with
security in mind. Any administrative tool running in your environment, security-related
or otherwise, will require certain policies and procedures to ensure a successful
deployment and operation. When you start specifi cally addressing security-related
tools, you have to be sure to incorporate everything from separation of duties to
principle of least privilege, as well as access tracking and usage reporting.

### Executable and End-User Requirements

As with almost any security-related application, the first things to think about when
starting the installation process includes security of the user context for the application
and what permissions are required to manipulate the executable. Commonly you will
find that the user must have root permissions on a UNIX system and administrator
rights on a Windows box for both application installation and execution. Security best
practices for accountability dictate that in order for administrative access to be properly
tracked, Nmap users must have credentials that are individually identifi able. For example,
John must have a personal use account and an administrative use account, both of
which personally identify John as the account holder. If a common administrative
username is utilized across the team, you have lost all tracking and auditing abilities.
Shared “administrator” or “root” usage can be a hard habit to break; however it only
takes getting caught by one auditing requirement to justify making the break.
This is connected to another important security best practice, the principle of
least privilege. If John’s day-to-day work does not require administrative access, he
should be logged in with his personal use account the majority of time. He must
only switch to the administrative account when and if the details of his work require
those extra access privileges. The theory behind this practice is that by limiting his
access to the administrative account, he is helping to limit exposure to any vulnerability
that might be associated with the use of that account. For example, many worms
have achieved superior results for the simple reason that users were logged on at the
time of infection with higher-than-necessary privilege. There are also ways of limiting
users’ access by properly setting up and utilizing user groups or granting temporary
access via commands like run as in the Windows Active Directory environment.
Access control can also be implemented in the UNIX world via the use of group
permissions and commands like sudo.


## Usecase 5. Malware Detection

- Execute probes to discover Trojan and worm backdoors.

## Usecase 6. Vulnerability Exploitation

- Execute scripts to exploit a detected vulnerability.

# 30DaysofKaliLinux

Day 1 & 2: Environment Setup and Initial Recon

VirtualBox lab: Kali (attacker) + Metasploitable2 (target) on host-only network 192.168.56.0/24. nmap -sn found 4 live hosts; nmap -sV -sC against the target turned up the standard Metasploitable2 service set (vsftpd 2.3.4, Samba 3.0.20, UnrealIRCd, Tomcat manager, MySQL 5.0.51a, root shell backdoor on 1524, and more).

Known exploits (Exploit-DB):

vsftpd 2.3.4 (21) — backdoored, CVE-2011-2523
Samba 3.0.20 (139/445) — CVE-2007-2447, username map script injection
UnrealIRCd (6667/6697) — backdoored, CVE-2010-2075
Tomcat (8180) — default creds, WAR upload RCE

Nmap flags used: -sT connect scan · -sV version detect · -sC default scripts · -O OS detect · -p- all ports · -oN save output · -v verbose

Day 3: DNS & Email Security Recon

Target: zonetransfer.me (DigiNinja's intentionally misconfigured practice domain).

A/NS: resolves to 5.196.105.14, served by nszm1/nszm2.digi.ninja
MX: 7 records, all Google Workspace (ASPMX.L.GOOGLE.COM at priority 0)
TXT: just a Google site-verification string, no SPF
DKIM (google._domainkey): NXDOMAIN, not configured. SOA serial shows the zone hasn't changed since 2019.

Takeaway: no SPF or DKIM, so mail spoofed as this domain sails through basic checks, even though the MX records point at real, live Google mail servers.
Content30 Days of Kali Linux
Day 1 & 2: Environment Setup and Initial Recon
Github Link: https://github.com/dk201105/30DaysofKaliLinux/tree/main/Day1&2
Setup
Installed VirtualBox, Kali Linux (attacker VM), and Metasploitable2 (target VM) on a host-only network, 192.168.56.0/24.
Host Discovery
sudo nmpasted30 Days of Kali Linux
Day 1 & 2: Environment Setup and Initial Recon
Github Link: https://github.com/dk201105/30DaysofKaliLinux/tree/main/Day1&2
Setup
Installed VirtualBox, Kali Linux (attacker VM), and Metasploitable2 (target VM) on a host-only network, 192.168.56.0/24.
Host Discovery
sudo nmpasted

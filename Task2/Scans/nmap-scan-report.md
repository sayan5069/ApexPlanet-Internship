# Nmap Scan Report — Metasploitable2

## Scan Details

| Field | Value |
|-------|-------|
| **Target** | Metasploitable2 VM |
| **Target IP** | 192.168.x.x *(update with actual IP)* |
| **Scan Date** | *(update with actual date)* |
| **Scanner** | Nmap 7.x |
| **Scan Type** | SYN + UDP + Version + OS Detection + Scripts |

## Commands Used

```bash
# Host discovery
nmap -sn 192.168.1.0/24

# Full TCP SYN scan with version and OS detection
nmap -sS -sV -O -sC -p- -T4 -oA metasploitable_full 192.168.1.100

# UDP scan (top ports)
nmap -sU --top-ports 100 -T4 192.168.1.100

# Vulnerability scripts
nmap --script=vuln 192.168.1.100
```

---

## Open Ports — TCP

| Port | State | Service | Version | Notes |
|------|-------|---------|---------|-------|
| 21/tcp | open | ftp | vsftpd 2.3.4 | **Backdoor CVE-2011-2523** |
| 22/tcp | open | ssh | OpenSSH 4.7p1 | Outdated version |
| 23/tcp | open | telnet | Linux telnetd | Cleartext protocol |
| 25/tcp | open | smtp | Postfix smtpd | Open relay possible |
| 53/tcp | open | domain | ISC BIND 9.4.2 | DNS service |
| 80/tcp | open | http | Apache 2.2.8 | Multiple CVEs |
| 111/tcp | open | rpcbind | 2 (RPC #100000) | RPC service |
| 139/tcp | open | netbios-ssn | Samba 3.x | **CVE-2007-2447** |
| 445/tcp | open | microsoft-ds | Samba 3.x | **CVE-2007-2447** |
| 512/tcp | open | exec | netkit-rsh rexecd | Remote exec |
| 513/tcp | open | login | OpenBSD rlogind | Remote login |
| 514/tcp | open | shell | Netkit rshd | Remote shell |
| 1099/tcp | open | java-rmi | GNU Classpath grmiregistry | RMI registry |
| 1524/tcp | open | bindshell | Metasploitable root shell | **Backdoor shell** |
| 2049/tcp | open | nfs | 2-4 (RPC #100003) | NFS share |
| 2121/tcp | open | ftp | ProFTPD 1.3.1 | FTP alternate |
| 3306/tcp | open | mysql | MySQL 5.0.51a | No root password |
| 3632/tcp | open | distccd | distccd v1 | **CVE-2004-2687** |
| 5432/tcp | open | postgresql | PostgreSQL 8.3 | Default credentials |
| 5900/tcp | open | vnc | VNC protocol 3.3 | No auth |
| 6000/tcp | open | X11 | access denied | X11 display |
| 6667/tcp | open | irc | UnrealIRCd | **Backdoor CVE-2010-2075** |
| 6697/tcp | open | irc | UnrealIRCd | IRC SSL |
| 8009/tcp | open | ajp13 | Apache Jserv 1.3 | Tomcat AJP |
| 8180/tcp | open | http | Apache Tomcat 5.5 | Default creds |

---

## Open Ports — UDP

| Port | State | Service | Notes |
|------|-------|---------|-------|
| 53/udp | open | domain | DNS |
| 67/udp | open | dhcpc | DHCP client |
| 68/udp | open | dhcpc | DHCP client |
| 111/udp | open | rpcbind | RPC |
| 137/udp | open | netbios-ns | NetBIOS |
| 138/udp | open | netbios-dgm | NetBIOS |
| 161/udp | open | snmp | SNMP (community: public) |
| 2049/udp | open | nfs | NFS |

---

## OS Detection

```
OS: Linux 2.6.9 - 2.6.33
CPE: cpe:/o:linux:linux_kernel:2.6
```

---

## Critical Findings Summary

| # | Vulnerability | Port | CVE | CVSS |
|---|--------------|------|-----|------|
| 1 | vsftpd 2.3.4 Backdoor | 21/tcp | CVE-2011-2523 | 10.0 |
| 2 | Samba usermap_script RCE | 139/445/tcp | CVE-2007-2447 | 10.0 |
| 3 | UnrealIRCd Backdoor | 6667/tcp | CVE-2010-2075 | 10.0 |
| 4 | Bindshell Backdoor | 1524/tcp | - | 10.0 |
| 5 | MySQL No Root Password | 3306/tcp | - | 9.0 |
| 6 | Tomcat Default Credentials | 8180/tcp | - | 9.0 |
| 7 | VNC No Authentication | 5900/tcp | - | 9.0 |
| 8 | distccd RCE | 3632/tcp | CVE-2004-2687 | 9.3 |
| 9 | Telnet Cleartext | 23/tcp | - | 7.5 |
| 10 | SNMP Public Community | 161/udp | - | 6.4 |

---

## Recommendations

1. **Immediately patch or disable** vsftpd 2.3.4, UnrealIRCd, Samba 3.x
2. **Remove backdoor shells** on port 1524
3. **Set MySQL root password** and restrict remote access
4. **Change Tomcat default credentials** or disable manager interface
5. **Enable VNC authentication**
6. **Replace Telnet with SSH** for all remote access
7. **Change SNMP community string** from "public" to something complex
8. **Apply firewall rules** to restrict access to sensitive ports

---

*Note: This scan was performed on an intentionally vulnerable VM (Metasploitable2) in an isolated lab environment for educational purposes.*

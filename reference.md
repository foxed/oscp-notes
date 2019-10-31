# oscp-notes

here is a collection of commands i often use for penetration testing. i know there are hundreds of OSCP command guides/references, this is mostly for personal use and therefore the explanations for each command will be as fleshed out as i need them to be. also bear in mind that this is not a complete list.

note: sometimes i will use "ip-addr" to indicate where a target ip should go, sometimes 10.xx.xx.xx, etc
some of these commands i gleaned from working within the HTB labs, not just pwk lab work.


**service and safe scripts scan of an ip/network range**

-oN saves the output in nmap formatting (-oA saves output in xml/nmap/grep format)

`nmap -sC -sV ip-addr -oN '/root/Documents/[ip-addr]/initial.nmap'`

**udp scan**

`nmap -sU ip-addr`

**full port scan**

`nmap -p- ip-addr`

**OS fingerprint** 

`nmap -O ip-addr`

**gobuster using default wordlist and scanning for specific statuses**

saves output to gobuster-common.txt

`gobuster -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://10.xx.xx.xx:8080/ -s '200,204,301,302,307,403,500' -o gobuster-common.txt`

**gobuster -- fuzzing for file (vs. dir)***

`gobuster -u http://10.xx.xx.xx -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x aspx -o gobuster-default-aspx.txt`

**gobuster ignore ssl cert**

`gobuster -k`

**various methods of enumerating smb shares/servers**

`nmblookup -A 10.xx.xx.xx`

`enum4linux -A 10.xx.xx.xx`

note: enum4linux is finnicky on the pwk vm

**smbclient**

-L | list  -I | ip-address  -N | no-pass

`smbclient -L \\ralph -I xx.xx.xx.xx -N`

`smbclient //SERVER/share -I xx.xx.xx.xx -N`

**enum kerberos users**

`nmap -p88 --script=krb5-enum-users --script-args krb5-enum-users.realm='CHANGEME.local',userdb=/usr/share/seclists/Usernames/Names/names.txt -oA '/root/Documents/10.xx.xx.xx_88_kerberos' 10.xx.xx.xx`

**obtain hash from kerberos user**

cd /opt/impacket/examples

`python GetNPUsers.py htb.local/ -no-pass -usersfile users.txt -format hashcat -outputfile hashes.forest`




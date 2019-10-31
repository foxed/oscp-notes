# oscp-notes

here is a collection of commands i often use for penetration testing. i know there are hundreds of OSCP command guides/references, this is mostly for personal use and therefore the explanations for each command will be as fleshed out as i need them to be. also bear in mind that this is not a complete list.

## enumeration

service and safe scripts scan of an ip/network range

`nmap -sC -sV ip-addr`

udp scan

`nmap -sU ip-addr`

full port scan

`nmap -p- ip-addr`

OS fingerprint 

`nmap -O ip-addr`

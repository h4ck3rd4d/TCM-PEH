# Active Directory Initial Compromise

---
### Links
#### [Top 5 got Domain Admin](https://adam-toscher.medium.com/top-five-ways-i-got-domain-admin-on-your-internal-network-before-lunch-2018-edition-82259ab73aaa)
#### [mitm6](https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/)
#### [DirkJanm Blog for ntlm relay and kerberos delegation](https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/)
#### [Pentester's Guide to Printer Hacking](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack/)
---

---
### Tools
---

#### responder
#### impacket
#### mitm6
#### ntlmrelayx.py


---
## LLMNR Poisoning
---

>What is LLMNR and NBT NS poisoning?
>When a windows host cannot resolve a hostname using DNS, it uses the LLMNR protocol (Link-Local Multicast Name Resolution) to ask neighboring computers about it. ... When LLMNR/NBT-NS is used to resolve a name, any host on the network can reply. So, Responder is one of such tools that poisons the request.

#### MITM type attack.
When clients DNS resolution fails, attacker responds and says "I know where 'http://example.com' is. Send me your hash and I'll connect you"
Can be a wrong domain name like googgle.com, or an incorrect network drive.
Using responder, attacker listens on network for a failed dns event where responder can reply and victim will send attacker ntlm hash.

Try and crack hash offline for further lateral/vertical movement

### Attack Steps
- Turn on responder
- Wait for an event and captured hash
- try to crack hash offline

#### LLMNR Defense
Disable LLMNR and NBT-NS
If unable to disable LLMNR and NBT-NS, require Network Access Control and require strong passwords, (>14 characters, the more complex the harder to crack)


---
## SMB Relay
---

### Definition
Instead of cracking hashes offline, relay hashes to specific mains to attempt to gain access

- SMB signing must be disabled on target
- Relayed creds must have admin rights on target

### Attack Steps
- In responder.conf turn off SMB and HTTP
- Run Responder ``` responder -I eth0 -rdw ```
- run ntlmrelayx ``` ntlmrelayx.py -tf targest.txt -sbm2support ```
- if succesful, will obtain SAM file (similar to /etc/shadow on linux)
	- can attempt to crack hashes or try to pass hash
	- ntlmrelayx get interactive shell with -i , -e meterpreter.exe, -c "command" 

### ID hosts with SMB signing disabled
- Nessus scan
- nmap via ``` nmap --script=smb2-security-mode.nse -p 445 <ip range> ```
	- looking for ``` message signing enabled but not required ```

### Mitigations
- Enable SMB signing on all devices
	- pro: completely stops attack
	- con: can cause performance issue
- Disable NTLM auth
	- if kerberos fails, Windows defaults back to NTLM
- Account Tiering
- Local admin restriction


### Gain Shell
- run  metasploit, and use psexec
	- may fail first time or two, and try different targets

- psexec.py as alternate
	- ``` psexec.py domain/user:password@<ip> ```
- smbexec.py or wmiexec.py as options


---
## IPv6 Attacks
---

#### What is IPv6 attack
Similar to other attacks, attacker acts as DNS for ipv6 requests and tricks victim machine into sending hash to attacker
> "If an attacker is on the local network, either physically (via a drop device) or via an infected workstation, it is possible to perform a DNS takeover using mitm6, provided IPv6 is not already in use in the network. When this attack is performed, it is also possible to make computer accounts and users authenticate to us over HTTP by spoofing the WPAD location and requesting authentication to use our rogue proxy."
> Dirk-jan Mollema [blog article](https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/)

#### Steps
- ``` mitm6 -d <domain> ```
- ``` ntlmrelayx.py -6 -t ldaps://<ip> -wh fakewpad.domain -l lootfile ```
- wait

With normal user, mitm6 can grab AD info such as groups, policies, users, computers etc...
If admin logs in, mitm6 can create new user with Admin rights.

---
## Passback Attacks
---

Pose as LDAP server, when user authenticates to MFP, attacker recieves credentials

> "The stored LDAP credentials are usually located on the network settings tab in the online configuration of the MFP and can typically be accessed via the Embedded Web Service (EWS). If you can reach the EWS and modify the LDAP server field by replacing the legitimate LDAP server with your malicious LDAP server, then the next time an LDAP query is conducted from the MFP, it will attempt to authenticate to your LDAP server using the configured credentials or the user-supplied credentials."
> Taken from this [article](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack)

### Default mfp creds
- Vendor:Username:Password
	- Ricoh:admin:blank
	- HP:admin:admin or blank
	- Canon:ADMIN:canon
	- Epson:EPSONWEB:admin

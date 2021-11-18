# Active Directory Initial Compromise

---
## Links
[Top 5 got Domain Admin](https://adam-toscher.medium.com/top-five-ways-i-got-domain-admin-on-your-internal-network-before-lunch-2018-edition-82259ab73aaa)
---

---
## Tools
---

### responder
Usage: responder -I eth0 -w -r -f

### impacket



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


#### LLMNR Defense
Disable LLMNR and NBT-NS
If unable to disable LLMNR and NBT-NS, require Network Access Control and require strong passwords, (>14 characters, the more complex the harder to crack)


---
## SMB Relay
---

### Definition
Instead of cracking hashes offline, relay hashes to specific mains to attempt to gain access

--SMB signing must be disabled on target
--Relayed creds must have admin rights on target

### Setup Attack
- In responder.conf turn off SMB and HTTP
- Run Responder



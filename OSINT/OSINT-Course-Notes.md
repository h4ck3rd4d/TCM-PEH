# Course notes from TCM-Security's OSINT Course

---
## Sock Puppet
---
- a fake persona used for investigation purposes. Should have no connection to your real identity.
	- fake name, fake picture (thispersondoesnotexist.com)
	- burner phone for 2FA
	- different IP address, different laptop
	- privacy.com for using credit/debit cards not tied to your real self

---
## Search Engine OSINT
---
- Search engines Google, Bing, Yandex(best for images), DuckDuckGo, Baidu(asian searches)

---
### Search Parameters (Google Dorking)
---
- site:domain   ex site:reddit.com returns results only from reddit.com
- "strict search" putting quotes around search paramter searches for exact phrase
- AND, OR ex this AND this OR that  returns results where both this' are true, if not then the OR condition
- "*" the wildcard operator     ex "Heath Adams" the * mentor
- filetype:"type"   pdf,xlsx,csv,txt,docx
- site:domain -subdomain excludes results with -subdomain listed ex, site:tesla.com -www returns ir.tesla.com shop.tesla.com etc..
- intext:"text" search for specific text
- inurl:"text" specific text in a url
- intitle:"text" specific text in title of page 

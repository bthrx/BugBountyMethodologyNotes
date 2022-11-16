# Recon/Analysis
## Roots
gather root domains
### Acquisitions
www.crunchbase.com
### ASNs
http://bgp.he.net
Enumerate with Amass
```shell
amass intel -asn <asn numbers>
```
### Reverse WHOIS
```shell
amass intel -whois
```
### Google Dorking
inurl:site 
Copyright text
Terms of Service Text
Privacy Policy Text
### Shodan
www.shodan.io/search?=target.tld

## Subdomains
### Linked/JS Discovery
SubDomainizer
```shell
python3 ~/SubDomainizer/SubDomainizer.py <website>
```
### Scraping
Amass (add asn, cidr, etc, and feed new asn's back into amass intel -asn)
```shell
amass enum <site> -config /path/to/amass/config.ini
```
Project Discovery Chaos(api key is in Amass but just in case)
```shell
choas -d <domain> -key <key>
```
Subfinder
```shell
subfinder -d <website>
```
gau
```shell
gau <website>
```
Github Subdomains
```shell
github-subdomains -d <domain> -t <github token>
```
Cloud_Enum(run later with goaltdns mutations)
```shell
cloud_enum.py -k [keyword(usually company or domain name)] -m
```
### Bruteforce
##### Use wordlists from [[Wordlists]]
and generate own
Amass
```shell
amass enum -brute -d <domain>  -src -w wordlist.txt -rF dnsresolvers.txt
```
ffuf
```shell
ffuf -u FUZZ.domain.tld -w wordlist.txt
```
##### Generate wordlist and mutations
Goaltdns (use -w to use a custom permutations generator list)
```shell
goaltdns -l <list of hosts gathered so far> 
```
Browse site with Golden Nuggets burp extension to generate wordlist
use gau and wordlistgen to make even more site specific wordlist
```shell
echo website.tld | gau | wordlistgen | sort -u
```

##### Rerun tools with new custom wordlist
```shell
ffuf -u FUZZ.domain.tld -w <mutations>
```
```shell
feroxbuster -u -w <wordlist> -d 0
```
Cloud_enum again with mutations and generated wordlists
```shell
cloud_enum -kF <path with generated wordlist> -m <add path if you want to use custom mutations else it will use fuzz.txt>
```
Search Dorking (take previous found subdomains and add them as -www.website.com)
     site:website -www.website.com
     site:website -anothersubdomain.website.com -www.website.com
##### Port Scan
dnmasscan
```shell
dnmasscan subdomainsanddomains.txt -oG <path for output>
```
nmap from dnmasscan
```shell
nmap 
```
##### Screenshotting websites
Eyewittness/Aquatone/HTTPscreenshot (any new options in this space?)
##### Misc
nuclei scan subdomains, misconfigs, buckets, token leaks, etc (custom and templates from https://github.com/projectdiscovery/nuclei-templates)
```shell
nuclei 
```

## Technology
Browse site with wappalyzer, whatruns, and trufflehog  extensions turned on

## Finding CVE's and Misconfigs
Run Nuclei with default templates
```shell
nuclei -l domains.txt -t brute-force/* -t cves/* -t basic-detections/* -t dns/* -t files/* -t panels/* -t security-misconfigurations/* -t subdomain-takeover/* -t technologies/* -t tokens/* -t vulnerabilities/*
```
Create custom nuclei templates as well
Jaeles Scanner
```shell
jaeles scan -s <signatur you want to use> -u domain.tld
```
https://github.com/jaeles-project/jaeles-signatures
## Content Discovery
### Directories
```shell
ffuf -u <domains> -w ~/'Assetnote Wordlists'/httparchive_directories_1m_2022_07_28.txt -w <path to custom wordlist> -recursion -recursion-strategy greedy -c -of md -o <path to output in vault>
```

```shell
cat ~/Desktop/bthrxvault/bthrx/PWST/'Juice Shop'/In-scope/In-scope-domain  
s.md | feroxbuster --stdin -d 0 -w ~/'Assetnote Wordlists'/raft-large-directories.txt -o ~/Desktop/bthrx  
vault/bthrx/PWST/'Juice Shop'/Output/feroxbuster_directories.md --force-recursion --silent -e
```

### Use Wordlists based on tech
Fuzz tech specific wordlists on appropriate domains/endpoints [[Wordlists]] (especially asset note lists)
```shell
ffuf -u domains -w <tech wordlists>
```
```shell
ferox -u <site> --auto-tune -d 0 -w <path to specific wordlists based on recon>
```

### APIs
```shell
ffuf -u <domains> -w ~/'Assetnote Wordlists'/httparchive_apiroutes.txt -w <path to custom wordlist from goldennuggets.py -recursion -recursion-strategy greedy -c -of md -o <path to output in vault>
```
Use Kiterunner for APIs
```shell
kr
```
Pay attention for Config files
Look for admin logins
If you find open source technology, install technology and use source2URL to map every path in a default install.
```shell
./source2url.sh
```
### COTS/Paid
Try and get demo or demo call to get idea of how it works
### Custom
Use burp extension Scavenger to create worldist from burp history
### Historical
```shell
echo website.tld | gau | wordlistgen | sort -u
```
### Recursive
Always do recursion on 401 errors
### Mobile APIs
Use APKLeaks to get mobile API's
```shell
apkleaks -f /path/to/apk
```
## Spidering
### ZAProxy
Right click domain, Select Attack => Spider
### Burp
Right click domain, Scan, Crawl, Configure 
to Never stop crawl due to application errors
### GoSpider (infinite recursion)
```shell
gospider -s <webdomain> -d 0
```
### Hakrawler
```shell
echo <website> | hakrawler
```
## JavaScript
xnLinkFinder to parse inline JS instead of .js files
```shell
xnLinkFinder -i <URL or textfile of URLs or Burp XML output file> -o /path/to/output.md -d 2
```
Use GAP burp extenstion which finds both parameters and paths
https://github.com/xnl-h4ck3r/burp-extensions
Check retired JS
Retire JS or inside Burp pro
## Parameter Analysis
Use GAP burp extension to pull params from site history
Fuzz with ffuf
```shell
ffuf -u website.com/?param=FUZZ -w /path/to/param/fuzzing/wordlist.txt
```
Use GF with gf-patterns to identify exploits in params and more
```shell
cat urls.txt | gf <pattern.gf name>
```
## Appliction Analysis
### Questions to ask
#### How does the app pass data?
resource?parameter=value&param2=value
or
Method -> /route/resource/sub-resource/
or
/graphql?
#### Where and how are users appear?
Understand how users are referenced and where in the application for [[Broken Access Control]] [[Authentication]] [[Business Logic Vulns]][[Information Disclosure]] [[IDOR]]
##### Users
###### How
UID
Email
username
UUID
###### Where
Cookies
API calls

#### Does the site have multi-tenancy or user levels?
Dictate how to test for authorization and access bugs
##### Users
- App is designed for multiple customers
- App has multiple user levels
	- Admin
	- Tenant/Account Admin
	- Tenant/Account User
	- Tenant/Account Viewer
	- Unauthenticated functionality
#### Does this site have a unique threat model?
If app houses more than just PII data, it is easy to forget to target that data in your testing. IE API keys, application data, etc. User data can be considered useless or useful depending on threat model of app.
#### Has there een past security research & vulns?
Test disclosed reports and vulns for regression in app
#### How does the app handle:
If using a framework, figure out how the framework handles input and output  sanitization of dangerous characters and if custom code test it yourself)
[[XSS]] [[CSRF]] [[SQLi]] [[Template Injection]]
## Mapping out places for vulns
Analyze what parts of the app could be vulnerable. Example:
#### Upload Functions
##### Integrations (such as Facebook Profile/Instagram images)
[[XSS]]
##### Self Uploads
###### XML based (docs/pdf)
[[XXE]] [[SSRF]] [[XSS]]
###### Image
[[XSS]] [[RCE]]
Name
Binary Header
Metadata
##### Where is data stored?
###### s3 perms
#### Content Types
##### Look for Multi-part forms
##### Look for content type XML
##### Look for content type json
#### APIs
##### Methods
#### Accounts
##### Profile
###### Stored XSS [[XSS]]
##### App Custom Fields
##### Integrations
[[SSRF]] [[XSS]]
#### Errors
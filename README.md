A set of steps to follow during the reconnaissance stage in external assessments/penetration tests.

# General Information
## Know your target
- Find the company's main domain
 - Understand what they do and what industry they are in
 - Understand approximately what the number of employees is 
 - Find out where they are located throughout the world

 **How** 
 - Company Website
 - LinkedIn

## Company Acquisitions, Mergers, and Subsidiaries

One of the most important tasks - find out as much about acquisitions, mergers, and subsidiaries belonging to the target company,  so we can widen our attack surface.

**How**

 - If it's a public company -  go to investor relation on their website, and look for financial/quarterly/annual reports, most of the time it will be listed there.
 - Crunchbase
 - Company Website(look for the news section)
 - Searching on Google
 - Company PR platforms(e.g. LinkedIn)

## General Tech Used
- Find what mail server they use
- Find what CMS they use
- Find what CRM they use

**How**
- DNS records
- Wappalyzer
- BuiltWith 
- DMARC & SPF records
- LinkedIn job posts

 # Domains
Along with finding subsidiaries, this is one of the most important tasks we have - we want to find as many domains belonging to the target company as possible, so that we can find as many assets as we can to scan later on.

### Reverse WHOIS
Run a WHOIS lookup of the domains you have, try to find the:
-  Registrar name 
- Company name 
- HQ address
- Registrar email

After we find some of these fields with values, we run them in one of the tools to find other domains registered using them.

**Tools**

whoisxmlapi, reversewhois.io, whoxy.com

**Note:** Reverse WHOIS is not nececerally accurate and can have false positives, so validate what you find.

### Reverse NS Lookup
If the company has its unique nameserver(e.g. ns1.tesla.com) we can perform a reverse lookup on its IPs to find all other domains that use this name server.

**Tools**

SecurityTrails

**Note:** If it doesn't belong the company, we still perform the reverse lookup and look for keywords that we think can be belonging to the target company, but beware this is not always accurate and requires validation.

### Reverse IP Lookup
We perform an IP reverse lookup on IPs we know that belong to the target company.
We can also run a reverse lookup on the company's hosting ips if it doesnt belong to them and search for keywords, exactly like in reverse ns lookup, but again - beware that this is not always accurate.

**Tools**

SecurityTrails

### Ad/Analytics Relationship 
BuilthWith

### Favicon Hashing
Shodan

# Subdomains
We want to find as many subdomains as we can to have as many assets to scan as we can.

    subfinder -silent -s all -dL domains.txt -o subdomains.csv

> - We can also use amass and assetfinder
> 
> - Don't forget to insert your API keys in the subfinder config file

# IP Ranges
We want to find as many IP ranges that belong to the company/are used by it to widen our attack surface.
We primarily want:
1. IP ranges the company owns
2. Any other ranges they have(hosted)

**How**
-  RIR Databases (RIPE, ARIN, AFRINIC, APNIC, LACNIC)
- Databases/tools that collect ranges of IPs from the different databases -  networksdb.io (Collects ranges from different databases)
- ASNs - [https://bgp.he.net/](https://bgp.he.net/)
- WHOIS - We can query the databases servers and get every range the company has:
  
`    whois -h whois.arin.net <company>
`
- For ranges that are not registered under the company name, we can use a normal WHOIS query on the domains/subdomains/IPs and find the range in the WHOIS response.
- Tools/websites that regularly scan and index cloud IPs by company - **kaeferjaeger** 

## Scans
By now, you should have one or a few asset files, containing domains, subdomains and IP ranges that belong to your target company.
Using these assets, we will start scanning to find an opening that will allow us to compromise the system, this is the order I like to do things.

1. **Shodan** - I like to start with a passive Shodan port scan, to start understanding the company and making sure I don't miss any open ports when active scanning due to IP blocks.
2. **Nmap** - The next step would be to scan all domains, subdomains, and IP ranges we found for open ports and services using nmap:
    `nmap -Pn -sV -iL file.txt -oA scan_results`

**Note:** We can also run --script vuln to execute all scripts that scan for vulnerabilities, if you have too much assets to scan, do it first for interesting interfaces that you suspect are vulnerable. 

3. **gowitness**- We will feed our nmap .xml file to gowitness to take screenshots of all the interfaces we scanned that have open ports:
    `gowitness nmap -f nmap_output_file.xml --open`
 
 4. **Nuclei** - we will scan our assets for critical and high vulnerabilities first, then if nothing is found - continue to the rest.

4. **Nessus** - we will scan all assets using the Nessus "basic network scan" on all ports and web vulnerabilities. 

Note: If you have a large volume of assets and can deal with it, I recommend executing some of the scans simultaneously and not according to order to save time.
Example: Execute the Nessus scan, then passively find open ports using Shodan and manually review it while Nessus does its thing.

**It goes without saying that:**
- You need to have permission to perform a scan
- The type of scan you want to use depends on how loud you want to be.

You can also scan for misconfigured certificates using:

    tlsx -u host -p port -wc -ss -un -mm -re -ex

# Files, GitHub Secrets, Cloud Assets/Resources
### Files/pictures containing sensitive data, either within their **content**  or **metadata**, can reveal emails, usernames, credentials, domains, service versions, secrets, code security issues, etc.

**How**
1. Dorking
2. Directory Enumeration (brute-forcing using wordlists)
3.  Mobile Application & Application Analysis
4. Metadata Analysis 
5. Pictures taken by the employees/in company locations(from Google/social media)

**Tools**

- Google, Yandex, DuckDuckGo
- Dirb, dirsearch
- MobSF (mobile), bearer, semgrep
bandit for Python code(didn’t check this one yet)
- MetaFinder, pymeta, and Metagoofil to find metadata and some software data, Exiftool to analyze deeper
- Pictures - Google(images and maps - company location), Facebook, Twitter, LinkedIn

### GitHub secrets, such as credentials, API keys and encryption keys. 

**GitHub Dorking**
1. Using tools like trufflehog
2. Manually using a list of keywords:
```password
dbpassword
dbuser
access_key
secret_access_key
bucket_password
redis_password
root_password
Private Key
user
key
cert
ca
private
ssh
smtp
ftp
api
```
# Employee Information
When we reach a point where we have a lot of interesting login interfacfes, we would probably want to initiate a password spray. For that, we will need a list of employees + their emails.

**How**
 - Creating a list of employees using LinkedIn, finding the company’s mail pattern, and generating a list of emails using a script.
 - SMTP user enumeration
- Dedicated sales/marketing tools like ZoomInfo

**Tools**
- CrossLinked
Example: `crosslinked -f {first}{last}@domain.com 'company_name`’

**A final and important note:**
We first gather everything that we scan(in case we are time-sensitive). That means main domains, their subdomains, and IP ranges.


To be continued...

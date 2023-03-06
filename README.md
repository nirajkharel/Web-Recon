
## Subdoman Enumeration
**Subfinder**
- Configure the necessary API Keys on ~/.config/subfinder/provider-config.yaml
  ```bash
  subfinder -d domainname.com -o subfinderoutput
  
  # Active Scanning
  subfinder -d domainname.com -nW -r -rL resolver.txt -o subfinderoutputactive 
  ```

**Findomain**
  ```bash
  findomain -t domain.com -o findomainoutput
  ```

**Assetfinder**
  ```bash
  assetfinder --subs-only domain.com | assetfinderoutput
  ```

**Amass**
- Configure the necessary API Keys on ~/.config/amass/config.ini
  ```bash
  amass enum -d domain.com -config ~/.config/amass/config.ini
  
  # Passive Scanning
  amass enum -d domain.com -passive -config ~/.config/amass/config.ini
  
  # Active Scanning
  amass enum -d domain.com -active -config ~/.config/amass/config.ini
  
  # Resolve IP as well
  amass enum -ip -d domain.com -active -config ~/.config/amass/config.ini
  ```
**Chaos**
- Download the API Key from https://chaos.projectdiscovery.io
- Open Bashrc or Zshrc file and export the API Key
  - `export CHAOS_KEY=`
  ```bash
  chaos -d domain.com -silent -o chaosoutput
  ```
**Github Subdomains**
- Login into github and grab the API Keys.
- Install [Github-Subdomain Enumeration tool](https://github.com/gwen001/github-subdomains)
  ```bash
  go install github.com/gwen001/github-subdomains@latest
  
  # Open Bashrc or ZSHrc file and insert
  export GITHUB_TOKEN=<token>
  
  # Execute the command
  github-subdomains -d example.com -o githuballoutput.txt
  
  # Show only subdomains
  github-subdomains -d example.com -raw -o githubonlysubdomains.txt
  ```
**Sorting Results**
- Sort the result with `sort` command
  ```bash
  sort subfinderoutput findomainoutput assetfinderoutput chaosouput amassaoutput > sortedsubdomains
  cat sortedsubdomains | httprobe | tee alivesorteddomains
  ```
  
**Visualize Subdomains**
- Make a directory to store the aquatone result and run the aquatone tool.
  ```bash
  mkdir aquatoneresults
  cat alivesorteddomains | aquatone -out aquatoneresults
  ```
## Subdomain Enumeration - Extras
**Subject Alternative Name (SAN)**
- Download the Python script from [Here](https://github.com/appsecco/the-art-of-subdomain-enumeration/blob/master/san_subdomain_enum.py)
  ```bash
  wget https://github.com/appsecco/the-art-of-subdomain-enumeration/blob/master/san_subdomain_enum.py
  python san_subdomain_enum.py domainname.com
  ```
**Content Security Policy**
- Content-Security-Policy is a HTTP header, which allows you to create a whitelist of sources of trusted content, and instructs the browser to only execute or render resources from those sources. There is a chance that subdomains can be listed on it.
  ```bash
  curl -s -I -L "https://domain.com/" | grep -Ei '^Content-Security-Policy:' | sed "s/;/;\\n/g"
  ```
## ASN Enumerations
- It is a group of IP/Network that has own routing policies.
- Enumerating with [asn cmd](https://github.com/nitefood/asn)
  ```bash
    wget https://raw.githubusercontent.com/nitefood/asn/master/asn
    chmod +x asn
    sudo mv asn /usr/local/bin

    # Usage
    asn -d domainname.com
    asn -d IP
    asn -d <asn-number>
  ```
- Enumerating with amass
  ```bash
  amass intel -asn <asn-number>
  ```
- Enumerating with Netcraft
  - Insert URL on the below url parameter
  - https://sitereport.netcraft.com/?url=
  - Click on Site Report

## Subdomain Bruteforcing
**GoBuster**
  ```bash
  # Installation
  sudo apt install gobuster

  # Uses DNS subdomain enumeration mode
  gobuster dns -d domain.com -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

  # Users VHOST enumeration mode
  gobuster vhost -u domain.com -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
  ```

**FFUF**
  ```bash
  # Installation
  go install github.com/ffuf/ffuf/v2@latest

  # Usage
  ffuf -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -u https://domain.com -H "Host:FUZZ.domain.com"
  ```

**WFUZZ**
  ```bash
  # Installation
  sudo apt install wfuzz

  # Usage: We can brute force the subdomain with sub-figher module.
  wfuzz -c -f sub-fighter -u 'https://domain.com' -H "Host: FUZZ.domain.com"  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
  ```

## DNS Resolvers and BruteForce
**DNS Validators**
- It is a tool to create a list of DNS resolvers.
- Install the tool from [DNSValidator](https://github.com/vortexau/dnsvalidator)
  ```bash
  # Installation
  git clone
  https://github.com/vortexau/dnsvalidator
  cd dnsvalidator
  sudo python3 setup.py install
  
  #Usage
  dnsvalidator -tL https://public-dns.info/nameservers.txt -threads 20 -o resolvers.txt
  ```
  
**ShuffleDns**
- It can be used for resolving the subdomains and bruteforcing as well and is a wrapper of massdns.
- Resolving the subdomains
  ```bash
  shuffledns -d domain.com -list subdomains.txt -r resolvers.txt -o output.txt
  # Here the resolvers can be acquired from above dns resolver tool and subdomains.txt can be acquired from above subdomain enumeration tools.
  ```
- Bruteforcing the subdomains
  ```bash
  shuffledns -d domain.com -w subdomainwordlists.txt -r resolvers.txt -o output.txt
  # It performs the brute force using the wordlist provided and use resolvers.txt to resolve the subdomains.
  ```
**PureDNS**
- It is a fast domain resolver and subdomain bruteforcing tool that can accurately filter out wildcard subdomains and DNS poisoned entries.
- Resolving the Subdomains
    ````bash
    puredns resolve subdomains.txt --resolvers resolvers.txt

    # Only print resolved subdomains (Quiet mode)
    puredns resolve subdomains.txt --resolvers resolvers.txt -q | tee resolved.txt
    ````
- Bruteforcing the subdomains
  ```bash
  puredns bruteforce wordlists.txt domain.com
  
  # List of domains
  puredns brutefoce wordlists.txt domains.txt
  ```

## Enumerate the Technologies 
**Wappalyzer**
- It is a browser extension which detects the technologies used on the application.
- For Firefox: https://addons.mozilla.org/en-US/firefox/addon/wappalyzer/
- For Chrome: https://chrome.google.com/webstore/detail/wappalyzer-technology-pro/gppongmhjkpfnbhagpmjfkannfbllamg

**NetCraft**
 -It is a browser extension mainly used for identifying malicious and phishing pages but can also detect the technologies used on the application.
- For Firefox: https://addons.mozilla.org/en-US/firefox/addon/netcraft-toolbar/?src=external-apps-download
- For Chrome: https://chrome.google.com/webstore/detail/netcraft-extension/bmejphbfclcpmpohkggcjeibfilpamia

**WhatWeb**
  ```bash
  whatweb domainname.com
  ```
**WhoIs**
  ```bash
  whois domainname.com
  ```
## Directory, URL & Parameter Enumeration
**GAU - Get All URLs**
  ```bash
  # Installation
  go install github.com/lc/gau/v2/cmd/gau@latest
  
  # Usage
  echo domainame.com | gau -o output.txt
  
  # Match Specific response code
  echo domainname.com | gau --mc 200,500
  
  # Include subdomains of target domain
  echo domainname.com | gau --subs
  ```

**GoSpider**
  ```bash
  # Installation
  go install github.com/jaeles-project/gospider@latest
  
  # Usage
  gospider -s domainname.com
  gospider -S domainnameList.txt
  ```
**Katana**
- Fast and Fully configurable web crawling
  ```bash
  # Installation
  go install github.com/projectdiscovery/katana/cmd/katana@latest
  
  # Basic Usage
  katana -u https://domain.com
  
  # Crawl JS file and enable crawling of known files
  katana -u https://domain.com -js-crawl -known-files all
  ```
  
**Arjun**
- Arjun can find query parameters for URL endpoints.
  ```bash
  # Installation
  pip3 install arjun
  
  # Single URL
  arjun -u https://api.example.com/endpoint
  
  # Specify HTTP method
  arjun -u https://api.example.com/endpoint -m POST
  
  # Export
  arjun -u https://api.example.com/endpoint -oT output.txt
  ```
- Arjun supports importing targets from BurpSuite, simple text file and raw request files. Arjun can automatically identify the type of input file so you just need to specify the path.
  ```bash
  # Uncheck the "base64" option while exporting items in Burp Suite
  arjun -i targets.txt
  ```
**Paramspider**
- Finds parameters from web archives, subdomains without interacting with target host.
  ```bash
  # Installation
  git clone https://github.com/devanshbatham/ParamSpider
  cd ParamSpider
  pip3 install -r requirements.txt
  
  # Basic Usage
  python3 paramspider.py --domain domainname.com --output output.txt
  
  # Find Nested Parameters
  python3 paramspider.py --domain domainname.com --level high
  
  # Using with a custom placeholder text (default is FUZZ), e.g don't add a placeholder
  python3 paramspider.py --domain domainname.com --placeholder ''
  ```
  
## Directory BruteForcing
**GoBuster**
  ```bash
  gobuster dir -u https://domainname.com -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
  ```
**DirSearch**
  ```bash
  # Installation
  pip3 install dirsearch

  # Usage: Basic
  dirsearch -u https://domain.com -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt

  # Filter Result and Extensions
  dirsearch -u https://domain.com -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -e php,aspx -i 200 -o output.txt
  ```
  
**FFUF**
  ```bash
  ffuf -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u https://domain.com/FUZZ
  ```

**WFUZZ**
  ```bash
  wfuzz -c -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt https://domain.com/FUZZ
  ```

**FeroxBuster**
  ```bash
  # Installation
  sudo apt install -y feroxbuster

  # Usage
  feroxbuster -u https://domain.com -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt

  # For Specific extension
  feroxbuster -u https://domain.com -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x pdf -x js,html -x php txt json,docx
  ```

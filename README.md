# One-liner Pentesting
Penggunaan 1 baris command untuk melakukan uji penetrasi dan bug bounty.
## 1. Passive Subdomain Enumeration
Scrapes subdomains from Certificate Transparency logs using crt.sh
```bash
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value' | sort -u
```

## 2. Subdomain Discovery + Live Host Filtering
Finds subdomains and checks which ones are live using HTTP(S).
```bash
subfinder -d target.com -silent | httpx -silent
```

## 3. Directory Brute-Forcing with ffuf
Finds hidden directories and files with common wordlists.
```bash
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -mc 200,204,301,302,403
```

## 4. JavaScript Endpoint Discovery
Extracts URLs from JS files to look for API endpoints or keys.
```bash
curl -s https://target.com/app.js | grep -oE 'https?://[^"]+'
```

## 5. JavaScript URL Parameter Mining
Finds parameter-style patterns which could lead to query injection points.
```bash
curl -s https://target.com/app.js | grep -oP '\w+=\w+'
```

## 6. GitHub Dorking for Secrets
Clones potential repos linked to the organization. Combine with trufflehog or gitleaks.
```bash
github-subdomains -d target.com | xargs -I{} sh -c 'echo {}; git clone https://github.com/{}'
```

## 7. CORS Misconfig Check
Checks if CORS is misconfigured to allow arbitrary origins.
```bash
curl -H "Origin: https://evil.com" -I https://target.com | grep "Access-Control-Allow-Origin"
```

## 8. JS File Map Discovery via Source Maps
If .map files are exposed, parse them to understand JS structure and hidden logic.
```bash
curl -s https://target.com/static/js/app.js.map | jq
```

## 9. Subdomain Takeover Detection
Combines subdomain scan + CNAME check + fingerprinting for takeover signs.
```bash
subfinder -d target.com -silent | dnsx -a -resp-only | xargs -I{} curl -s -L {} | grep -Ei "NoSuchBucket|404|unavailable|heroku"
```

## 10. Automated Nuclei Scanning on Live Hosts
Scans target subdomains for known CVEs using Nuclei templates.
```bash
subfinder -d target.com -silent | httpx -silent | nuclei -t cves/ -o target_cves.txt
```

## 11. Filter for Interesting Endpoints from Wayback
Finds potentially interesting legacy endpoints from archive.org.
```bash
waybackurls target.com | grep -E '\.php|\.asp|\.jsp|\.json|\.action' | tee endpoints.txt
```

## 12. S3 Bucket Enumeration
Tries to access S3-style URLs for public or restricted buckets.
```bash
echo target | while read domain; do echo "https://$domain.s3.amazonaws.com"; done | httpx -mc 200,403
```

## 13. Identify Technologies in Use
Provides quick tech stack fingerprinting for CMS, frameworks, server software, etc.
```bash
whatweb https://target.com
```

## 14. Filter Live Assets by Status Code
Returns only hosts responding with interesting HTTP codes.
```bash
cat hosts.txt | httpx -status-code -mc 200,403,500 -silent
```

## 15. Port Scan Fast
Quickly scans for open ports across a wide range.
```bash
naabu -host target.com -p 1-10000
```




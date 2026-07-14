# 30DaysofKaliLinux

## Day 1 & 2: Environment Setup and Initial Recon

VirtualBox lab: Kali (attacker) + Metasploitable2 (target) on host-only network `192.168.56.0/24`. `nmap -sn` found 4 live hosts; `nmap -sV -sC` against the target turned up the standard Metasploitable2 service set (vsftpd 2.3.4, Samba 3.0.20, UnrealIRCd, Tomcat manager, MySQL 5.0.51a, root shell backdoor on 1524, and more).

**Known exploits (Exploit-DB):**
- vsftpd 2.3.4 (21) — backdoored, CVE-2011-2523
- Samba 3.0.20 (139/445) — CVE-2007-2447, username map script injection
- UnrealIRCd (6667/6697) — backdoored, CVE-2010-2075
- Tomcat (8180) — default creds, WAR upload RCE

**Nmap flags used:** `-sT` connect scan · `-sV` version detect · `-sC` default scripts · `-O` OS detect · `-p-` all ports · `-oN` save output · `-v` verbose

---

## Day 3: DNS & Email Security Recon

Target: `zonetransfer.me` (DigiNinja's intentionally misconfigured practice domain).

- **A/NS:** resolves to `5.196.105.14`, served by `nszm1`/`nszm2.digi.ninja`
- **MX:** 7 records, all Google Workspace (`ASPMX.L.GOOGLE.COM` at priority 0)
- **TXT:** just a Google site-verification string, no SPF
- **DKIM** (`google._domainkey`): NXDOMAIN, not configured. SOA serial shows the zone hasn't changed since 2019.
- **Zone transfer** (`dig axfr @nsztm2.digi.ninja`) succeeds outright, dumping the full zone: CERT/HINFO records, an SRV record, internal-looking `127.0.0.1` entries, and subdomains never surfaced by manual queries.

**Takeaway:** no SPF or DKIM, so mail spoofed as this domain sails through basic checks, even though the MX records point at real, live Google mail servers. An open zone transfer on top of that hands over the entire internal record set for free.

---

## Day 4: Web Application Recon

Target: `zonetransfer.me`, continuing into the web layer.

- **whatweb:** Apache, hosted in Germany (OVH), HTTPS + HSTS enforced (`max-age=63072000`). Two joke headers (`do_not_hack_me`, an XSS-styled `X-Powered-By`) — not real vulnerabilities, just the maintainer's humor.
- **wafw00f:** no WAF detected.
- **nikto:** confirms Apache, update check returns a 403 (possible light rate-limiting), same joke headers, no CGI directories found.

**Takeaway:** HTTPS/HSTS is solid and there's no WAF in front of it. The odd headers are cosmetic. Nikto's 403 is worth re-testing with dedicated WAF-bypass techniques if this domain gets revisited.

---

## Day 5: SQL Injection (DVWA)

DVWA via Docker, Security Level: Low.

- **Baseline:** a single `'` breaks the query and returns a raw MariaDB syntax error, confirming unsanitized string concatenation and disclosing the DBMS.
- **Boolean-based:** `' OR '1'='1` returns every row in the `users` table instead of one.
- **UNION-based:** `' UNION SELECT user, password FROM users -- -` dumps usernames and MD5 password hashes from a completely unrelated table.

**Takeaway:** classic SQL injection (OWASP A03:2021). Fix: parameterized queries, strict input validation, generic error messages, and bcrypt/Argon2 instead of MD5 for password storage.

---

## Day 6: RSA Key Recovery from Image Metadata (picoCTF)

`image.jpg` + `flag.enc`. The JPEG's `exiftool` comment field held a hex-encoded PEM private key; decoded it with `xxd -r -p` into a valid 2048-bit RSA key, then `openssl pkeyutl -decrypt` against `flag.enc` recovered the flag: `picoCTF{rs4_k3y_1n_1mg_0a64c2f9}`

**Takeaway:** the JPEG COM marker is a blind spot for casual viewers. Hex-encoding stops a plain-text string search but not a metadata-first read. `openssl pkeyutl` is the current tool; `rsautl` is deprecated.

---

## Day 7: Steganography & Metadata Forensics (picoCTF)

Three challenges in one session:

1. **`digits.bin`** — raw bitstream, reconstructed via Python into `flag.jpg`. Flag rendered as text in the image: `picoCTF{h1dd3n_1n_th3_b1n4ry_3d2e65ba}`
2. **`confidential.pdf`** — `Author` metadata field is base64. Decoded straight to the flag: `picoCTF{puzzl3d_m3tadata_f0und!_c999e2a4}`
3. **`img.jpg`** — `Comment` field is base64, decodes to `steghide:<base64>`. Decoding again gives the steghide passphrase (`pAzzword`), used with `steghide extract -sf` to pull `flag.txt`: `picoCTF{h1dd3n_1n_1m4g3_1c55ccd0}`

**Takeaway:** layered encoding stops a naive single decode but not repeated decoding. Steghide payloads are invisible to casual viewing; metadata is the way in.

---

## Day 8: Base64 Image Recovery and OCR Extraction (picoCTF)

`logs.txt` is a base64-encoded PNG (`base64 -d` reveals the `PNG`/`IHDR` signature). Saved as `img.png`: an illustration with a hex string burned into the image itself. `tesseract` OCR pulled the hex string out; `base64 -d` rejected it (wrong character set), `basenc --base16 -d` decoded it to the flag: `picoCTF{forensics_analysis_is_amazing_5daa4a2f}`

**Takeaway:** base64 exists to carry binary through text-only channels. OCR catches text baked into pixels that metadata tools miss. A pure hex character set (`0-9`, uppercase `A-F`, no `+`/`/`/`=`) is the signal to reach for `basenc --base16 -d` instead of `base64 -d`.

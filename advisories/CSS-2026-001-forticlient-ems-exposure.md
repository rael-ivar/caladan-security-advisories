# CSS-2026-001: CVE-2026-35616 — FortiClient EMS Improper Access Control

**Published:** 2026-04-20  
**Author:** Rael Ivar, Caladan Security Studio  
**CVE:** CVE-2026-35616  
**CVSS Score:** 9.8 Critical (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)  
**CWE:** CWE-284 — Improper Access Control  
**Vendor:** Fortinet  
**Product:** FortiClient Enterprise Management Server (EMS)  
**Affected Versions:** 7.4.5, 7.4.6  
**Fixed In:** 7.4.7  
**CISA KEV Listed:** Yes (remediation deadline: April 9, 2026)  

---

## Summary

An improper access control vulnerability in Fortinet FortiClient EMS allows an unauthenticated remote attacker to send crafted requests that bypass API authentication and authorization, resulting in unauthorized code or command execution. The vulnerability is actively exploited in the wild and was added to the CISA Known Exploited Vulnerabilities catalog with a mandatory remediation deadline of April 9, 2026.

This advisory documents our independent exposure analysis — a passive reconnaissance and behavioral probe study of internet-facing FortiClient EMS deployments to assess the real-world patching status of affected organizations.

---

## Exposure Analysis

Using the Censys platform (free tier), we identified **1,590 internet-exposed FortiClient EMS instances** globally. Of those, **72 hosts** from the first 100 results were confirmed on the 7.4.x branch (the vulnerable version family) via passive CSP header fingerprinting.

### Detection Methodology

**Version-era fingerprinting (passive):**
- 7.4.x signal: `Content-Security-Policy: img-src 'self' https: data: blob:`
- 7.2.x signal: `Content-Security-Policy: img-src 'self' data:` (no `https:` token)
- Confirmed via webpack hash-based CSS filenames (`signin.[contenthash].css`) vs explicit version params (`fcems-signin.min.css?v=7.2.x.NNNN`)

**Vulnerability confirmation (behavioral probe — Bishop Fox methodology):**

Send two POST requests to `/api/v1/auth/signin` with body `{"name":"admin","password":"wrongpassword"}`:
1. Baseline request (no extra headers)
2. Spoofed request with header `X-SSL-CLIENT-VERIFY: SUCCESS`

Interpretation:
- **Vulnerable (7.4.5/7.4.6):** Baseline returns 401, spoofed returns 500 — authentication bypassed via header injection
- **Patched (7.4.7):** Both return 401 — header stripped or neutralized

---

## Findings

### Named Organizations Identified (Passive TLS/Certificate Analysis)

We extracted organizational identities from TLS certificates on 18 high-signal hosts:

| Organization | Sector | Country | Version Era | Probe Result |
|---|---|---|---|---|
| JVHL (120+ hospital network, 5.4M patients) | Healthcare | US | 7.4.x | PATCHED (401/401) |
| Ayuntamiento de Vera | Municipal Government | Spain | 7.4.x | PATCHED (401/401) |
| Assas University (Paris II) | Higher Education | France | 7.4.x | PATCHED (401/401) |
| Bühler Industries | Manufacturing | Canada | 7.4.x | PATCHED (400/400) |
| Engage2Excel | HR Services | US | 7.4.x | PATCHED (401/401) |
| Krüger GmbH | Consumer Goods | Germany | 7.4.x | PATCHED (401/401) |
| University of Bern | Research University | Switzerland | 7.4.x | PATCHED (401/401) |
| Finecom | Technology Services | Thailand | 7.4.x | PATCHED (401/401) |

### Full 100-Host Behavioral Probe Results (April 20, 2026)

| Result | Count |
|---|---|
| Vulnerable (401→500 pattern) | **0** |
| Patched/Safe (symmetric response) | **78** |
| Unreachable | **22** |
| **Total** | **100** |

Two hosts confirmed vulnerable on April 18 (AXS Bolivia, Telecom Egypt) had patched by April 20 — remediating within 48 hours of confirmation. This is the fastest observed remediation in our dataset.

---

## Interpretation

**The CISA KEV deadline drove measurable patching.** 100% of reachable hosts in the highest-visibility portion of the Censys dataset are patched as of April 20, 2026. Healthcare, government, university, and enterprise organizations — including those outside CISA jurisdiction — have remediated.

**Caveat on coverage:** This analysis covers the top 100 results from the Censys free tier. Censys reports approximately 3,835 version-matched vulnerable hosts in their full dataset. The tail population (smaller organizations, weaker fingerprint matches, regions with less CERT pressure) may contain unpatched instances not visible in our sample.

**The proxy stratum matters.** Approximately 132 of 1,590 exposed EMS instances are directly API-accessible without an Apache reverse proxy. These represent the highest-exploitability subset — no header-stripping layer between attacker and the Django backend. All named organizations in our dataset fell into this directly-accessible category.

---

## Recommendations

Organizations running FortiClient EMS 7.4.5 or 7.4.6 should update to 7.4.7 immediately. Hotfixes are also available for 7.4.5/7.4.6.

Security teams can use the detection methodology above to validate patching status without lab access. The 401/401 symmetric response on `/api/v1/auth/signin` is a reliable patched indicator.

---

## References

- Fortinet PSIRT Advisory: FG-IR-26-099
- CISA KEV: [CVE-2026-35616](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- NVD: CVE-2026-35616
- Bishop Fox behavioral detection methodology

---

## About

This advisory was produced by [Caladan Security Studio](https://caladansecuritystudio.com), an independent security research studio. Contact: rael.ivar@caladan.cc

*Caladan Security Studio follows responsible disclosure principles. All active probing was conducted using standard HTTP requests consistent with what any authenticated or unauthenticated user would send. No systems were accessed without authorization, exploited, or disrupted.*

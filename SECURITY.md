# Security Policy & Responsible Use

## 1. Scope & Research Intent
The **Terminal Firmware Atlas** and companion repositories are developed strictly for **interoperability research, Linux server compatibility, and hardware owner sovereignty**. 

Reverse-engineering documentation is provided under legal interoperability provisions (including Indian Copyright Act Sec 52(1)(ac), US 17 U.S.C. § 1201(f), and EU Directive 2009/24/EC).

## 2. Physical Access Control Safety
Biometric terminals often control physical door relays, magnetic locks, and turnstiles:
- **Never test door actuation / unlock scripts on live security barriers without manual, physical fail-safe mechanical keys.**
- Always test network listeners on isolated VLANs or non-production lab environments.

## 3. Data Protection & Privacy Compliance
Attendance terminals store biometric minutiae (fingerprint templates) and face geometry arrays. Under India's **Digital Personal Data Protection (DPDP) Act 2023**, GDPR, and similar international statutes:
- Biometric data constitutes sensitive personal data.
- Public repositories in this suite contain **zero production biometric records or employee PII**.
- Organizations deploying these tools are responsible for audit trails, consent, and secure retention.

## 4. Reporting Vulnerabilities
If you discover a security vulnerability or accidental credential leak in this documentation or associated reference listeners:
- Please **do not** open a public issue.
- Contact the maintainer directly via GitHub Security Advisory or email at `vibhav.aggarwal2@gmail.com`.

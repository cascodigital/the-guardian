# The Guardian — Email Security Audit Tool

Public email security auditor that analyzes **SPF**, **DMARC**, and **MX** records to detect spoofing vulnerabilities and generate a risk score (0–100) for any domain.

**Live:** [cascodigital.com.br/guardian](https://cascodigital.com.br/guardian)

## How it works

1. User enters a domain
2. Cloudflare Worker (`functions/api/audit.js`) queries DNS records via Cloudflare DNS-over-HTTPS
3. SPF, DMARC, and MX are parsed and scored
4. Frontend displays the risk score with remediation recommendations

## Stack

- Vanilla HTML/CSS/JS (no dependencies)
- Cloudflare Workers (serverless backend)
- Cloudflare DNS-over-HTTPS API

## Deploy

Hosted on Cloudflare Pages with Workers integration. Part of the [Casco Digital](https://cascodigital.com.br) platform.

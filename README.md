# the-guardian

A public email-security audit page that scores a domain's SPF, DMARC and MX posture in the browser, aimed at small-business owners checking their spoofing exposure.

![License](https://img.shields.io/badge/license-MIT-blue) ![JavaScript](https://img.shields.io/badge/JavaScript-ES2020-yellow) ![Status](https://img.shields.io/badge/status-active-green)

**Live:** [cascodigital.com.br/diagnostico](https://cascodigital.com.br/diagnostico)

## Overview

Most small companies have no idea whether someone can send email as their domain. Existing checkers (MXToolbox
and similar) return raw records and protocol jargon that a non-technical owner cannot act on. The Guardian
queries the domain's DNS straight from the visitor's browser, turns SPF, DMARC and MX findings into a single
0-100 protection score, and explains the business impact and the fix in plain language. The page is in
Brazilian Portuguese because it targets Brazilian visitors.

## Features

- Looks up SPF (`TXT`), DMARC (`_dmarc` `TXT`) and `MX` records through Cloudflare DNS-over-HTTPS, from the browser.
- Classifies SPF by its `all` qualifier (`+all`, `?all`, `~all`, `-all`, missing or incomplete).
- Classifies DMARC by policy (`p=reject`, `p=quarantine`, `p=none`, missing or incomplete).
- Computes a 0-100 protection score (higher is safer) and maps it to four risk levels.
- Shows the raw record, a per-record recommendation and a business-impact summary.
- Reassembles TXT records that DNS returns split into several quoted strings.
- Normalizes input such as `https://example.com/path` or `user@example.com` to `example.com`.
- Runs as one static HTML file: no backend, API keys, database or build step.

## How it works

```text
Browser (guardian.html, vanilla JS)
    |  3 parallel fetch() calls, Accept: application/dns-json
    |    TXT  example.com
    |    TXT  _dmarc.example.com
    |    MX   example.com
    v
cloudflare-dns.com/dns-query (public DoH JSON API, CORS-enabled)
    |
    v
parseTxtRecords -> analyzeSpf / analyzeDmarc / analyzeMx -> calcScore -> renderResults
```

`guardian.html` is a single self-contained file (inline CSS and vanilla JS). The domain never reaches a
server operated by this project: the browser calls Cloudflare's public resolver directly, which serves the
JSON DoH API with permissive CORS headers. Any static host works.

The score starts at 100 and loses points per finding:

| Check | Result | Points subtracted |
|---|---|---|
| SPF | `+all` / missing / `?all` / no `all` qualifier / `~all` / `-all` | 50 / 40 / 30 / 20 / 15 / 0 |
| DMARC | missing / no `p=` tag or unknown policy / `p=none` / `p=quarantine` / `p=reject` | 35 / 20 / 15 / 5 / 0 |
| MX | missing / present | 15 / 0 |

The result is clamped to 0-100 and mapped to levels: 90+ low risk, 70-89 moderate, 50-69 high, below 50
critical. If any of the three lookups fails, the page shows a DNS error instead of a partial score.

## Requirements

- Any static web host (Cloudflare Pages, GitHub Pages, nginx, a local file server).
- A modern browser with `fetch` and ES2020 support; outbound HTTPS to `cloudflare-dns.com`.
- No environment variables or API keys.

## Quick start

1. Clone the repository:
   ```bash
   git clone https://github.com/cascodigital/the-guardian.git
   cd the-guardian
   ```
2. Serve the folder with any static server:
   ```bash
   python3 -m http.server 8080
   ```
3. Open `http://localhost:8080/guardian.html`, enter a domain such as `example.com` and click **Auditar Agora**.

To publish on Cloudflare Pages: **Workers & Pages** > **Create** > **Pages** > **Connect to Git**, framework
preset **None**, build command empty, output directory `/`.

## Configuration

No runtime configuration. The only tunable values are constants in the inline script:

| Variable / key | Required | Default | Description |
|---|---|---|---|
| `DOH_ENDPOINT` | No | `https://cloudflare-dns.com/dns-query` | DoH resolver; must serve the JSON API (`application/dns-json`) with CORS |
| `calcScore()` penalty tables | No | see table above | Points subtracted per SPF/DMARC/MX status |
| `scoreLevel()` thresholds | No | `90` / `70` / `50` | Score boundaries between risk levels |

## Usage

The audit can be reproduced from a shell with the same queries the page sends:

```bash
curl -s -H 'Accept: application/dns-json' 'https://cloudflare-dns.com/dns-query?name=example.com&type=TXT'
curl -s -H 'Accept: application/dns-json' 'https://cloudflare-dns.com/dns-query?name=_dmarc.example.com&type=TXT'
curl -s -H 'Accept: application/dns-json' 'https://cloudflare-dns.com/dns-query?name=example.com&type=MX'
```

## Limitations

- Checks are string-based: it does not validate SPF syntax, count DNS lookups (10-lookup limit) or follow
  `include:` chains.
- Only the first SPF and DMARC records are evaluated; multiple SPF records (itself an error) are not flagged.
- DKIM is not checked (selectors are not discoverable from DNS alone).
- DMARC `sp=`, `pct=` and reporting tags (`rua`, `ruf`) are not evaluated.
- Relies on a third-party resolver; networks that block `cloudflare-dns.com` get a DNS error.
- User-facing text is Portuguese only.

## License

MIT — see [LICENSE](LICENSE).

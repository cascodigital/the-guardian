# the-guardian

A public email-security audit page that scores a domain's SPF, DMARC and MX posture, aimed at small-business owners checking their spoofing exposure.

![License](https://img.shields.io/badge/license-MIT-blue) ![JavaScript](https://img.shields.io/badge/JavaScript-ES2020-yellow) ![Status](https://img.shields.io/badge/status-active-green)

**Live:** [cascodigital.com.br/guardian](https://cascodigital.com.br/guardian)

## Overview

Most small companies have no idea whether someone can send email as their domain. Existing checkers (MXToolbox
and similar) return raw records and protocol jargon that a non-technical owner cannot act on. The Guardian
queries the domain's DNS, turns SPF, DMARC and MX findings into a single 0-100 risk score, and explains the
business impact and the fix in plain language. The page is in Brazilian Portuguese because it targets
Brazilian visitors.

## Features

- Looks up SPF (`TXT`), DMARC (`_dmarc` `TXT`) and `MX` records through Cloudflare DNS-over-HTTPS.
- Classifies SPF by its `all` qualifier (`+all`, `?all`, `~all`, `-all`, missing or incomplete).
- Classifies DMARC by policy (`p=reject`, `p=quarantine`, `p=none`, missing or incomplete).
- Computes a weighted risk score and maps it to four levels (low, moderate, high, critical).
- Shows the raw record, a per-record recommendation and a business-impact summary.
- Normalizes input such as `https://example.com/path` to `example.com`.
- Runs without API keys, databases or build tooling.

## How it works

```text
Browser (guardian.html)
    |  GET /api/audit?domain=example.com
    v
Cloudflare Pages Function (functions/api/audit.js)
    |  3 parallel DoH queries: TXT, _dmarc TXT, MX
    v
cloudflare-dns.com/dns-query  -->  JSON { score, spf, dmarc, mx }
```

`guardian.html` is a single self-contained file (inline CSS and vanilla JS). On submit it calls the Pages
Function, which runs the three DNS lookups with `Promise.all` and scores the results. Higher score means more
risk:

| Check | Result | Points |
|---|---|---|
| SPF | missing / `+all` / `?all` / `~all` / no qualifier / `-all` | 40 / 35 / 20 / 15 / 15 / 5 |
| DMARC | missing / `p=none` / no policy / `p=quarantine` / `p=reject` | 40 / 20 / 15 / 5 / 0 |
| MX | missing / present | 10 / 0 |

The total is capped at 100. The page maps it to levels: 0-20 low, 21-50 moderate, 51-75 high, 76+ critical.
The API returns permissive CORS headers so the endpoint can be called from other origins.

## Requirements

- A Cloudflare account with Cloudflare Pages (Functions enabled by default).
- No environment variables or API keys; only the public Cloudflare DoH resolver is used.
- Optional for local testing: Node.js and `wrangler`.

## Quick start

1. Fork or clone the repository.
2. In Cloudflare, go to **Workers & Pages** > **Create** > **Pages** > **Connect to Git** and select the repo.
3. Build settings: framework preset **None**, build command empty, output directory `/`.
4. After deployment, test the API:
   ```bash
   curl "https://<your-project>.pages.dev/api/audit?domain=example.com"
   ```

To run locally instead:

```bash
npx wrangler pages dev .
# then open http://localhost:8788/guardian
```

## Configuration

No runtime configuration. The only tunable values are constants in the source:

| Variable / key | Required | Default | Description |
|---|---|---|---|
| `DOH` (`functions/api/audit.js`) | No | `https://cloudflare-dns.com/dns-query` | DNS-over-HTTPS endpoint (JSON API) |
| `domain` query parameter | Yes | — | Domain to audit; scheme and path are stripped |

## Usage

API response example:

```json
{
  "ok": true,
  "domain": "example.com",
  "score": 25,
  "spf":   { "exists": true, "value": "v=spf1 -all", "score": 5, "status": "strict", "label": "..." },
  "dmarc": { "exists": true, "value": "v=DMARC1; p=none", "score": 20, "status": "monitoring", "label": "..." },
  "mx":    { "exists": true, "records": ["0 ."], "score": 0 }
}
```

Errors return `{ "ok": false, "error": "..." }` with HTTP 400 (missing `domain`) or 500 (lookup failure).
`label` and `error` strings are in Portuguese because the page displays them directly.

## Limitations

- Checks are string-based: it does not validate SPF syntax, count DNS lookups (10-lookup limit) or follow
  `include:` chains.
- Only the first SPF record is evaluated; multiple SPF records (itself an error) are not flagged.
- DKIM is not checked (selectors are not discoverable from DNS alone).
- DMARC `sp=`, `pct=` and reporting tags (`rua`, `ruf`) are not evaluated.
- No rate limiting on the public endpoint.
- User-facing text is Portuguese only.

## License

MIT — see [LICENSE](LICENSE).

# Nehboro

Community-powered browser extension for threat protection. Homepage at [nehboro.github.io](https://nehboro.github.io).

## What is this repo

This is the **public GitHub Pages site** for the Nehboro browser extension. It serves four purposes:

1. **Documentation** - the landing page at the root describes the extension
2. **Live threat feeds** - the `/feeds/` folder holds CSV files that the extension fetches
3. **URL Scanner** - the `/scan/` page runs Nehboro detections client-side against any URL (but less detection logics than the extension itself)
4. **Community Reports browser** - the `/reports/` page shows all threats reported by users

The browser extension source code lives in [github.com/Nehboro/nehboro](https://github.com/Nehboro/nehboro)

extension is available on the chrome webstore: https://chromewebstore.google.com/detail/nehboro/ljgklnaofelbcnegjniagpmjknkmaiom

## Contents

```
/
├── index.html              # Landing page
├── nehboro_logo_256.png    # Logo
├── favicon.png             # Favicon (transparent Totoro icon)
├── feeds/                  # Threat feeds consumed by the extension
│   ├── domains.csv         # Scam & phishing domains
│   ├── urls.csv            # Malicious URL patterns
│   ├── ips.csv             # IPs, CIDR ranges, IP wildcards
│   └── ports.csv           # Suspicious ports
├── scan/
│   └── index.html          # Client-side URL scanner (zero setup)
├── reports/
│   ├── index.html          # Browse all community reports
│   ├── view/index.html     # Individual report detail page
│   ├── reports.json        # Aggregated report index (auto-generated)
│   └── *.jsonl             # Raw NDJSON dumps from ntfy (auto-generated)
└── .github/workflows/
    └── fetch-reports.yml   # Pulls reports from ntfy.sh/nehboro-reports
```

## Feeds

Plain CSV files in `/feeds/`, supporting:

| File | Accepts | Examples |
|---|---|---|
| `domains.csv` | Domains, wildcards | `evil.com`, `*.evil.com`, `pay*.net` |
| `urls.csv` | Full URLs, wildcards | `https://evil.com/x.ps1`, `*://evil.*/gate.php` |
| `ips.csv` | IPs, CIDR, wildcards | `1.2.3.4`, `10.0.0.0/24`, `192.168.*.*` |
| `ports.csv` | Ports and ranges | `4444`, `8080-8085` |

Lines starting with `#` are comments. Headers are auto-detected and skipped.

These feeds focus on **confirmed scam and phishing infrastructure**. Chrome caps the number of static blocking rules per extension, so the feeds are intentionally curated rather than exhaustive. The extension's **dynamic heuristic engine** (97 detections) handles everything else.

## Reports system

Reports flow into this repo automatically:

1. **Extension** sends reports to `ntfy.sh/nehboro-reports` when the user reports a page (or auto-reports at score ≥110)
2. **URL Scanner** sends reports to the same topic when the user clicks Report (or auto-reports at score ≥110)
3. **GitHub Action** (`fetch-reports.yml`) polls `ntfy.sh/nehboro-reports/json?poll=1&since=...` for new messages, archives them as `reports/{timestamp}.jsonl` and rebuilds `reports/reports.json` (the aggregated index)
4. **Reports page** (`/reports/`) fetches `reports.json` and displays browsable, filterable, searchable list with permalinks

## URL Scanner

`/scan/` runs some of the Nehboro extension detections entirely client-side:

- Detection modules loaded at runtime from `cdn.jsdelivr.net/gh/Nehboro/nehboro@main/`
- Target HTML fetched via `api.allorigins.win` (free public CORS proxy, no signup)
- Most of the detections run against the fetched content
- Results saved to `localStorage` for 24h dedup
- Auto-reports threats at score ≥110 to the same ntfy topic
- Manual Report button for any score
- Shareable URL: `/scan/?u=https%3A%2F%2Fexample.com`

## Contributing

- **Report a threat**: use the extension's Report button or the web scanner's Report button, or open an issue.
- **Submit a detection**: open a PR on the [extension repo](https://github.com/Nehboro/nehboro) with a new `detections/*.js` module.
- **Add to feeds**: open a PR modifying the CSV files in `/feeds/`.

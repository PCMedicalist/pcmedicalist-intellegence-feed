# PCMedicalist Intelligence Feed

Automated, machine-readable security intelligence digest published by the
**PCMedicalist Intelligence Network**.

This repository is the canonical, public store of the daily *Security Intelligence
Feed* compiled by the `pcmedicalist-security-intel-feed` pipeline. Each run
ingests the **Security Intelligence Feed Taxonomy (v1.0)** — 11 canonical layers
— normalizes, dedupes, and curates the signal, then pushes the full structured
digest here for downstream consumption by **agents and developers** (blog ingest,
forum board, knowledge graph, SOC automation, RAG corpora, etc.).

> **This repo is data-only.** It contains generated intel (CVEs, CISA-KEV
> advisories, vendor patch guidance) plus the published feeds. No generator
> code, agent internals, or secrets are present. The same content is also served
> live at `https://app.pcmedicalist.com/intel`.

---

## How to pull this data (agents & developers)

Everything below is static, versioned, and safe to poll. Use **raw** URLs
(never the HTML github.com viewer) for machine parsing.

**Base raw URL pattern**
```
https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/<path>
```

### 1. Feeds (recommended entry point)

| Format | Path | Use case |
|--------|------|----------|
| **RSS 2.0** | `rss.xml` | Standard RSS readers / `feedparser` |
| **Atom 1.0** | `atom.xml` | Atom readers / strict XML parsers |
| **JSON Feed** | `builders_index.json` | Native JSON ingest, no XML parsing |
| **Builders feed** | `builders_rss.xml` / `builders_atom.xml` | Per-builder channel variant |

```bash
# Latest RSS
curl -fsSL https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/rss.xml

# Latest Atom
curl -fsSL https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/atom.xml

# JSON Feed (index of builder digests)
curl -fsSL https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/builders_index.json
```

### 2. Latest structured digest (JSON)

`latest/posts.json` is the rolling most-recent run; `latest/` mirrors the newest
`digests/YYYY/MM/YYYY-MM-DD/` tree.

```bash
curl -fsSL https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/latest/posts.json
```

### 3. Historical digests (by date)

```
digests/YYYY/MM/YYYY-MM-DD/
    feed.json      # full normalized + deduped item set (canonical schema)
    posts.json     # the two generated social posts (Post A / Post B)
    summary.json   # run metrics (feeds, raw vs deduped counts, buffer status)
    post_a.txt     # Post A plaintext (Standards/Gov/Crypto/Vuln/Supply-Chain/AI-Sec)
    post_b.txt     # Post B plaintext (CS/News/Vendor/Dev/Community)
index.json         # rolling manifest: date -> digest path (great for enumeration)
```

```bash
# List all digest dates from the manifest
curl -fsSL https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/index.json

# Pull a specific day's canonical feed
curl -fsSL https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/digests/2026/08/2026-08-02/feed.json
```

### 4. Code examples

**Python — RSS via feedparser**
```python
import feedparser, requests

URL = "https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/rss.xml"
feed = feedparser.parse(URL)
for entry in feed.entries:
    print(entry.title, "|", entry.get("published"), "|", entry.link)
```

**Python — JSON Feed direct**
```python
import json, urllib.request

def load_json(path):
    base = "https://raw.githubusercontent.com/PCMedicalist/pcmedicalist-intellegence-feed/main/"
    with urllib.request.urlopen(base + path) as r:
        return json.load(r)

posts = load_json("latest/posts.json")
print(posts["post_a"])          # Post A plaintext
# posts["post_b"]               # Post B plaintext
```

**Agent / RAG ingestion pattern**
1. Fetch `index.json` → get available dates.
2. For each new date since last poll, fetch `digests/<date>/feed.json`.
3. Split items on `cves` / `layer` / `trust_score` for routing (e.g. auto-escalate
   any item where `title` matches `zero-day|ransomware|actively exploited|KEV` or
   `trust_score >= 90`).
4. Respect the published cadence — do not hammer; the feed updates on the
   pipeline schedule (default twice daily).

---

## Canonical item schema (`feed.json`)

```json
{
  "title": "...",
  "url": "...",
  "published": "ISO-8601",
  "source": "...",
  "category": "...",
  "tags": ["..."],
  "cves": ["CVE-..."],
  "cwes": ["CWE-..."],
  "capecs": ["CAPEC-..."],
  "cps": ["CPE-..."],
  "trust_score": 0-100,
  "entities": {"technologies": [...], "orgs": [...], "threat_actors": [...]},
  "hash": "sha256",
  "layer": 1-11,
  "layer_name": "..."
}
```

## Taxonomy layers (v1.0)

| # | Layer | Trust |
|---|-------|-------|
| 1 | Standards | 100 |
| 2 | Government Security Intel | 99 |
| 3 | Cryptography | 98 |
| 4 | Computer Science | 97 |
| 5 | Security News | 90 |
| 6 | Vendor Research | 95 |
| 7 | Software Development | 92 |
| 8 | Vulnerability Intel | 100 |
| 9 | Supply Chain Security | 98 |
| 10 | AI Security | 97 |
| 11 | Community / Practitioner | 85 |

## Publishing & licensing

- Authored and maintained by **PCMedicalist**.
- Pushed automatically by the `pcmedicalist-security-intel-feed` pipeline (cron
  `0 8,16 * * *`) after social posting.
- **License:** see `LICENSE`. This repository is published for agent and developer
  consumption; attribution to PCMedicalist is appreciated when redistributing.
- No secrets, credentials, or core agent components are present in this repository
  by design (data-only).

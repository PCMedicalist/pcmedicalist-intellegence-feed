# PCMedicalist Intelligence Feed

Automated security intelligence digest repository for the **PCMedicalist Intelligence Network**.

This repo is the canonical store of the daily Security Intelligence Feed compiled by the
`pcmedicalist-security-intel-feed` skill. Each day the pipeline ingests the
**Security Intelligence Feed Taxonomy (v1.0)** — 11 canonical layers — normalizes, dedupes,
and curates the signal into two social posts (Buffer), then pushes the full structured
digest here for downstream use (blog, forum board, knowledge graph).

## Structure

```
digests/YYYY/MM/YYYY-MM-DD/
    feed.json      # full normalized + deduped item set (canonical schema)
    posts.json     # the two generated social posts (Post A / Post B)
    summary.json   # run metrics (feeds, raw vs deduped counts, buffer status)
    post_a.txt     # Post A plaintext (Standards / Gov / Crypto / Vuln / Supply-Chain / AI-Sec)
    post_b.txt     # Post B plaintext (CS / News / Vendor / Dev / Community)
latest/            # rolling copy of the most recent digest
index.json         # rolling manifest of every digest (date -> path) for blog/forum ingestion
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

## Canonical item schema

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

## Automation

Pushed automatically by `run_daily.py` (cron: `0 8,16 * * *`) after social posting.
Authored by **PCMedicalist**.

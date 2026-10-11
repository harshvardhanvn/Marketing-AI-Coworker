# Marketing AI Coworker

An agentic marketing workspace: four specialist AI agents coordinated by a
Marketing Manager, all reasoning through **Sarvam AI** as the LLM provider,
with a Flask backend and a responsive SaaS dashboard.

Built for a 1–2 day hackathon. It runs end-to-end with **no third-party
marketing integrations** — a realistic sample business and CSV dataset are
included so every workflow is demonstrable offline.

---

## Quick start

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Configure (optional - app runs in DEMO MODE without a key)
cp .env.example .env
#   edit .env and set SARVAM_API_KEY=...   (get one at dashboard.sarvam.ai)

# 3. Run
python app.py
#   or:  ./run.sh
```

Then open <http://localhost:5000>.

Run the tests:

```bash
python -m pytest -q
```

---

## How it works

```
Browser (dashboard)
      |  JSON + multipart
      v
Flask  app.py  --->  db.py (SQLite persistence)
      |
      +--> agents/
      |      seo_analyst.py        fetch + parse a real page, detect issues
      |      campaign_analyst.py   CSV -> Python metrics -> Sarvam interpretation
      |      keyword_research.py   intent clusters (no fabricated volume)
      |      content_creator.py    ad/landing/blog/social/email copy
      |      marketing_manager.py  orchestrator -> prioritised growth plan
      |
      +--> services/
      |      website.py      fetch + on-page SEO extraction (+ SSRF guard)
      |      csv_metrics.py  CTR / CPC / CVR / CPA / ROAS + snapshot diff
      |
      +--> sarvam_client.py  official `sarvamai` SDK wrapper + demo mode
```

**Division of labour:** every *number* is computed in Python. Sarvam is only
ever asked to *interpret* those numbers or write copy. This keeps the metrics
trustworthy and auditable regardless of the LLM.

---

## Feature status (honest)

### Genuinely working (verified by automated tests + a live HTTP smoke run)

| Feature | Where | Notes |
|---|---|---|
| Dashboard UI, nav, forms, charts, exports | `templates/`, `static/` | Rendered in a real headless browser; 10/10 nav items wired; no JS errors |
| Campaign CSV → CTR, CPC, CVR, CPA, ROAS | `services/csv_metrics.py` | Computed in Python; independently recomputed in tests |
| CSV validation + tolerant column aliases | `services/csv_metrics.py` | Missing/negative/non-numeric inputs rejected with clear errors |
| Real website fetch + SEO extraction | `services/website.py` | Live-fetched a real public page during QA (title, H1s, alt text, score) |
| Keyword clusters (intent-based) | `agents/keyword_research.py` | 4 intent buckets; deliberately no volume/CPC numbers |
| Content generation | `agents/content_creator.py` | 5 asset types; output is editable and saved |
| Manager orchestration | `agents/marketing_manager.py` | Ran 3 agents, aggregated 6 recommendations, produced a growth plan |
| Recommendation tracking | `db.py` + `/api/recommendations` | Status can be moved open → in_progress → done/dismissed |
| Snapshot comparison | `services/csv_metrics.compare_snapshots` | Diffs two snapshots with correct "higher/lower is better" logic |
| Persistence | `db.py` (SQLite) | Recommendations, snapshots, content, analyses, experiments |
| Exports | `/api/export/*.csv` | Recommendations and per-snapshot CSV downloads |
| Input validation + JSON error handling | `app.py` | 400/404/413/422/500 all return structured JSON |
| API-key handling | `config.py`, `sarvam_client.py` | Read from env only; never logged, never hard-coded |

### Working, but running on DEMO data right now

- **All LLM output** (SEO advice, campaign interpretation, keyword copy,
  content drafts, growth plan) is produced in **DEMO MODE** because no
  `SARVAM_API_KEY` was available in the build sandbox. Demo output is always
  prefixed `[DEMO MODE ...]` and every result carries a `demo: true` flag, so
  it can never be mistaken for a live model response. Set the key and the exact
  same code path calls Sarvam live.
- **The sample business and CSV** ("GreenLeaf Organics", 6 campaigns) are
  illustrative data, not a real client.

### Requires external credentials / integrations to go further

- **Live Sarvam responses** — needs `SARVAM_API_KEY`.
- **Keyword search-volume / CPC / difficulty** — intentionally *not*
  implemented; there is no keyword-data source wired up, and inventing those
  numbers would be fabrication. Add a keyword API (e.g. a paid SEO data
  provider) to populate them.
- **Real ad-platform data** — the app ingests CSV exports; it does not call
  the Google Ads / Meta APIs. Connect those to pull data automatically.
- **Public web fetch** — works against publicly reachable hosts. In a
  restricted network (like the build sandbox, which uses an allowlisted egress
  proxy) only allowlisted hosts are reachable.

---

## API reference

| Method | Path | Purpose |
|---|---|---|
| GET | `/` | Dashboard |
| GET | `/api/health` | App + Sarvam status, record counts |
| GET | `/api/business` | Seeded business context |
| POST | `/api/seo/analyze` | `{ "url": "https://..." }` |
| POST | `/api/campaign/analyze` | multipart `file`, or `{ "use_sample": true }` |
| POST | `/api/keywords/research` | `{ "topic": "..." }` |
| POST | `/api/content/generate` | `{ "asset_type", "topic", "tone" }` |
| POST | `/api/manager/run` | `{ "url"?, "topic"?, "use_sample_csv"? }` |
| GET | `/api/recommendations` | List recommendations |
| PATCH | `/api/recommendations/<id>` | `{ "status": "done" }` |
| GET | `/api/snapshots` | List snapshots |
| POST | `/api/snapshots/compare` | `{ "older_id", "newer_id" }` |
| POST/GET | `/api/experiments` | Create / list experiments |
| GET | `/api/history` | Analysis history |
| GET | `/api/export/recommendations.csv` | Export recommendations |
| GET | `/api/export/snapshots/<id>.csv` | Export a snapshot |
| GET | `/api/sample/campaign.csv` | Download sample dataset |

---

## Security notes

- The API key is read **only** from the `SARVAM_API_KEY` environment variable
  (or `.env`, which is git-ignored). It is never written to the database, the
  logs, or any API response.
- Uploads are size-capped and restricted to `.csv`.
- The website fetcher has an SSRF guard: literal private/loopback/link-local
  IPs and `.local`/`.internal` hostnames are refused. When local DNS is
  unavailable (proxy environments) it falls back to hostname checks and
  defers to the network layer's own egress policy.
- The bundled `SECRET_KEY` default is a dev placeholder — set
  `MAC_SECRET_KEY` for anything public.

## Limitations

- SQLite is fine for a hackathon demo; use Postgres for real concurrency.
- The Flask dev server is not for production; a `Procfile` with gunicorn is
  included for deployment.
- No auth layer — do not expose this publicly without adding one.

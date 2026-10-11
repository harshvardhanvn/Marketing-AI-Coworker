# Five-Minute Hackathon Demo Script

**Before you start (do this once, off-stage):**

```bash
pip install -r requirements.txt
python app.py          # leave running; open http://localhost:5000
```

If you have a Sarvam key, put it in `.env` as `SARVAM_API_KEY=...` first —
then every step below returns *live* model output instead of demo output. If
you don't, everything still runs; output is tagged **DEMO**.

Two browser tabs help: the app, and (optionally) `sample_data/sample_campaign_v2.csv`.

---

### 0:00 – 0:30 — Frame it

Open the dashboard. Point at the top-right badge: **DEMO MODE** (or **LIVE
Sarvam** if you set a key).

> "This is a Marketing AI Coworker — four specialist agents on top of Sarvam AI,
> coordinated by a Marketing Manager agent. The dashboard you're seeing is the
> real running app, not a mockup. We're on the sample brand, GreenLeaf Organics."

---

### 0:30 – 1:30 — SEO Analyst (real website)

Click **SEO Analyst**. Click **Use sample** (fills `https://example.com`), then
**Analyse page**.

> "This actually fetches a live public web page and parses it — no mock data.
> It pulled the real title, counted the H1s, found images missing alt text, and
> scored the page 48 out of 100. Sarvam then turns those machine-detected
> issues into prioritised fixes."

Point at the findings table (severity / area / issue / fix) and the commentary box.

---

### 1:30 – 2:30 — Campaign Analyst (the maths)

Click **Campaign Analyst** → **Analyse sample dataset**.

> "Six campaigns. CTR, CPC, conversion rate, CPA and ROAS are computed in
> Python — not by the LLM — so the numbers are exact and auditable. Sarvam is
> only asked to interpret them."

Point at the metric cards (CTR 3.51%, ROAS 5.68x), the ROAS bar chart, and the
per-campaign table. Then:

> "It flagged the best-ROAS campaign and the worst-CPA campaign, and saved this
> run as a snapshot."

---

### 2:30 – 3:00 — Keyword Research

Click **Keyword Research**, type `cold pressed coconut oil`, submit.

> "It builds intent-based clusters — informational, commercial, transactional,
> navigational — with a content opportunity for each. Notice what it *doesn't*
> do: it never invents search volume or CPC, because we have no keyword-data
> source. We'd rather show nothing than fabricate a number."

---

### 3:00 – 3:30 — Content Creator

Click **Content Creator**, choose **Ad copy**, topic `organic millet snacks`,
submit.

> "Editable ad copy in one shot — headline variants, primary text, CTA. Every
> asset is saved, so you can come back to it."

---

### 3:30 – 4:30 — Marketing Manager (the orchestration)

Click **Marketing Manager**. Tick **Use sample campaign CSV** and **Also draft
ad copy**. Optionally put `organic millet snacks` in the topic box. Click **Run
growth analysis**.

> "This is the whole point. The manager runs every specialist it has inputs for
> — campaign analysis, keyword research, content — then combines all their
> findings into one prioritised growth plan, ranked HIGH / MEDIUM / LOW."

Point at the "Agents run" line, the combined recommendations table, and the
growth plan text.

---

### 4:30 – 5:00 — Compare + close the loop

Click **Snapshot Compare**. Pick two snapshots, click **Compare**.

> "Every campaign run is a snapshot. Here it diffs two of them — and it knows
> the direction that matters: higher CTR is good, higher CPA is bad."

(For a dramatic demo, upload `sample_data/sample_campaign.csv` as snapshot 1 and
`sample_data/sample_campaign_v2.csv` as snapshot 2 — the second is a genuine
improvement, so the deltas light up green.)

Then click **Recommendations**:

> "Everything the agents suggested is tracked here — you can move each one from
> open to in-progress to done, and export the lot as CSV. That's the full loop:
> analyse → recommend → track → measure → re-measure."

**Close:** "Four agents, one manager, real data processing, real web extraction,
Sarvam for reasoning — and it runs end-to-end in one command."

---

## If a judge asks…

- **"Is the AI output real?"** — "With a Sarvam key, yes. Right now it's in
  demo mode, and every demo response is labelled `[DEMO MODE]` and flagged in
  the data. We never fake a live call."
- **"Where do the metrics come from?"** — "Pure Python in
  `services/csv_metrics.py`. The tests recompute them independently."
- **"What's not built?"** — "Search-volume data (no provider wired up, and we
  won't invent it), and live ad-platform APIs — we ingest CSV exports instead."
- **"Is the key safe?"** — "Environment variable only. Never stored, logged, or
  returned."

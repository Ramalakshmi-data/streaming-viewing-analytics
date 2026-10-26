# Streaming Viewing Analytics: Watch Next

**Which titles do the same subscribers spend their time on, and can that signal alone drive a useful "watch next" list?**

Two self-contained, interactive HTML pages built from 18,000 viewing records (2,000 subscribers × 9 titles).

🔗 **Live demo:** https://ramalakshmi-data.github.io/streaming-viewing-analytics/

![Project preview](docs/social-preview.png)

## What's inside

| Page | What it does |
|---|---|
| [`correlation-matrix.html`](correlation-matrix.html) | Heatmap of Pearson correlations between titles, ordered by clustered audience group, with a diverging scale and the value in every cell. |
| [`watch-next.html`](watch-next.html) | Tick the titles you've watched; the rest are ranked by their strongest audience link to your picks, with a plain-language reason for each. |
| [`index.html`](index.html) | Landing page linking both. |

| Desktop | Mobile |
|---|---|
| ![Recommender on desktop](docs/watch-next-desktop.png) | <img src="docs/matrix-mobile.png" width="240" alt="Heatmap on mobile"> |

## Method

- **Viewing time.** Minutes = episodes watched × episode length, per subscriber per title. Every subscriber has exactly one record per title.
- **Correlation.** Pearson *r* between every pair of titles across the 2,000 subscribers.
- **Audience groups.** Average-linkage agglomerative clustering on the correlation matrix, merging groups while their average pairwise *r* is ≥ 0.4. The heatmap's axis order and group dividers come from this.
- **Match score.** For each unwatched title: the strongest *r* with any title you've ticked, × 100, floored at 0. Ties are broken by the mean *r*. Titles with *r* < 0.2 are listed separately as having no meaningful link.

### Key findings

Four audience groups emerge, with almost no overlap between them (|*r*| ≤ 0.04 across groups):

| Group | Titles | Average *r* within group |
|---|---|---|
| 1 | Dahmer · You · Dexter | 0.90 |
| 2 | Stranger Things · Dark · Umbrella Academy | 0.88 |
| 3 | It's Okay To Not Be Okay · Queen Of Tears | 0.87 |
| 4 | Big Bang Theory | standalone audience |

The strongest single pair is **Dark ↔ Umbrella Academy** (*r* = 0.96).

### Design decision: max, not mean

An earlier version averaged a candidate's correlation over *everything* you'd watched and mapped −1…+1 onto 0–100. That had two problems:

- **Unrelated titles looked half-relevant.** *r* = 0 became a 50 "match".
- **Varied tastes diluted strong links.** Someone who watched Queen Of Tears *and* You saw It's Okay To Not Be Okay at only 72, because its strong link to Queen Of Tears was averaged with no link to You.

Using the strongest link (an item-based nearest-neighbour approach) gives it 87, and unrelated titles score 0–1.

## Engineering notes

- **One file per page:** data embedded, vanilla JavaScript, no libraries or network calls.
- **Computed in the browser:** only the per-subscriber minutes matrix is embedded (no IDs); correlation, clustering and ranking run client-side.
- **Accessible:**
  - Arrow-key navigation through heatmap cells, with a live readout.
  - A labelled checkbox fieldset, and focus management after "Mark watched".
  - All text meets WCAG AA contrast on every surface; a lighter red is used for small text.
- **Mobile:** the heatmap fits a 390 px screen, and recommendation cards reflow.
- **Genre tags are editorial.** They are not in the viewing data and play no part in any calculation; they're labelled that way on the pages.

## Rebuild from data

```bash
pip install pandas openpyxl
python src/build.py path/to/viewing.xlsx
```

The spreadsheet needs columns `Subscriber ID, Show Name, Episodes Watched, Episode Length (m)`. The raw file is not committed (see [`data/README.md`](data/README.md)).

## Disclaimer

Independent portfolio project, not affiliated with or endorsed by Netflix. Title names are used descriptively. Code is MIT-licensed.

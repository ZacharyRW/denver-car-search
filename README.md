# Denver Car Search Tracker

A phone- and web-viewable page that shows the latest used-car listings matching a
specific search (reliable / safe / efficient family car, under $10k, within 100 mi
of Aurora CO 80015 — hybrid, EV, or efficient gas).

## How it works

The **engine** and the **display** are deliberately separate:

- **Engine (runs on one always-on computer):** A scheduled task in the Claude
  desktop app drives a logged-in Google Chrome to search Cars.com, Craigslist, and
  **Facebook Marketplace** (which only works from a browser where you're signed in).
  It writes the results to `listings.json` and pushes the commit to this repo.
- **Display (runs anywhere):** GitHub Pages serves `index.html`, which loads
  `listings.json` and renders the ranked, sortable shortlist. Open the Pages URL
  from your phone, the web, or any computer to see the latest — Facebook included.

Facebook Marketplace **cannot** be scraped from GitHub Actions (login required,
CI IPs blocked, no public API), which is why the search runs on your desktop and
only the *results* live on GitHub.

## Repo structure

```
denver-car-search/
├── index.html          # the tracker page (GitHub Pages serves this)
├── listings.json       # current results — overwritten by each scheduled run
├── history/            # optional: dated JSON snapshots so you can see what changed
│   └── 2026-07-15.json
└── .github/workflows/
    └── validate.yml    # checks each listings.json push so a bad commit can't break the page
```

`index.html` is static and never needs editing — it just reads `listings.json`.
The daily task only rewrites the JSON, so updates are tiny, clean commits.

## Page features

- **Sortable columns** (click a header, or use the sort dropdown on a phone) and
  **fuel-type / text filters** to narrow the shortlist.
- **Card layout on phones** — no sideways scrolling.
- **✕ hide** on each row to dismiss listings you've ruled out (stored in the
  browser's localStorage, per device); "Show N hidden" brings them back.
- **NEW badge** when a row's `firstSeen` matches the latest `updated` date.
- **Stale-data warning** if the JSON is more than 36 hours old, so you know when
  the desktop task has stopped running.
- **Dark mode** follows the device setting.
- All listing text is HTML-escaped and only `http(s)` listing URLs are rendered,
  so a scraped title can't inject markup into the page.

## One-time setup

1. **Create the repo.** On GitHub, create a new repo named `denver-car-search`
   (public is simplest for Pages; private also works on any modern plan).
2. **Add these files.** Upload/commit `index.html`, `data/listings.json`, and this
   README (drag-and-drop in the GitHub web UI is fine, or `git push`).
3. **Enable Pages.** Repo → **Settings → Pages** → Source: **Deploy from a branch**,
   Branch: **main**, Folder: **/(root)** → Save. After a minute your site is live at:
   `https://<your-username>.github.io/denver-car-search/`
4. **Give the desktop task push access** (so it can update `listings.json`):
   - Easiest: connect the **GitHub connector** in Claude, or
   - Clone the repo locally on the always-on machine and let the task run
     `git commit` + `git push` in that folder using a saved credential / PAT.

   > Handle any GitHub token yourself — don't paste it into chat. Store it via
   > `git` credentials or the GitHub CLI (`gh auth login`) on the machine.

## Data format (`listings.json`)

```json
{
  "updated": "2026-07-15T08:00:00-06:00",
  "listings": [
    {
      "rank": 1,
      "veh": "2010 Toyota Prius",
      "type": "HYB",              // HYB | PHEV | EV
      "price": 7100,
      "miles": 170899,             // or null → shows "ask"
      "mpg": "48 mpg",
      "dist": 12,                  // miles from 80015
      "loc": "Littleton",
      "src": "Cars.com",
      "url": "https://…",          // direct link to the listing
      "why": "Short reason it made the list.",
      "firstSeen": "2026-07-15"    // if == updated date, shows a NEW badge
    }
  ]
}
```

The page marks a row **NEW** when its `firstSeen` matches the latest `updated` date,
so you can spot fresh listings at a glance.

## Notes

- The always-on machine must have the Claude desktop app open for the scheduled run
  to fire, and Chrome signed into Facebook.
- Viewing works on any device — nothing to install on your phone.
- Buying reminders (get a pre-purchase inspection, check hybrid battery health, watch
  EV winter range, avoid salvage titles) are built into the page.

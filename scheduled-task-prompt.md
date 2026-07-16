<!--
BACKUP of the daily scheduled task that powers this tracker.

WHAT THIS IS
  This is the exact prompt/instructions run every morning (~8:02 AM Mountain, cron "0 8 * * *")
  by a scheduled task in the Claude desktop app. It drives a logged-in Chrome to search the
  car sites, enriches the results via NHTSA, and commits listings.json + history/<date>.json +
  user-state.json to this repo. The live page (index.html) reads those files.

HOW TO RESTORE IT (if this computer is ever lost/replaced)
  In the Claude desktop app, ask Claude:
    "Create a daily scheduled task called denver-car-search-daily that runs at 8:00 AM,
     using the prompt in scheduled-task-prompt.md in my ZacharyRW/denver-car-search repo."
  or paste the body below (everything under the frontmatter) as the task prompt.
  Requires: a logged-in Chrome (Claude-in-Chrome extension) and access to commit to this repo.

  Original location on disk:
    ~/Documents/Claude/Scheduled/denver-car-search-daily/SKILL.md

Last synced to repo: 2026-07-15
-->

---
name: denver-car-search-daily
description: Daily used-car search for a reliable/safe/efficient family car under $10k near Aurora CO; captures photos/VIN/deal-rating/history flags, enriches recalls via NHTSA, publishes listings.json + dated history/ snapshot + user-state.json
---

Run a used-car search for Zachary, then publish the results to his GitHub Pages tracker (repo ZacharyRW/denver-car-search, branch main). The live page is https://zacharyrw.github.io/denver-car-search/ and it reads listings.json first, then dated history/ snapshots.

BUYER CRITERIA:
- Location: lives at Aurora, CO 80015; commutes to Sam's Club in Lone Tree, CO + daycare runs (~25 mi round trip on a workday).
- Budget: under $10,000 (slightly flexible for a standout).
- Wants: reliable, safe for a young child, good gas mileage. Hybrid or electric both welcome; efficient gas OK. Has Level 1 (regular outlet) home charging, so EVs are fine for the short commute.
- Search radius: up to 100 miles from 80015 (KSL/Utah only if explicitly stretching — flag the distance).
- Prioritize: Toyota Prius, Toyota Camry Hybrid, Honda/Ford hybrids, Chevy Volt (plug-in), Nissan Leaf (2017+ preferred for battery), Ford/Hyundai/Kia hybrids.

STEP 1 — SEARCH (use Google Chrome via the browser tools):
1. Cars.com: https://www.cars.com/shopping/results/?zip=80015&maximum_distance=100&list_price_max=10000&stock_type=used&fuel_slugs%5B%5D=hybrid&fuel_slugs%5B%5D=electric&sort=list_price
2. CarGurus: https://www.cargurus.com/Cars/inventorylisting/viewDetailsFilterViewInventoryListing.action?zip=80015&distance=100&maxPrice=10000&sortType=PRICE&sortDir=ASC (set fuel = hybrid/electric). CarGurus is the best source for the deal rating ("Great Deal") and the IMV market-value delta.
3. Craigslist Denver: https://denver.craigslist.org/search/cta?max_price=10000&query=hybrid&sort=priceasc (also repeat query=prius and query=leaf). Use list view; grab each posting's direct URL.
4. Facebook Marketplace (user is logged in): https://www.facebook.com/marketplace/denver/search?minPrice=3000&maxPrice=10000&sortBy=price_ascend&query=hybrid (radius 100 mi; also query=prius and query=nissan%20leaf). Also check Facebook dealer pages surfaced in results. Ignore $1/Free "parts out" listings.
5. Carvana: https://www.carvana.com/cars/hybrid?price=0-10000 (and /cars/electric). Carvana ships — note delivery to 80015.
6. eBay Motors: https://www.ebay.com/sch/6001/i.html?_nkw=hybrid&_udhi=10000&_stpos=80015&_sadis=100&LH_BIN=1 (Buy-It-Now within 100 mi; also prius, nissan+leaf).
7. OfferUp: https://offerup.com/search?q=prius&price_max=10000 (near Aurora CO; also hybrid, nissan leaf).
8. Hertz Car Sales / Enterprise Car Sales: https://www.hertzcarsales.com/ and https://www.enterprisecarsales.com/ — hybrids/EVs under $10k near 80015 (fleet cars, clean history, no-haggle).
9. KSL Cars (Utah — only if stretching the radius; flag distance clearly): https://cars.ksl.com/search/make/Toyota/model/Prius/priceTo/10000 (and hybrid/leaf).

For each relevant listing capture ALL of the following where shown (use null when a field isn't available — DO NOT invent values):
- year/make/model, price, mileage, location + distance from Aurora 80015, source, direct listing URL.
- photo: the URL of the listing's primary thumbnail image (read the <img> src, or right-click the photo -> copy image address). THIS IS THE SINGLE MOST VALUABLE NEW FIELD — grab it for every listing. Prefer a stable https image URL; skip data: URLs.
- vin: the VIN when shown (Cars.com/CarGurus/Carvana/dealer detail pages almost always list it on the detail page; Craigslist/FB/OfferUp usually don't).
- dealRating: the source's own deal label verbatim if present ("Great Deal", "Good Deal", "Fair Deal", "Overpriced"). Cars.com and CarGurus show these.
- imvDelta: CarGurus' price-vs-market number as a signed integer in dollars (below market = negative, e.g. -800 for "$800 under market"; above market = positive). null if not shown.
- daysListed: the source's own "listed N days ago" / days-on-market if shown (a number). This is the SELLER's days-on-market, separate from firstSeen (which the tracker computes itself).
- accident: the free history blurb dealers often show — "No accidents reported", "1 accident reported", etc. Capture verbatim; null if absent.
- title: title status if shown — "Clean", "Salvage", "Rebuilt", "Lemon". null if not stated. NEVER shortlist a salvage/rebuilt/junk title; screen those out.
Screen out salvage/parts/junk. Aim for the ~12 best by value across ALL sources (reliable model, low miles for price, close by, good deal rating / below IMV).

STEP 1b — ENRICH VIA NHTSA (free, no key; use the browser or a fetch):
For every listing that has a vin:
- Decode exact trim: https://vpic.nhtsa.dot.gov/api/vehicles/DecodeVinValues/<VIN>?format=json (use Trim / Series to sharpen the model text if useful).
For every listing (VIN or not), get open recalls by model-year:
- https://api.nhtsa.gov/recalls/recallsByVehicle?make=<MAKE>&model=<MODEL>&modelYear=<YEAR>
- Set recalls = the count of returned campaigns (results array length). 0 is a valid, meaningful value (report it, don't null it). If the API errors, use null.

STEP 1c — DEDUP:
The same car often appears on multiple sources (e.g. Cars.com + CarGurus, or a dealer's own FB page). Deduplicate:
- If two listings share the same vin, they are the SAME car — keep one entry (prefer the source with the richest data / best price), and you may note the other source in "why".
- If no VIN, treat same year+make+model+mileage+price within the same city as a probable duplicate and collapse it.

STEP 2 — PUBLISH TO GITHUB (repo ZacharyRW/denver-car-search, branch main):
Build the results as this exact JSON shape (call this the DATA):
{
  "updated": "<current ISO8601 timestamp with -06:00 or -07:00 MT offset>",
  "criteria": {"budget_max":10000,"zip":"80015","radius_miles":100},
  "listings": [ {"rank":1,"veh":"YEAR MAKE MODEL","type":"HYB|PHEV|EV","price":0,"miles":0,"mpg":"","dist":0,"loc":"","src":"","url":"","photo":null,"vin":null,"dealRating":null,"imvDelta":null,"daysListed":null,"accident":null,"title":null,"recalls":null,"why":"","firstSeen":"YYYY-MM-DD"} ]
}
Rules for the data:
- "type" must be HYB, PHEV, or EV (use HYB for a plain efficient gas car too, or note in why).
- "miles" is a number or null. "recalls"/"imvDelta"/"daysListed" are numbers or null. "photo"/"vin"/"dealRating"/"accident"/"title" are strings or null.
- Include EVERY field on every listing (use null for missing) so the page can rely on the shape.
- "firstSeen": FIRST fetch the current listings.json from the repo. For any listing whose url (or vin) already appears there, KEEP its existing firstSeen date. For genuinely new listings, set firstSeen to today's date. The tracker shows a red NEW badge when firstSeen == the run date.
- Rank best-value first.

Now write THREE files to the repo:
1. listings.json (repo root) — the current feed, overwritten every run (the DATA above). This is what the live page reads first.
2. history/YYYY-MM-DD.json — a dated snapshot for today, identical content to listings.json, in the history/ folder. The filename MUST use HYPHENS (e.g. history/2026-07-15.json) — index.html fetches these as history/<YYYY-MM-DD>.json (from toISOString().slice(0,10)); an underscore filename would silently break price deltas, sparklines, and GONE rows. Compute the date from the SAME Mountain-Time timestamp used in "updated". If today's snapshot already exists, OVERWRITE it (a re-run refreshes, not duplicates).
3. user-state.json (repo root) — the cross-device store for the user's pins/notes/status stages. IMPORTANT: this file is owned by the USER via the page, not by you. FIRST fetch the existing user-state.json from the repo. If it exists, RE-COMMIT IT UNCHANGED (never clobber the user's stars/notes/pipeline stages). If it does NOT exist yet, create it once as: {"version":1,"updated":"<ISO timestamp>","cars":{}} and commit that empty seed. Do not otherwise modify it.

Commit the files to ZacharyRW/denver-car-search on branch main using the GitHub connector (create-or-update file; one or more commits is fine), message like "Update listings + history snapshot <date>". If the GitHub connector is unavailable, instead drive Chrome to github.com/ZacharyRW/denver-car-search/edit/main/listings.json (and .../new/main for history/<date>.json) and commit. Do not paste any GitHub token into chat.

STEP 3 — REPORT: Post a short ranked summary in chat highlighting anything NEW vs the previous listings.json, one line each with the direct link, source, deal rating, and any below-market IMV. Note EV winter-range and hybrid-battery caveats only where relevant. Confirm listings.json, today's history/<date>.json, and user-state.json were committed. Do not contact any seller or take any action beyond searching, committing the files, and reporting.

The live page updates automatically once listings.json is committed: https://zacharyrw.github.io/denver-car-search/

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

Last synced to repo: 2026-07-16

NOTE: this repo copy is now the source of truth — after editing it here, update the
desktop task's SKILL.md to match (or re-create the task from this file).
-->

---
name: denver-car-search-daily
description: Daily used-car search for a reliable/safe/efficient family car under $10k near Aurora CO; captures photos/VIN/deal-rating/history flags/seller type, enriches recalls + crash ratings via NHTSA and MPG via FuelEconomy.gov, reports per-source status, publishes listings.json + dated history/ snapshot + user-state.json
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
4. Facebook Marketplace (user is logged in) — DO NOT GIVE UP EARLY ON THIS SOURCE. Manual searching here regularly yields more real listings than any other source; if you get zero results, that is almost always a loading/scrolling problem on your end, not an empty market. Work it like this:
   - Run EACH of these queries (radius 100 mi): https://www.facebook.com/marketplace/denver/search?minPrice=3000&maxPrice=10000&sortBy=price_ascend&query=hybrid — then repeat with query=prius, query=nissan%20leaf, query=chevy%20volt, query=camry%20hybrid, query=fusion%20hybrid.
   - Marketplace lazy-loads: after the page settles, SCROLL DOWN at least 4–5 times per query and wait for new tiles to render before reading results. The first screen alone is not the result set.
   - If a query shows nothing or errors, RELOAD and retry it once more before moving on. Only after a reload + rescroll still shows an empty grid or a login wall may you mark Facebook as blocked — and then you MUST say so in the "sources" status (below) instead of failing silently.
   - Click into promising listings for price/mileage/photo/location detail. Also check Facebook dealer pages surfaced in results. Ignore $1/Free "parts out" listings.
5. Carvana: https://www.carvana.com/cars/hybrid?price=0-10000 (and /cars/electric). Carvana ships — note delivery to 80015.
6. eBay Motors: https://www.ebay.com/sch/6001/i.html?_nkw=hybrid&_udhi=10000&_stpos=80015&_sadis=100&LH_BIN=1 (Buy-It-Now within 100 mi; also prius, nissan+leaf).
7. OfferUp: https://offerup.com/search?q=prius&price_max=10000 (near Aurora CO; also hybrid, nissan leaf).
8. Hertz Car Sales: https://www.hertzcarsales.com/ — hybrids/EVs under $10k near 80015 (fleet cars, clean history, no-haggle).
9. Enterprise Car Sales — search this one every run, don't skip it: https://www.enterprisecarsales.com/used-cars-trucks-suvs/search.html?location=Denver%2C+CO — set max price $10,000 and filter fuel to hybrid/electric (use the site's own search near Denver/Aurora if that URL layout changes). Fleet cars with clean history, no-haggle pricing, and a 7-day buyback; their under-$10k hybrids sell fast, so capture them the day they appear. sellerType = "Dealer".
10. KSL Cars (Utah — only if stretching the radius; flag distance clearly): https://cars.ksl.com/search/make/Toyota/model/Prius/priceTo/10000 (and hybrid/leaf).

For each relevant listing capture ALL of the following where shown (use null when a field isn't available — DO NOT invent values):
- year/make/model, price, mileage, location + distance from Aurora 80015, source, direct listing URL.
- year, make, model as SEPARATE structured fields (year = number, make/model = strings, e.g. 2015, "Ford", "C-Max Energi") in addition to the combined "veh" string — the page builds NHTSA/recall links from these, and parsing them back out of "veh" breaks on multi-word names.
- sellerType: "Dealer" or "Private". Dealer for any dealership/fleet/Carvana/Hertz/Enterprise listing; Private for by-owner Craigslist/Facebook/OfferUp posts. null only if genuinely unclear.
- photo: the URL of the listing's primary thumbnail image (read the <img> src, or right-click the photo -> copy image address). THIS IS THE SINGLE MOST VALUABLE NEW FIELD — grab it for every listing. Prefer a stable https image URL; skip data: URLs.
- vin: the VIN when shown (Cars.com/CarGurus/Carvana/dealer detail pages almost always list it on the detail page; Craigslist/FB/OfferUp usually don't).
- dealRating: the source's own deal label verbatim if present ("Great Deal", "Good Deal", "Fair Deal", "Overpriced"). Cars.com and CarGurus show these.
- imvDelta: CarGurus' price-vs-market number as a signed integer in dollars (below market = negative, e.g. -800 for "$800 under market"; above market = positive). null if not shown.
- daysListed: the source's own "listed N days ago" / days-on-market if shown (a number). This is the SELLER's days-on-market, separate from firstSeen (which the tracker computes itself).
- accident: the free history blurb dealers often show — "No accidents reported", "1 accident reported", etc. Capture verbatim; null if absent.
- title: title status if shown — "Clean", "Salvage", "Rebuilt", "Lemon". null if not stated. NEVER shortlist a salvage/rebuilt/junk title; screen those out.
- carSeatNotes: one sentence on this SPECIFIC car for a rear-facing child seat — rear-seat room, LATCH access, door opening, hatchback cargo for daycare bags (e.g. "Midsize rear seat fits rear-facing easily" vs "Subcompact — verify front-passenger clearance with the seat installed").
- nextAction: the single most useful next step for THIS listing (e.g. "Ask private seller for VIN + title photo before driving out", "Dealer — request the free Carfax link and recall-work receipts", "Ask for a LeafSpy battery report before viewing").
Screen out salvage/parts/junk. Aim for the ~12 best by value across ALL sources (reliable model, low miles for price, close by, good deal rating / below IMV).

STEP 1b — ENRICH VIA FREE APIS (no key needed; use the browser or a fetch). Cache per year+make+model — listings sharing a model-year share these values, so one lookup covers them all:
For every listing that has a vin:
- Decode exact trim: https://vpic.nhtsa.dot.gov/api/vehicles/DecodeVinValues/<VIN>?format=json (use Trim / Series to sharpen the model text if useful).
For every listing (VIN or not), get open recalls by model-year:
- https://api.nhtsa.gov/recalls/recallsByVehicle?make=<MAKE>&model=<MODEL>&modelYear=<YEAR>
- Set recalls = the count of returned campaigns (results array length). 0 is a valid, meaningful value (report it, don't null it). If the API errors, use null.
For every listing, get the official NHTSA crash-test rating:
- https://api.nhtsa.gov/SafetyRatings/modelyear/<YEAR>/make/<MAKE>/model/<MODEL> — take the first VehicleId from Results, then fetch https://api.nhtsa.gov/SafetyRatings/VehicleId/<VehicleId> and read OverallRating.
- Set nhtsaStars = that number (1–5). If the rating is "Not Rated" or the API returns nothing, use null. Do not guess.
For every listing, get official EPA fuel economy:
- https://www.fueleconomy.gov/ws/rest/vehicle/menu/options?year=<YEAR>&make=<MAKE>&model=<MODEL> — pick the option matching the trim (prefer automatic), then fetch https://www.fueleconomy.gov/ws/rest/vehicle/<ID> and read comb08 (combined MPG; for EVs this is MPGe).
- Set epaMpg = that number. If no match, use null. Keep the human "mpg" string too — epaMpg is the machine-readable version the page uses for $/mo math and scoring.

STEP 1c — DEDUP:
The same car often appears on multiple sources (e.g. Cars.com + CarGurus, or a dealer's own FB page). Deduplicate:
- If two listings share the same vin, they are the SAME car — keep one entry (prefer the source with the richest data / best price), and you may note the other source in "why".
- If no VIN, treat same year+make+model+mileage+price within the same city as a probable duplicate and collapse it.

STEP 2 — PUBLISH TO GITHUB (repo ZacharyRW/denver-car-search, branch main):
Build the results as this exact JSON shape (call this the DATA):
{
  "updated": "<current ISO8601 timestamp with -06:00 or -07:00 MT offset>",
  "criteria": {"budget_max":10000,"zip":"80015","radius_miles":100},
  "sources": [ {"src":"Cars.com","status":"ok","count":6,"note":""} ],
  "listings": [ {"rank":1,"veh":"YEAR MAKE MODEL","year":0,"make":"","model":"","type":"HYB|PHEV|EV","price":0,"miles":0,"mpg":"","epaMpg":null,"dist":0,"loc":"","src":"","sellerType":null,"url":"","photo":null,"vin":null,"dealRating":null,"imvDelta":null,"daysListed":null,"accident":null,"title":null,"recalls":null,"nhtsaStars":null,"carSeatNotes":null,"nextAction":null,"why":"","firstSeen":"YYYY-MM-DD"} ]
}
Rules for the data:
- "sources" is the honesty ledger and MUST have one entry for EVERY source in STEP 1, every run — even the ones that produced nothing. status is one of: "ok" (searched, results read), "empty" (searched fine, nothing matched), "blocked" (login wall / bot block / wouldn't load after a retry), "error" (site broken), "skipped" (deliberately not searched — say why in note). count = listings that made the final shortlist from that source. The page shows this strip so the user can tell "no Facebook cars" apart from "Facebook wasn't really searched". Never report ok for a source you didn't actually read.
- "type" must be HYB, PHEV, or EV (use HYB for a plain efficient gas car too, or note in why).
- "miles" is a number or null. "recalls"/"imvDelta"/"daysListed"/"year"/"epaMpg" are numbers or null. "nhtsaStars" is 1–5 or null. "photo"/"vin"/"dealRating"/"accident"/"title"/"make"/"model"/"carSeatNotes"/"nextAction" are strings or null. "sellerType" is "Dealer", "Private", or null.
- Include EVERY field on every listing (use null for missing) so the page can rely on the shape.
- "firstSeen": FIRST fetch the current listings.json from the repo. For any listing whose url (or vin) already appears there, KEEP its existing firstSeen date. For genuinely new listings, set firstSeen to today's date. The tracker shows a red NEW badge when firstSeen == the run date.
- Rank best-value first.

Now write THREE files to the repo:
1. listings.json (repo root) — the current feed, overwritten every run (the DATA above). This is what the live page reads first.
2. history/YYYY-MM-DD.json — a dated snapshot for today, identical content to listings.json, in the history/ folder. The filename MUST use HYPHENS (e.g. history/2026-07-15.json) — index.html fetches these as history/<YYYY-MM-DD>.json (from toISOString().slice(0,10)); an underscore filename would silently break price deltas, sparklines, and GONE rows. Compute the date from the SAME Mountain-Time timestamp used in "updated". If today's snapshot already exists, OVERWRITE it (a re-run refreshes, not duplicates).
3. user-state.json (repo root) — the cross-device store for the user's pins/notes/status stages. IMPORTANT: this file is owned by the USER via the page, not by you. FIRST fetch the existing user-state.json from the repo. If it exists, RE-COMMIT IT UNCHANGED (never clobber the user's stars/notes/pipeline stages). If it does NOT exist yet, create it once as: {"version":1,"updated":"<ISO timestamp>","cars":{}} and commit that empty seed. Do not otherwise modify it.

Commit the files to ZacharyRW/denver-car-search on branch main using the GitHub connector (create-or-update file; one or more commits is fine), message like "Update listings + history snapshot <date>". If the GitHub connector is unavailable, instead drive Chrome to github.com/ZacharyRW/denver-car-search/edit/main/listings.json (and .../new/main for history/<date>.json) and commit. Do not paste any GitHub token into chat.

STEP 3 — REPORT: Post a short ranked summary in chat highlighting anything NEW vs the previous listings.json, one line each with the direct link, source, deal rating, and any below-market IMV. Include a one-line source roll-call (e.g. "Cars.com ✓6 · Facebook blocked · Enterprise ✓2") matching the "sources" array. Note EV winter-range and hybrid-battery caveats only where relevant. Confirm listings.json, today's history/<date>.json, and user-state.json were committed. Do not contact any seller or take any action beyond searching, committing the files, and reporting.

The live page updates automatically once listings.json is committed: https://zacharyrw.github.io/denver-car-search/

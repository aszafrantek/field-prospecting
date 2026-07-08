# Field Prospecting Tool — Claude Context

This repo is a single-file iPad-ready prospecting tool for Tekmetric field sales reps. It lives at **https://aszafrantek.github.io/field-prospecting/** and is deployed via GitHub Pages from `index.html`.

Claude should read this file at the start of every session and follow all conventions exactly. Do not invent new formats — match what is already in the file.

---

## Architecture

- **One file**: everything lives in `index.html` — HTML, CSS, and JS in a single file (~270 KB)
- **GitHub Pages**: deploys automatically from `main` branch via `.github/workflows/pages.yml`
- **Firebase Firestore**: stores each rep's visit notes and shop statuses, project `tekmetric-prospecting`
- **No build step**: plain HTML/JS, validated with `node --check` before every deploy

---

## Deploying Changes

Never use git CLI — the sandbox has no GitHub credentials. Always deploy by:

1. Validate JS first: extract `<script>` tag contents → write to `/tmp/validate.js` → run `node --check /tmp/validate.js`
2. Navigate Chrome to `https://github.com/aszafrantek/field-prospecting/upload/main`
3. Upload `index.html` via the file input (`ref_143`)
4. Set commit summary via `document.querySelector('input[placeholder="Commit summary"]')`
5. Click commit: `[...document.querySelectorAll('button')].find(b => b.textContent.trim() === 'Commit changes').click()`
6. GitHub Pages redeploys in ~60 seconds

---

## Data Structure

All prospect data lives inside the `<script>` tag in three JS objects: `PLANS`, `NO_VISIT`, and `PLAN_ORIGINS`.

### PLANS — prospect shops

```javascript
const PLANS = {
  "Plan Name Here": [
    {id:"unique-slug",shop:"Shop Name",day:"Day 1",address:"123 Main St, City ST 00000",phone:"(xxx) xxx-xxxx",tags:["Technet","BG"],stars:"4.8",reviewInfo:"210 reviews",notes:"Visit notes here"}
  ]
};
```

**Rules:**
- `id` must be a unique kebab-case slug across the entire file — check for collisions before adding
- `day` must be exactly `"Day 1"`, `"Day 2"`, etc. — no other format
- `tags` is an array — see Tags section below for all valid values
- `stars` is a string (e.g. `"4.8"`) or `null`
- `reviewInfo` is a string or `null`
- `notes` is a string — use `·` (middle dot) as separator within notes
- No trailing commas after the last shop in a plan array
- After removing a shop, check for double-commas `,,` and fix them

### NO_VISIT — existing Tekmetric customers

```javascript
const NO_VISIT = {
  "Plan Name Here": [
    {shop:"Shop Name",city:"City, ST",address:"123 Main St",notes:"Existing Tekmetric customer — reason here"}
  ]
};
```

When a shop is confirmed as an existing customer, remove it from `PLANS` and add it to `NO_VISIT` under the same plan key.

### PLAN_ORIGINS — starting point for Google Maps routing

```javascript
const PLAN_ORIGINS = {
  "Plan Name Here": "City, ST"
};
```

Every plan in `PLANS` must have a matching entry here. The value is the anchor city/hotel for that trip (e.g. TopGolf location or downtown hotel).

---

## Tags — EXACT casing required

Wrong casing breaks the visual badge rendering. Use these exact strings:

| Tag | Meaning |
|-----|---------|
| `"Mitchell"` | Mitchell1 CRM confirmed (SureCritic or direct) |
| `"Technet"` | TechNet Professional Automotive Service member — **lowercase 'n'** |
| `"BG"` | BG Products affiliate |
| `"AutoTechIQ"` | AutoTechIQ certified shop |
| `"Marquee"` | High-value Mitchell1 shop (500+ SureCritic reviews) |
| `"MSO"` | Multi-shop operation |
| `"Euro"` | European vehicle specialist |
| `"Fisher"` | Fisher Auto Parts affiliate |
| `"FMP"` | FMP Partners Network affiliate |
| `"RepairPal"` | RepairPal certified |
| `"CARFAX"` | CARFAX Service Shop |
| `"Assoc"` | State association member |
| `"Caveat"` | ⚠ Verify before visiting — no confirmed CRM signal |
| `"TirePros"` | Tire Pros buying group member |
| `"AAA"` | AAA Approved Auto Repair |
| `"NAPA"` | NAPA AutoCare affiliate |
| `"ASE"` | ASE certified |
| `"BBB"` | BBB Accredited |
| `"Goodyear"` | Goodyear authorized dealer |

**Critical:** `"Technet"` not `"TechNet"` — the filter dropdown uses `"Technet"` as the option value.

---

## Adding a New Territory

### Step 1: Research
Use web search to find 8–12 independent auto repair shops per day. No national chains (no Midas, Meineke, Pep Boys, Firestone, Jiffy Lube, Mavis, NTB, Valvoline). For each shop collect:
- Full street address with ZIP
- Phone number
- Google star rating + review count
- Any affiliations (Mitchell1/SureCritic, AutoTechIQ, TechNet, BG, NAPA, AAA, etc.)
- Brief notes (specialty, ownership history, notable signals)

### Step 2: Check TechNet
Search `https://www.technetprofessional.com/find-shop` by ZIP code for each day's area. Tag any matches with `"Technet"`. Add TechNet shops not already in the plan as new entries.

### Step 3: Check BG Products
Search `https://www.bgprod.com/find-a-shop/` by ZIP. Tag matches with `"BG"`.

### Step 4: Plan structure
Organize shops into days based on geography (cluster stops within the same corridor). Name the plan descriptively, e.g. `"National Harbor MD 6-Day"`. Add to:
1. `PLANS` — shop entries with correct day labels
2. `NO_VISIT` — empty array `[]` initially
3. `PLAN_ORIGINS` — the TopGolf/hotel anchor city

### Step 5: Validate and deploy
Run `node --check` on the extracted script. Fix any syntax errors. Deploy via GitHub upload.

---

## Day Structure Conventions

- Day 1 = closest to the anchor (TopGolf or hotel)
- Subsequent days radiate outward geographically
- Add a comment before each day's shops: `// ── Day N — Area Description ──`
- Field visit notes from real trips go at the end of the plan array with the note format: `"YYYY visit (date time) — [CRM system]. [observations]. [contact name] — [email]"`

---

## JS Validation

Always validate before deploying. Extract script and check:

```python
with open('index.html', 'r') as f:
    html = f.read()
start = html.find('<script>')
script = html[start+8:html.rfind('</script>')]
with open('/tmp/validate.js', 'w') as f:
    f.write(script)
# Then: node --check /tmp/validate.js
```

Common errors after edits:
- Double commas `,,` after removing a shop entry — replace with `,`
- Missing comma between entries — add `,` before `{shop:`
- Wrong tag casing — `"TechNet"` should be `"Technet"`

---

## Existing Plans (as of July 2026)

| Plan | Origin | Days |
|------|--------|------|
| Cape Cod / Salem NH 3-Day | Hyannis, MA | 3 |
| Portsmouth / Seacoast NH 2-Day | Downtown Portsmouth, NH | 2 |
| Portland ME 3-Day | Portland, ME | 3 |
| New Jersey — Parsippany 10-Day | Parsippany, NJ | 10 |
| Mobile AL 10-Day | Downtown Mobile, AL | 10 |
| Winston-Salem 3-Day | Downtown Winston-Salem, NC | 3 |
| Manchester / Concord NH 3-Day | Manchester, NH | 3 |
| Nashua / South NH 3-Day | Nashua, NH | 3 |
| North Shore MA 3-Day | Salem, MA | 3 |
| Vermont 2-Day | Lebanon, NH | 2 |
| Rhode Island 2-Day | Providence, RI | 2 |
| Greater Greensboro NC 6-Day | Downtown Greensboro, NC | 6 |
| National Harbor MD 6-Day | Oxon Hill, MD | 6 |

---

## Team Config (in script, top of file)

```javascript
const MY_PLANS = null;       // null = show all; or ["Plan Name"] to hardcode filter
const MY_SYNC_PREFIX = "";   // "" = no prefix; or "sarah-" to namespace Firebase notes
```

These are the hardcoded defaults. Reps can override via the ⚙ settings UI in the app (no code edit needed) — settings are stored in their browser's localStorage.

---

## Firebase

- Project: `tekmetric-prospecting`
- Data path: `sessions/{SYNC_KEY}/shops/{shopId}`
- `SYNC_KEY` is auto-generated per device (UUID), optionally prefixed with rep name
- Visit notes and statuses are per-rep — no shared state between reps
- Reps can sync across their own devices using the 🔗 share button

---

## Cross-referencing Existing Customers

Before finalizing a new territory, cross-reference all prospect shops against the NEW EXISTING USERS Google Sheet (accessible from an authenticated browser tab using the gviz/tq API). Any confirmed matches:
1. Remove from `PLANS`
2. Add to `NO_VISIT` with note: `"Existing Tekmetric customer — [source]"`
3. Exception: if a shop has a confirmed demo booking date but is NOT in the existing customers list, keep them in `PLANS` (they booked but didn't convert)

---

## Notes Style Guide

- Use `·` (middle dot U+00B7) to separate note clauses: `"Family-owned since 1978 · ASE certified · M–F 8–5"`
- Hours format: `M–F 8–5` or `M–Sat 8–4`
- Field visit notes start with: `"YYYY visit (M/D time) — "`
- Demo booked format: `"DEMO BOOKED M/D Xam · Contact name"`
- Marquee shops: add `★★ MARQUEE` at end of notes
- Caveat shops: add `⚠` at end of notes

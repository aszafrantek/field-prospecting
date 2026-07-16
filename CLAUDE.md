# Field Prospecting Tool — Claude Context

This repo is a single-file iPad-ready prospecting tool for Tekmetric field sales reps. It deploys via GitHub Pages from `index.html` and is accessible at **https://[YOUR_GITHUB_USERNAME].github.io/field-prospecting/**.

Claude should read this file at the start of every session and follow all conventions exactly. Do not invent new formats — match what is already in the file.

---

## New Rep Quickstart

This repo is a **GitHub template**. Every rep gets their own copy — their own URL, their own plans, their own data.

### Step 1 — Create your repo from the template

1. Go to **https://github.com/aszafrantek/field-prospecting**
2. Click **"Use this template"** → **"Create a new repository"**
3. Set repository name to `field-prospecting`
4. Set visibility to **Public** (required for free GitHub Pages)
5. Click **"Create repository"**

### Step 2 — Enable GitHub Pages

1. In your new repo, go to **Settings → Pages**
2. Under "Source", select **Deploy from a branch**
3. Branch: `main` / folder: `/ (root)` → Save
4. Your app will be live at `https://[your-github-username].github.io/field-prospecting/` within ~60 seconds

### Step 3 — Open a Claude (Cowork) session and paste this prompt

```
You are helping me set up a field prospecting tool for Tekmetric auto repair sales.

My GitHub username is: [YOUR_GITHUB_USERNAME]

Fetch my copy of the tool:
  https://raw.githubusercontent.com/[YOUR_GITHUB_USERNAME]/field-prospecting/main/index.html

Save it as index.html. Then read CLAUDE.md at:
  https://raw.githubusercontent.com/[YOUR_GITHUB_USERNAME]/field-prospecting/main/CLAUDE.md

First, clear out all existing plans from PLANS, NO_VISIT, and PLAN_ORIGINS — they belong
to a different rep. Set all three to empty objects: PLANS = {}; NO_VISIT = {}; PLAN_ORIGINS = {};

Then build me a plan for my territory: [YOUR CITY/REGION, N days].
Deploy to: https://github.com/[YOUR_GITHUB_USERNAME]/field-prospecting/upload/main
```

### Step 4 — Open the app on your iPad

Go to your GitHub Pages URL. Tap ⚙ (settings, top right) and set:
- **Your name / sync prefix** — e.g. `evan-` — this namespaces your visit notes in Firebase so they don't mix with other reps
- **My plans** — optionally restrict to just your plan names

Your status and notes sync across your own devices via the 🔗 share button. They are not visible to other reps.

### Step 5 — Field workflow

At the end of each day, tell Claude: **"Log today's Granola notes"**. Claude will pull your meeting notes from Granola, match them to shops in your plan, and update the notes for each visited shop, then redeploy.

---

## Architecture

- **One file**: everything lives in `index.html` — HTML, CSS, and JS in a single file (~300–550 KB)
- **GitHub Pages**: deploys automatically from `main` branch
- **Firebase Firestore**: stores each rep's visit notes and shop statuses, project `tekmetric-prospecting`
- **No build step**: plain HTML/JS, validated with `node --check` before every deploy

---

## Deploying Changes

Never use git CLI — always deploy via Chrome:

1. Validate JS first: extract `<script>` tag → write to `/tmp/validate.js` → run `node --check /tmp/validate.js`
2. Navigate Chrome to `https://github.com/[USERNAME]/field-prospecting/upload/main`
3. Read the page with `read_page` to get fresh element refs
4. Upload `index.html` via the file input ref
5. Click the commit message field, type a short message
6. Click "Commit changes"
7. GitHub Pages redeploys in ~60 seconds

---

## Data Structure

All prospect data lives inside the `<script>` tag in three JS objects: `PLANS`, `NO_VISIT`, and `PLAN_ORIGINS`.

### PLANS — prospect shops

```javascript
const PLANS = {
  "Plan Name Here": [
    {id:"unique-slug", shop:"Shop Name", day:"Day 1", address:"123 Main St, City ST 00000",
     phone:"(xxx) xxx-xxxx", tags:["Technet","BG"], stars:"4.8", reviewInfo:"210 reviews",
     notes:"Visit notes here"}
  ]
};
```

**Rules:**
- `id` must be a unique kebab-case slug across the entire file — check for collisions before adding
- `day` must be exactly `"Day 1"`, `"Day 2"`, etc.
- `tags` is an array — see Tags section below for all valid values
- `stars` is a string (e.g. `"4.8"`) or `null`
- `notes` is a string — use `·` (middle dot U+00B7) as separator within notes
- No trailing commas after the last shop in a plan array
- After removing a shop, check for double-commas `,,` and fix them
- For confirmed demo-booked shops, add `defaultStatus:"won"` to the entry

### NO_VISIT — existing Tekmetric customers

```javascript
const NO_VISIT = {
  "Plan Name Here": [
    {shop:"Shop Name", city:"City, ST", address:"123 Main St",
     notes:"Existing Tekmetric customer — reason here"}
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

## Status Values

| Value | Display |
|-------|---------|
| `"new"` | Not yet visited |
| `"visited"` | Visited, no follow-up yet |
| `"followup"` | Follow-up needed |
| `"won"` | 📅 Demo Booked |
| `"not-interested"` | Declined |
| `"skip"` | Skip this stop |

Stat bar counts:
- **Visited** = visited + follow-up + demo booked + not-interested (any physical stop)
- **Demo Booked** = won only

---

## Adding a New Territory

### Step 1: Research
Find 8–12 independent auto repair shops per day. No national chains (no Midas, Meineke, Pep Boys, Firestone, Jiffy Lube, Mavis, NTB, Valvoline). For each shop collect:
- Full street address with ZIP
- Phone number
- Google star rating + review count
- Any affiliations (Mitchell1/SureCritic, AutoTechIQ, TechNet, BG, NAPA, AAA, etc.)
- Brief notes (specialty, ownership, notable signals)

Target profile: 4+ stars, independent, owner-operated, 2–15 bays, general repair or specialty.

### Step 2: Check TechNet
Search `https://www.technetprofessional.com/find-shop` by ZIP for each day's area. Tag matches with `"Technet"`. Add any TechNet shops not already in the plan.

### Step 3: Check BG Products
Search `https://www.bgprod.com/find-a-shop/` by ZIP. Tag matches with `"BG"`.

### Step 4: Check AutoTechIQ
Search `https://www.autotechiq.com/find-a-shop`. Tag matches with `"AutoTechIQ"`.

### Step 5: Plan structure
- Name the plan: `"[City/Region] [N]-Day"` — e.g. `"Columbus OH 5-Day"`
- Day 1 = closest to anchor hotel/TopGolf
- Subsequent days radiate outward geographically
- Add to `PLANS`, `NO_VISIT` (empty array `[]`), and `PLAN_ORIGINS`
- Add day comments: `// ── Day N — Area Description ──`

### Step 6: Cross-reference existing customers
Before finalizing, cross-reference all prospect shops against the existing customers Google Sheet (accessible via the gviz/tq API from an authenticated browser tab). Any confirmed matches:
1. Remove from `PLANS`
2. Add to `NO_VISIT` with note: `"Existing Tekmetric customer"`

### Step 7: Validate and deploy

---

## JS Validation

Always validate before deploying:

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
- Double commas `,,` after removing a shop — replace with `,`
- Wrong tag casing — `"TechNet"` should be `"Technet"`
- Escaped quotes inside notes can break regex — use index-based string slicing

---

## Editing Notes Safely

Notes fields can contain escaped quotes (`\"`). Always use index-based slicing, never regex:

```python
idx_start = html.find('{id:"shop-id"')
notes_start = html.find(',notes:"', idx_start)
entry_end = html.find('"}', notes_start + 8)
old_section = html[notes_start:entry_end+2]
new_section = ',notes:"' + clean_notes + '"}'
html = html[:notes_start] + new_section + html[entry_end+2:]
```

---

## Day Structure Conventions

- Day 1 = closest to the anchor (TopGolf or hotel)
- Subsequent days radiate outward geographically
- Comment before each day: `// ── Day N — Area Description ──`
- Field visit notes format: `"VISITED M/D/YY — [observations]. [CRM system]. [contact / email]. [next step]."`
- Demo booked format: `"DEMO BOOKED: [date/time] · [format e.g. Google Meet / in-person]"`

---

## Logging Field Notes from Granola

At end of day, say: **"Log today's Granola notes"**

Claude will:
1. Pull meetings from the Granola MCP for the current day
2. Filter to shop visits only (skip internal meetings, 1:1s)
3. Match each meeting to a shop by name in `PLANS`
4. Update the `notes` field using index-based slicing
5. Validate JS and deploy

---

## Team Config (in script, top of file)

```javascript
const MY_PLANS = null; // null = show all; or ["Plan Name"] to hardcode filter
const MY_SYNC_PREFIX = ""; // "" = no prefix; or "evan-" to namespace Firebase notes
```

Reps can override via the ⚙ settings UI — stored in browser localStorage, no code edit needed.

---

## Firebase

- Project: `tekmetric-prospecting`
- Data path: `sessions/{SYNC_KEY}/shops/{shopId}`
- `SYNC_KEY` is auto-generated per device (UUID), optionally prefixed with rep name
- Visit notes and statuses are per-rep — no shared state between reps
- Reps sync across their own devices via the 🔗 share button

---

## Notes Style Guide

- Use `·` (middle dot U+00B7) to separate clauses: `"Family-owned since 1978 · ASE certified · M–F 8–5"`
- Hours format: `M–F 8–5` or `M–Sat 8–4`
- Field visit notes start with: `"VISITED M/D/YY — "`
- Demo booked: append `"DEMO BOOKED: [date/time] · [format]"` to notes + add `defaultStatus:"won"` to entry
- Marquee shops: add `★★ MARQUEE` at end of notes
- Caveat shops: add `⚠` at end of notes

---

## Quality Signals (prioritize these)

- **Mitchell1 / SureCritic**: any shop with 50+ reviews is an active Mitchell1 customer
- **500+ SureCritic reviews**: tag `"Marquee"` — high-value target
- **TechNet**: already in an industry network — receptive to tools
- **AutoTechIQ**: tech-forward, growth-oriented
- **4.8+ Google stars, 100+ reviews**: quality indicator
- **NAPA AutoCare / AAA Approved**: established, credible shops
- **Owner-operated, 3–10 bays**: ideal Tekmetric buyer profile
- **Multi-location (MSO)**: tag `"MSO"` — higher ARR opportunity

Avoid: tire-only shops, transmission-only, body shops, dealers, fleet-only.

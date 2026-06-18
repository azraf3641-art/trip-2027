# The Trip · 2027 Shortlist — shared voting site

A single static page (`index.html`) that shows the venue shortlist and collects
votes. **No backend, no build step, no database, no install, no secrets in the
page.** The only third-party content is Google Fonts and one embedded results
chart (your own published Google chart).

**How voting works:** votes go to a **Google Form** (one multiple-choice
question = which venue). Each card's **“Vote for this”** button opens the Form
with the venue already selected — the voter signs in with Google and taps
**Submit**. **One vote per person**, changeable by editing their response.
Everyone sees an **anonymous** running tally embedded on the page (vote counts
per venue, no names, no emails). Host the page free on **GitHub Pages**.

---

## Privacy — read this (it's the whole point of the email setup)

You want: voters stay **anonymous to each other**, you **collect their emails**,
and **nothing leaks**. This build does that by keeping two things separate:

- **Public (what voters see):** an embedded chart of **only the venue column** —
  counts per venue. No emails, no names, no timestamps.
- **Private (what only you see):** the linked Google **Sheet / Responses tab**,
  which has each voter's email next to their pick.

The critical setting: **turn the Form's own “View results summary” OFF.** That
public summary page is the one place collected emails could be exposed, so we
don't use it at all — the page shows the venue-only chart instead. By Google's
design emails are owner-only, but their docs don't *guarantee* the summary
excludes them, so we remove the risk entirely rather than bet on it.

> **30-second safety check before you share the link:** submit one test vote,
> then open your **published chart link in an incognito window** (logged out).
> Confirm you see only venue counts — zero emails. If anything looks off, you've
> lost nothing, because the Form summary is already off.

---

## A. One-time setup (~15 min) — the organizer does this once

You'll create a Google Form, build a venue-only results chart, then paste
**three values** into `index.html`. Until you do, the page shows a tidy “not set
up yet” preview (buttons disabled), so it never looks broken.

### 1. Create the Form
1. Go to **https://forms.google.com** → **Blank form**. Title it e.g. *Trip venue vote 2027*.
2. Add **one** question, type **Multiple choice**. Question text: *Which venue?*
3. Add **one option per venue**, typed **exactly** as the card names below (same
   spelling, capitalization, spacing, punctuation — no extra/trailing spaces).
   Copy-paste so they match byte-for-byte:

   ```
   Laurel Island
   The Treehouse
   Lifebridge Sanctuary
   The Albedor
   McGee Island
   Happiness Is Camping
   Camp Lohikan
   Camp Mah-Kee-Nac
   The Retreat at Norwich Lake
   Clapboard Island Estate
   Tranquille Resort
   ```
   > If an option's text doesn't match the card's `name`, that card's button
   > opens the Form with **nothing pre-selected** (Google silently ignores a
   > non-matching value). Keep the two lists identical.

### 2. Settings — identity, one-vote-each, and NO public summary
Open the **Settings** tab and, under the **Responses** section, set:
- **Collect email addresses** → **Verified** (Google captures each voter's real
  account email; voters sign in with Google to submit).
- **Limit to 1 response** → **ON** (one real vote per person).
- **Allow response editing** → **ON** (so a voter can change their pick later).

Then, under the **Presentation** section:
- **View results summary** → **OFF**. ← *This is the privacy-critical one. Leave
  it off so no public page can ever show emails. The page uses the chart from
  step 4 instead.*

> Trade-off you accepted: “Verified” + “Limit to 1 response” require each voter
> to be signed into a Google account to vote. In return you get reliable voter
> emails and genuine one-vote-per-person. (If you'd rather keep voting
> login-free, switch **Collect email addresses → Responder input** and **Limit
> to 1 response → OFF**; voters then type an unverified email and can vote more
> than once. Tell me and I'll adjust the dedupe instructions.)

### 3. Get the prefill link + entry id  → fills values #1 and #2
1. In the Form **editor**, click the **⋮ menu** (top-right) → **“Get pre-filled link.”**
2. The form opens in preview. Pick **any one** venue as a sample → **“Get link”**
   (bottom) → **“COPY LINK.”**
3. From the copied URL (looks like
   `https://docs.google.com/forms/d/e/1FAIpQLSxxxx/viewform?usp=pp_url&entry.1234567890=Laurel%20Island`):
   - **Value #1 — `FORM_VIEWFORM_URL`**: everything **before** the `?`
     → `https://docs.google.com/forms/d/e/1FAIpQLSxxxx/viewform`
     ⚠️ Copy **only up to — not including — the `?`** (no `usp=pp_url`, no
     `=SampleVenue`). The sample venue doesn't matter; the page fills each card.
   - **Value #2 — `FORM_ENTRY_ID`**: just the **digits** in `entry.<digits>=`
     → `1234567890` (no `entry.` prefix, no spaces). One id covers all venues.

### 4. Build the anonymous results chart  → fills value #3
1. Form editor → **Responses** tab → **Link to Sheets** (green Sheets icon) to
   create/open the linked spreadsheet.
2. In the Sheet, find the column with the venue answers (e.g. column **B**). Add
   a small tally on a blank area or a new tab — for example in two columns:
   - Column “Venue”: paste the 11 venue names from step 1 (or use
     `=SORT(UNIQUE(B2:B))`).
   - Column “Votes”: next to each, `=COUNTIF(B:B, <that venue cell>)`.
   This tally references **only the venue column — never the email column.**
3. Select those two tally columns → **Insert → Chart** → pick a **column/bar**
   chart.
4. Click the chart → its **⋮ menu → “Publish chart”** → **Embed** tab →
   **Interactive** → **Publish** → copy the iframe snippet. Inside it is
   `src="https://docs.google.com/spreadsheets/d/e/2PACX-.../pubchart?oid=...&format=interactive"`.
   - **Value #3 — `FORM_CHART_EMBED_SRC`** = that **`src` URL** (just the URL
     inside the quotes, not the whole `<iframe>` tag).

### 5. Paste the three values into `index.html`
Near the top of the `<script>`, in the block marked
`PASTE YOUR GOOGLE FORM INFO HERE`:
```js
const FORM_VIEWFORM_URL  = "https://docs.google.com/forms/d/e/1FAIpQLSxxxx/viewform";          // #1
const FORM_ENTRY_ID      = "1234567890";                                                        // #2
const FORM_CHART_EMBED_SRC = "https://docs.google.com/spreadsheets/d/e/2PACX-.../pubchart?oid=123&format=interactive"; // #3
```

### 6. Test, then do the privacy check
- Double-click `index.html`: the banner is gone, buttons active, and the **Live
  results** chart shows under the cards.
- Click a **“Vote for this”** → the Form opens with that venue selected → sign in
  → Submit. **Two different Google accounts** can each vote and both show up.
- **Privacy check:** open `FORM_CHART_EMBED_SRC` in an **incognito window** and
  confirm it shows only venue counts — no emails.

---

## B. Deploy to GitHub Pages (free hosting)

1. Create a **public** GitHub repo (e.g. `trip-2027`).
2. Upload **`index.html`** (and this `README.md`) to the repo root; keep the file
   named exactly `index.html`.
3. Repo **Settings → Pages** → **Source: “Deploy from a branch”** → **Branch:
   `main`**, folder **`/ (root)`** → **Save**.
4. Wait ~1 minute, open the `https://<you>.github.io/trip-2027/` URL, share it.

> The repo is public (required for free Pages), but it contains **no secrets and
> no emails** — only the static page. Emails live solely in your private Sheet.

---

## C. Add or edit a venue

Venues live in `const V = [ ... ]` near the top of the `<script>`. Copy an
existing `{...}` block to add one; edit fields in place to change one.

- `name` — the title **and** the value sent to the Form. **Must exactly match a
  Form option (step 1) and the venue label in your Sheet tally (step 4).**
- `driveHrs`, `roof`, `priceSort` — drive the sorts/filters (see existing cards).
- Other fields (`where`, `dist`, `blurb`, `amen`, `cap`, `flags[]`, `verified`,
  `amt`, `amtq`, `note`, `kind`/`kindLabel`, `url`) are display-only.

**When you add/rename a venue, also update (a) the Form option and (b) the Sheet
tally** so all three stay identical. The card counter and “11 spots” label
update themselves from the data.

---

## D. Reset votes (start the tally over)

Form **editor → Responses tab → ⋮ menu → “Delete all responses.”** That zeroes
the linked Sheet, so the published chart drops to zero on its next refresh
(within a few minutes). Do this once before the real vote if you submitted test
responses. No edits to `index.html` needed.

---

## Troubleshooting

- **A card opens the Form with nothing selected** → that card's `name` doesn't
  match its Form option exactly (watch trailing spaces, curly vs straight
  apostrophes, capitalization).
- **Voters are asked to sign in** → expected: “Verified” email and “Limit to 1
  response” both require a Google sign-in. That's the trade for reliable emails +
  one-vote-each. Switch to “Responder input” + “Limit to 1 response” OFF if you
  want login-free voting instead.
- **Results chart is blank / not showing** → `FORM_CHART_EMBED_SRC` still has the
  `REPLACE_WITH…` placeholder, or you pasted the whole `<iframe>` tag instead of
  just the `src` URL. The chart also lags real votes by a few minutes (Google's
  publish-refresh cycle) — that's normal.
- **Vote buttons greyed out / banner showing** → `FORM_VIEWFORM_URL` still has a
  `REPLACE_WITH…` placeholder, or `FORM_ENTRY_ID` isn't digits-only.
- **I saw an email in the public chart** → you charted the email column by
  mistake. Rebuild the chart over the venue tally only (step 4), and make sure
  the Form's “View results summary” is OFF.

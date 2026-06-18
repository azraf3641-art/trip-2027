# The Trip · 2027 Shortlist — shared voting site

A single static page (`index.html`) that shows the venue shortlist and collects
votes. **No backend, no build step, no database, no install, no secrets** — and
nothing third-party runs in anyone's browser beyond the page itself.

**How voting works:** votes go to a **Google Form** (one multiple-choice
question = which venue). Each card's **“Vote for this”** button opens that Form
with the venue already selected — the voter just taps **Submit**. Everyone can
watch the running tally on the Form's own **live results page** via the
**“See live results”** button. Host the page free on **GitHub Pages**.

> **Why a Google Form instead of Supabase/Firebase/a server?** Zero
> dependencies, zero secrets in the page, and the smallest possible security
> surface. Google already stores, aggregates, and charts the responses for free.
> Trade-offs (accepted on purpose): the live tally is one tap away on Google's
> results page rather than embedded in the card grid (Google blocks embedding
> it), voting is “open the pre-filled form, then Submit” rather than one in-place
> tap, and — with login off — a friend *could* vote twice. All fine for ~30
> friends.

---

## A. One-time setup (~10 minutes) — the organizer does this once

You'll create a Google Form, then paste **three values** into `index.html`.
Until you do, the page shows a tidy “not set up yet” preview (buttons disabled),
so it never looks broken.

### 1. Create the Form
1. Go to **https://forms.google.com** → **Blank form**. Title it e.g. *Trip venue vote 2027*.
2. Add **one** question. Set its type to **Multiple choice**. Question text: *Which venue?*
3. Add **one option per venue**, typed **exactly** as the card names below
   (same spelling, capitalization, spacing, punctuation — no extra/trailing
   spaces). Copy-paste these so they match byte-for-byte:

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
   > If an option's text doesn't exactly match the card's `name`, that card's
   > button opens the Form with **nothing pre-selected** (Google silently
   > ignores a non-matching value — no error). Keep the two lists identical.

### 2. Keep it public + login-free
In the Form's **Settings** tab, open the **Responses** section and set:
- **Collect email addresses** → **Do not collect** (it's a dropdown in the
  current UI, not an on/off switch).
- **Limit to 1 response** → **off**.

(Both off = no Google sign-in needed to vote *or* to view results. Leaving
"Limit to 1 response" off is also what lets a friend change their pick by simply
voting again. Turning email collection on, or limiting to one response, forces
everyone to sign in — which breaks the no-login goal.)

### 3. Turn on shareable live results
Still in **Settings**, expand the **Presentation** section and turn **ON**
**“View results summary”** (some accounts label it *“See summary charts and text
responses”*). This lets anyone with the link see the live tally without logging
in. **If you skip this, the “See live results” button will prompt viewers for
access** even though voting itself still works.

### 4. Get the prefill link + entry id  → fills values #1 and #2
1. In the Form **editor**, click the **⋮ (three-dot) menu**, top-right →
   **“Get pre-filled link.”**
2. The form opens in preview. Pick **any one** venue as a sample answer →
   **“Get link”** (bottom) → **“COPY LINK.”**
3. The copied URL looks like:
   ```
   https://docs.google.com/forms/d/e/1FAIpQLSxxxxxxxx/viewform?usp=pp_url&entry.1234567890=Laurel%20Island
   ```
   Pull two pieces out of it:
   - **Value #1 — `FORM_VIEWFORM_URL`**: everything **before** the `?`
     → `https://docs.google.com/forms/d/e/1FAIpQLSxxxxxxxx/viewform`
     ⚠️ Copy **only up to — not including — the `?`**. Do **not** include
     `usp=pp_url` or the `=SampleVenue` at the end. (The sample venue you picked
     doesn't matter; the page fills each card automatically. If you paste the
     whole URL here, the vote buttons will pre-select nothing.)
   - **Value #2 — `FORM_ENTRY_ID`**: just the **digits** in `entry.<digits>=`
     → `1234567890` (no `entry.` prefix, no spaces). One id covers **all**
     venues.

### 5. Get the results link  → fills value #3
Look at your Form **editor** address bar — the URL ends in `/edit`. Copy it and
**replace `/edit` with `/viewanalytics`**:
```
editor:   https://docs.google.com/forms/d/1AbCdEf123.../edit
results:  https://docs.google.com/forms/d/1AbCdEf123.../viewanalytics
```
- **Value #3 — `FORM_RESULTS_URL`** = that `/viewanalytics` URL.

> ⚠️ **This is a different path than the vote link.** The results URL is
> `/forms/d/<id>/…` (**no “e”**) and uses a *different* id than the prefill URL's
> `/forms/d/e/<id>/…`. Don't try to build one from the other — copy each from its
> own place as described.

### 6. Paste the three values into `index.html`
Open `index.html`, find the block near the top of `<script>` marked
`PASTE YOUR GOOGLE FORM INFO HERE`, and replace:

```js
const FORM_VIEWFORM_URL = "https://docs.google.com/forms/d/e/1FAIpQLSxxxxxxxx/viewform"; // value #1
const FORM_ENTRY_ID     = "1234567890";                                                  // value #2
const FORM_RESULTS_URL  = "https://docs.google.com/forms/d/1AbCdEf123.../viewanalytics"; // value #3
```

### 7. Test locally
Double-click `index.html`. The “not set up yet” banner should be gone and the
buttons active. Click a **“Vote for this”** — the Form opens with that venue's
radio already selected; tap **Submit**. Click **“See live results”** — the charts
open with no login. **Two different browsers** can each vote and both show up in
the same results chart.

---

## B. Deploy to GitHub Pages (free hosting)

1. Create a **public** repo on GitHub (e.g. `trip-2027`).
2. Upload **`index.html`** (and this `README.md`) to the repo root. Keep the file
   named exactly `index.html`.
3. Repo **Settings → Pages** → **Source: “Deploy from a branch”** → **Branch:
   `main`**, folder **`/ (root)`** → **Save**.
4. Wait ~1 minute, then open the `https://<your-username>.github.io/trip-2027/`
   URL it shows. Share that link with everyone.

> The Google Form works the same from a `github.io` page and from a
> double-clicked file — it's just outbound links, no CORS to configure.

---

## C. Add or edit a venue

Venues live in the `const V = [ ... ]` array near the top of the `<script>` in
`index.html`. Each entry is one card. To **add** one, copy an existing `{...}`
block and edit its fields; to **edit**, change the fields in place.

Key fields:
- `name` — the venue title **and** the value sent to the Form. **It must match a
  Form option from step 1 exactly.**
- `order` — its number in “Original” sort (and the `NN / 11` label).
- `driveHrs` — number used by the “Closest to NYC” sort and the “Drive-able
  (≤3 hr)” filter (use a big number like `99` for fly-only).
- `roof` — `true` if everyone's under one roof (drives the “One-roof only” filter).
- `priceSort` — number used by the “Cheapest (verified)” sort (lower = first).
- `verified`, `amt`, `amtq`, `note`, `where`, `dist`, `blurb`, `amen`, `cap`,
  `flags[]`, `kind`/`kindLabel`, `url` — display fields (see existing cards).

**Whenever you add/rename a venue, also add/rename the matching option in the
Google Form (step 1)** so the names stay identical. (The `NN / 11` counter and
the “11 spots” label are cosmetic — update the `11` in `index.html` if you change
the count and want them exact.)

---

## D. Reset votes (start the tally over)

Open the Form **editor → Responses tab → ⋮ menu → “Delete all responses.”** That
zeroes the `/viewanalytics` charts. Do this once before the real vote if you
submitted test responses in step 7. Resetting does **not** change any of your
three pasted values — no edits to `index.html` needed afterward.

---

## Troubleshooting

- **A card opens the Form with nothing selected** → that card's `name` doesn't
  exactly match its Form option. Fix one to match the other (watch for trailing
  spaces, curly vs straight apostrophes, capitalization).
- **“See live results” asks for login** → in Form Settings, make sure *Collect
  email* and *Limit to 1 response* are OFF and *View results summary* is ON, and
  that the form isn't restricted to an organization.
- **Buttons are greyed out / banner shows** → one of the three values still
  contains a `REPLACE_WITH…` placeholder, or `FORM_ENTRY_ID` isn't digits-only.
  Finish step 6.
- **Want the live counts printed on the page itself?** Not reliably possible with
  zero backend — Google blocks embedding its results page (`X-Frame-Options`).
  The closest option is publishing a chart from the linked responses Sheet
  (`File → Share → Publish to web`), which refreshes on a delay (often several
  minutes), not instantly. The **“See live results”** link is always current and
  is the source of truth.

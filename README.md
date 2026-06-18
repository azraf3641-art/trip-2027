# The Trip · 2027 Shortlist — shared voting site

A single static page (`index.html`) that shows the venue shortlist and collects
votes. **No backend, no build step, no database, no install, no secrets, and no
third-party JavaScript running in the page.** (Google Fonts loads via `<link>`.)

**How voting works:** votes go to a **Google Form** (one **checkboxes** question =
which venues). On the page you tap **Add to my picks** on as many venues as you'd
be happy with, then **Submit my picks** opens the Form with them all pre-checked —
the voter signs in with Google and taps **Submit**. **One ballot per person**,
changeable by editing their response. An optional **“Count me in to help
organize”** toggle adds a volunteer flag to that ballot. **Results are private to
the organizer** — no public tally on the page; everything lands in your linked
Google Sheet. Host the page free on **GitHub Pages**.

---

## Privacy — everything is private to you (the organizer)

- The page shows **no results at all** — no tally, no chart, no “see results.”
- Each **vote**, voter **email**, and **volunteer** offer lands only in your
  linked **Google Sheet / Responses tab**.
- The Form's public **“View results summary” is OFF**, so there is no public page
  that could ever expose anyone's email, vote, or volunteer status.

---

## A. One-time setup (~15 min)

You'll create a Google Form, then paste up to **three values** into `index.html`.
Until you do, the page shows a tidy “not set up yet” preview (buttons disabled),
so it never looks broken.

### 1. Create the Form
1. Go to **https://forms.google.com** → **Blank form**. Title it *Trip venue vote 2027*.
2. **Question 1 — the vote.** Type **Checkboxes** (so voters can pick more than
   one venue), text *Which venues would you be happy with?*. Add one option per
   venue, typed **exactly** as the card names (copy-paste so they match
   byte-for-byte):
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
3. **Question 2 — volunteer (optional).** Type **Checkboxes**, text
   *Want to help organize the trip? (optional)*. Add **one** option, typed
   exactly:
   ```
   Yes, I'll help organize
   ```
   Leave this question **not required**. (This exact text must match
   `VOLUNTEER_VALUE` in `index.html`, which is already set to it.)

### 2. Settings (Settings tab)
Under the **Responses** section:
- **Collect email addresses** → **Verified**
- **Limit to 1 response** → **ON**
- **Allow response editing** → **ON**

Under the **Presentation** section:
- **View results summary** → **OFF**  ← keeps everything private.

> This means voters sign in with Google to vote (the trade for reliable emails +
> one-vote-each). If you'd rather keep voting login-free, set **Collect email →
> Responder input** and **Limit to 1 response → OFF**, and tell me — I'll adjust.

### 3. Get ONE prefilled link → all your values at once
1. Form editor → **⋮ menu** (top-right) → **“Get pre-filled link.”**
2. In the preview, **check two or three venues** AND **check the “Yes, I'll help
   organize” box**, then **“Get link”** → **“COPY LINK.”**
3. The copied URL looks like (ids are examples):
   ```
   https://docs.google.com/forms/d/e/1FAIpQLSxxxx/viewform?usp=pp_url&entry.1111111111=Laurel%20Island&entry.2222222222=Yes,+I%27ll+help+organize
   ```
   Pull out three things:
   - **`FORM_VIEWFORM_URL`** = everything **before** the `?`
     → `https://docs.google.com/forms/d/e/1FAIpQLSxxxx/viewform`
   - **`FORM_ENTRY_ID`** = the digits of the `entry.<digits>=` whose value is the
     **venue name** → `1111111111`
   - **`FORM_VOLUNTEER_ENTRY_ID`** = the digits of the `entry.<digits>=` whose
     value is **`Yes, I'll help organize`** → `2222222222`

### 4. Paste into `index.html`
Near the top of the `<script>`, in the `PASTE YOUR GOOGLE FORM INFO HERE` block:
```js
const FORM_VIEWFORM_URL       = "https://docs.google.com/forms/d/e/1FAIpQLSxxxx/viewform"; // #1, before the ?
const FORM_ENTRY_ID           = "1111111111";   // #2, venue question
const FORM_VOLUNTEER_ENTRY_ID = "2222222222";   // #3, volunteer question (optional — leave placeholder to hide it)
```
Leave `VOLUNTEER_VALUE` as `"Yes, I'll help organize"` (it must match the Form
option from step 1.3).

### 5. Test
- Double-click `index.html`: the banner is gone, the cards show **Add to my
  picks**, and the volunteer toggle shows at the top.
- Tap **Add to my picks** on a few venues → a **Submit my picks** bar appears at
  the bottom → (optionally flip the volunteer toggle) → click **Submit my picks** →
  the Form opens with those venues **pre-checked** (and the volunteer box if set)
  → sign in → Submit.
- Open your linked **Sheet**: the new row should show the **email**, the **venues**
  picked, and **“Yes, I'll help organize”** if checked.

---

## B. Deploy to GitHub Pages (free hosting)

1. Create a **public** GitHub repo (e.g. `trip-2027`).
2. Upload **`index.html`** (and this `README.md`) to the repo root; keep the file
   named exactly `index.html`.
3. Repo **Settings → Pages** → **Source: “Deploy from a branch”** → **Branch:
   `main`**, folder **`/ (root)`** → **Save**.
4. Wait ~1 minute, open the `https://<you>.github.io/trip-2027/` URL, share it.

> The repo is public (required for free Pages), but it holds **no secrets and no
> emails** — only the static page. Emails live solely in your private Sheet.

---

## C. Add or edit a venue

Venues live in `const V = [ ... ]` near the top of the `<script>`. Copy an
existing `{...}` block to add one; edit fields in place to change one.

- `name` — the title **and** the value sent to the Form. **Must exactly match a
  Form option (step 1.2).**
- `driveHrs`, `roof`, `priceSort` — drive the sorts/filters (see existing cards).
- Other fields (`where`, `dist`, `blurb`, `amen`, `cap`, `flags[]`, `verified`,
  `amt`, `amtq`, `note`, `kind`/`kindLabel`, `url`) are display-only.

When you add/rename a venue, also update its **Form option** so they stay
identical. The card counter and “11 spots” label update themselves.

---

## D. Reset votes (start over)

Form **editor → Responses tab → ⋮ menu → “Delete all responses.”** Do this once
before the real vote if you submitted test responses. No edits to `index.html`
needed.

---

## Troubleshooting

- **A card opens the Form with nothing selected** → that card's `name` doesn't
  match its Form option exactly (watch trailing spaces, curly vs straight
  apostrophes, capitalization).
- **Volunteer toggle doesn't appear** → `FORM_VOLUNTEER_ENTRY_ID` is still the
  `REPLACE_WITH…` placeholder or isn't digits-only. (Voting still works without
  it.)
- **Volunteer box isn't pre-checked** → the Form option text isn't exactly
  `Yes, I'll help organize` (must equal `VOLUNTEER_VALUE`).
- **Voters are asked to sign in** → expected: “Verified” email + “Limit to 1
  response” require a Google sign-in. Switch to “Responder input” + Limit-to-1
  OFF for login-free voting.
- **Vote buttons greyed out / banner showing** → `FORM_VIEWFORM_URL` still has a
  `REPLACE_WITH…` placeholder, or `FORM_ENTRY_ID` isn't digits-only.

# Global Rosary for Peace — Live Dashboard

Live sign-up dashboard for the Global Rosary for Peace, 22 October 2026.
Reads registrations from Supabase and refreshes every 30 seconds.

**Live at:** `https://hcfm.github.io/global-rosary/`

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole dashboard — markup, styles, logic |
| `rosary-config.js` | Holds the Supabase publishable key. **You create this.** |

---

## Setup — one file to add

The dashboard shows *"No key found"* until `rosary-config.js` exists beside
`index.html`. Create it with a single line:

```js
window.SUPABASE_KEY = "sb_publishable_...";
```

Get the key from **Supabase → Project Settings → API Keys → Publishable key**.
It is the same key already saved locally at `~/.hcfm/supabase.env`.

### Why it's safe to commit this

The **publishable** key is designed to sit in browser code. Row Level Security
on `public_registrations` restricts what it can read — verified 31 July 2026,
the view returns no `email`, no `phone_number`, no `ref`. HCFM already publishes
an equivalent key in the live registration app at `family-rosary-zeta.vercel.app`,
so this adds no new exposure.

**Never commit the `service_role` or `secret` key.** Those bypass RLS entirely,
can read every email and phone number, and can write and delete rows.

---

## Deploying to GitHub Pages

`hcfm.github.io` already serves Pages from `main` and already has `.nojekyll`
at the root, so a new folder just works:

1. Put `index.html` and `rosary-config.js` in a `global-rosary/` folder
2. Commit to `main`
3. Live at `https://hcfm.github.io/global-rosary/` within about a minute

Nothing at the site root is touched — the existing `index.html`, `colum.html`,
`victoria.html` and `system-map.html` are unaffected.

**Note:** `hcfm.github.io` is a public repo and Pages is public. Anyone with
the URL can view the dashboard. That is acceptable here because RLS exposes no
personal data — but it is a deliberate choice, not an accident.

---

## Data

- **Source:** Supabase project `fuijdbeifajbmkchldiz`, view `public_registrations`
- **Refresh:** every 30s; pauses when the tab is hidden; keeps the last good
  figures if a request fails
- **Counting rule:** each registration counts `max(group_count, 1)` people.
  `group_count` is the total *including* the registrant, so a blank or `0`
  still counts as one person. This matches `WP1-counting-rule-memo.md`.
- **Pagination:** PostgREST caps a response at 1000 rows. The dashboard pages
  through with the `Range` header, so the headline stays correct past 1000
  registrations rather than silently flattening.

### Temporary test-data filter

`index.html` contains:

```js
const EXCLUDE_TESTS = true;
const TEST_IDS = [ ...31 row-id prefixes... ];
```

31 of the 38 rows in the database are staff and developer test registrations
that were never cleared. Left in, they show 4,238 people and put Pakistan —
where HCFM does not operate — at the top of the country table.

This is a **blocklist, not an allowlist**. Anything not named in `TEST_IDS`
counts immediately, so a genuine new sign-up never needs a code change to
appear. (An earlier draft used an allowlist of known-real names; it would have
silently dropped the registration that arrived on 31 July.)

**Delete the block and set `EXCLUDE_TESTS = false`** once the developer adds an
`is_test` column and filters it out of the `public_registrations` view. That is
the proper fix; this is the stopgap.

---

## Brand

Fonts load from hcfm.org, which serves them with `access-control-allow-origin: *`:

- **Calluna** — display headings and all numerals
- **Whitney** — labels, small caps, UI text
- **PlaylistScript** — the script accent

Colours: `#FFB500` gold and `#89764B` muted gold, both confirmed against the
canonical tokens in `hcfm/hcfm-brand`. Black on white below the dark band.

**One known conflict.** `hcfm/hcfm-brand` specifies Whitney as the display face
(`--font-display: 'Whitney'`, h1 at weight 700). The live hcfm.org theme does
the opposite — "Global Rosary for Peace" and "Register" on `/pray-for-peace/`
render in **Calluna 700**, with Whitney reserved for 20px utility headings.
This dashboard follows the live site, so a visitor coming from the registration
page sees continuity. Worth resolving with Victoria, since one of the two is
wrong.

---

## Known limitations

- **No per-school or per-centre breakdown.** The registrations table has no
  `ref` or `utm_*` column, so sign-ups cannot be attributed to the 157 coded
  links in `WP2-link-manifest.csv`. Country is the only available cut, and
  `country_code` is null on every row — country is parsed from the tail of
  `place_formatted`.
- **Aggregation happens in the browser.** Fine to a few thousand rows. Beyond
  roughly 10,000, move it into a Postgres view returning pre-aggregated counts.
- **Polling, not streaming.** Supabase Realtime would give instant updates but
  the project is on the Free plan; 30-second polling is the safer choice until
  that changes.
- **One number, not two.** `WP1-counting-rule-memo.md` calls for institutional
  pledges and individual registrations to be kept as separate series. The
  dashboard currently shows a single blended figure, and Def. v1 was never
  signed off.

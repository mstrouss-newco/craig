# craig

craig is a board management tool for a nonprofit / foundation board. There is a public landing page, plus two parts that share one database:

1. **index.html** is the landing page (the public front door that explains what craig does and links to the two apps below). This is what GitHub Pages serves at the root.
2. **craig.html** is the staff dashboard (the back office that the organization runs internally).
3. **approve.html** is the board member portal (the page that goes out to directors by a link).

Both are plain HTML files. There is no build step, no framework, and no server to run. You open the file and it works.

---

## What it does

**Dashboard (craig.html), for staff:**
- Dashboard: a Key dates strip up top (next board meeting, next marketing meeting, next event, and so on, one card per kind), plus the next meeting, pending votes, pending signatures, latest updates, and board at a glance.
- Meetings: schedule meetings (each tagged with a kind: Board, Marketing, Committee, Fundraiser, or Event, which is what feeds the Key dates strip), mark attendance, and see a quorum bar.
- Documents: share decks, minutes, policies (by link), optionally tied to a meeting.
- Updates: post notes to the board between meetings.
- Grants: submit grants, collect votes, and a per director "who has responded" tracker. A grant auto approves once enough directors approve.
- Signatures: collect sign offs on resolutions and consents (a record of who signed and when).
- Governance: the board roster, roles, terms, status, and the organization name.

**Portal (approve.html), for board members:**
- A board member opens their link, identifies by email (or by a code in the link), and sees everything waiting on them: RSVP to the next meeting, approve or decline grants, sign consents, open shared documents, and read updates.
- Whatever a board member does here shows up in the dashboard, because both read and write the same database.

---

## Architecture (read this before changing anything)

- **Single file HTML.** Each file is fully self contained: HTML, CSS, and JavaScript all in one file. No external project files.
- **Vanilla JavaScript.** No React, no jQuery, no Supabase SDK. Data access is direct `fetch()` calls to the Supabase REST API.
- **No build step.** There is nothing to compile or bundle. Editing the file is the whole workflow.
- **Hosting: GitHub Pages.** The repo is served as a static site at https://mstrouss-newco.github.io/craig/ . Pushing to the repo updates the live site. index.html is the landing page that opens at the root, and it links on to craig.html and approve.html. When a feature is added or changed in a meaningful way, update index.html too so the landing page keeps describing what craig actually does.
- **Backend: Supabase**, which is a hosted Postgres database with an automatic REST API (PostgREST).

Why it is built this way: the founder (Mike) is non technical and wants files he can host anywhere with zero tooling. Keep this approach unless there is a strong reason to change it and Mike agrees. Do not introduce a framework or a build pipeline without checking first.

### Repo layout

```
craig/
├── craig.html        the staff dashboard
├── approve.html      the board member portal
├── craig-setup.sql   the full database schema (run in Supabase to rebuild)
├── README.md         this file
├── CLAUDE.md         operating rules for agents (Claude Code reads this automatically)
└── WORKLOG.md        running log, newest entries at the top
```

---

## Supabase

- **Project URL:** `https://nhfymtungjcspgxvaexd.supabase.co`
- **Public (publishable) key:** `sb_publishable_CY4d7M7zl4AjcLGVltF5LQ_SPQRuYNS`
  - This is browser safe and belongs in the client code. It is already public the moment the site is live, so do not treat it as a secret.
  - A legacy anon key in JWT format (starts with `eyJ`) also exists in the Supabase dashboard under Settings, API. It is a fallback if the publishable key ever misbehaves with the REST API. Swapping the `SB_KEY` constant in both HTML files is the only change needed.
- The app only uses the publishable key above. The database password and the `service_role` key are not used by the app, so they do not need to go anywhere near the code.
- **Schema:** lives in `craig-setup.sql`. Running that file in the Supabase SQL editor rebuilds every table and reloads the sample board.

### Tables

`settings`, `members` (each has an `access_token` for personal links), `meetings`, `attendance`, `documents`, `updates`, `grants`, `grant_votes`, `consents`, `signoffs`.

The three "who did what" tables (`attendance`, `grant_votes`, `signoffs`) use composite primary keys, one row per person per item, so two people acting at the same time never overwrite each other.

### How data flows in the code

- On load, `loadAll()` fetches every table in parallel, then assembles the rows into one in memory `db` object with friendly camelCase names (for example `meeting_date` becomes `date`, `grant_votes` get folded into each grant as a `votes` map).
- The screen renders from that `db` object.
- Every change calls `act(async () => { ...write to Supabase... })`, which writes, reloads, and re renders. The helper shows a "Working" pill and a red error toast if a call fails.
- The REST helpers are `sbGet`, `sbInsert`, `sbUpdate`, `sbDelete`, `sbUpsert`. Upserts pass `on_conflict` for the composite key tables.

If you change the database shape, change it in `craig-setup.sql` and in BOTH html files, because they share the same loader.

---

## Business rules (keep these consistent across both files)

- **Quorum** is a majority of active directors: `floor(active / 2) + 1`.
- A **grant auto approves** the moment its approve votes reach quorum.
- A **consent is fully executed** when sign offs reach quorum, or reach all active directors if it was marked "unanimous" (`require_all`).
- **Dates** stored as plain dates (YYYY-MM-DD) are parsed as local time (`parseLocal`) so they do not slip a day in the user's timezone.

---

## Design system (keep the look consistent)

- Colors: warm paper background `#EAE7DE`, card `#FBFAF6`, ink `#23211C`, pine green `#2F5D4F` (primary and "approved"), ochre `#BC8439` (pending), clay `#A8503C` (declined / negative).
- Fonts: Spectral (serif) for headings and the craig wordmark, Public Sans for body, both from Google Fonts. Public Sans is a civic typeface (it was made for US government public sites), which fits a nonprofit that serves the public. Spectral gives the headings a warm, editorial feel. This pairing replaced the earlier Fraunces and Hanken Grotesk to give craig its own look.
- Tone: warm, calm, plain spoken. The product is named after a person on purpose. It should feel like a trustworthy chief of staff, not enterprise software.

---

## Founder preferences (apply these everywhere)

- **Plain language.** Mike is non technical. Write to him, and write user facing copy, without jargon.
- **No em dashes. Ever.** Not in code comments, not in UI copy, not in these docs. Use commas, parentheses, or colons instead. This is a firm rule.
- **Warm, human design** as described above. Do not drift toward generic SaaS styling.
- Mike is fine handing agents a lot of autonomy. See CLAUDE.md for what that means in practice.

---

## Where things stand

The app works end to end against the live Supabase project, with the sample board loaded. Things that could come next, in no particular order:

- **Personal links for board members.** Every member has an `access_token` and the portal already accepts `approve.html?t=TOKEN`. The dashboard could generate and copy each director's personal link so they never type an email.
- **Sending email** for invites, links, and meeting reminders (would need an email service, for example a Supabase edge function plus Resend).
- **Real file uploads** for documents. Today documents are links only. Supabase Storage could hold actual files.
- **Legally binding e signatures.** Today signing is an internal record (name plus timestamp). A DocuSign style integration is the upgrade path.
- **Database access rules.** Row level security is set to open while building. If craig ever goes live to a real board, tightening it is worth doing. Not urgent for a side project.

---

## Future ideas (no commitment, just the bucket)

Committees, automatic meeting minutes, board packet / PDF export, Google Calendar sync (Mike already uses Google Calendar elsewhere), notifications, an audit log, finer permissions per role, term and election tracking, conflict of interest disclosures, multi organization support (today it is a single org in the `settings` row).

---

## Working on this repo

- Quick sanity check on the JavaScript without a browser: extract the script and run `node --check`. The whole UI is in the one inline `<script>` per file.
- Real test: open `craig.html` in a browser. If it cannot reach Supabase it shows a clear error screen with the exact message. Make a change, refresh, and confirm it persisted to know the database connection is healthy.
- Deploy: push to the repo. GitHub Pages serves it. There is no other deploy step.

See **CLAUDE.md** for how much latitude you have, and **WORKLOG.md** for the history and what to do at the end of your session.

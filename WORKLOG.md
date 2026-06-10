# craig work log

This is the shared memory across sessions. **Newest entries at the top.** Every agent adds an entry before finishing. Keep it honest, including dead ends, so the next agent does not repeat them.

## Entry template (copy this for a new entry)

```
## YYYY-MM-DD, short title (who)
Context: what you set out to do.
Changed: files touched and what changed.
Fixed: bugs resolved.
Tried but did not work: anything worth warning the next agent about.
Still open: what is unfinished or needs Mike.
Ideas: anything for later.
```

---

## 2026-06-09, landing page redesign (Claude, web session)
Context: Mike wanted a stronger narrative aimed at nonprofits and executive directors who manage a board and grants, with no talk of how the tool is built, plus a complete visual redesign with a modern Fortune 500 or slick startup feel (the old serif look read as generic).
Changed:
- index.html: full rebuild. New dark, premium look using Inter for text and Space Grotesk for headings, a gradient brand accent (blue to purple to teal), a glassy sticky nav, a centered hero with a gradient headline and a stat row, a six card feature grid (Meetings, Grants, Signatures, Documents, Updates, Governance), two role panels (command center for the executive director, director portal for board members), a gradient call to action band, and a footer.
- - Narrative now leads with running the board and grants without the chaos, and positions craig as the operating system for executive directors. Removed the entire how it is built section and any mention of single file HTML, REST, frameworks, or GitHub Pages.
  - Fixed: nothing functional. Buttons still point at craig.html (dashboard) and approve.html (portal). The font swap from Spectral and Public Sans only affected this landing page; the dashboard and portal still use their own design system.
  - Tried but did not work: pasted the full file into the GitHub editor via clipboard, which worked once the editor was focused by clicking a code line first. First pass had a double marker on the role panel list items (a leftover li:before square next to the green check); removed the li:before rule and re-pasted to fix it.
  - Still open: nothing for this. The kind column still needs adding in Supabase for non-Board Key dates to populate, carried over from earlier.
  - Ideas: a short testimonial or logo strip, and a small product screenshot in the hero, would push it further toward a polished SaaS landing page if Mike wants.

## 2026-06-09, fix off-center dashboard layout (Claude, web session)
Context: Mike reported the dashboard looked way off center, with a big empty gap between the sidebar and the main content.
Changed:
- craig.html: added a base CSS rule (.mnav{display:none;}) right after the .mobile-bar base rule. The mobile nav strip (.mnav) only had a display rule inside the @media (max-width: 860px) block, so on desktop it defaulted to display:block and sat in the flex row between the sidebar and main, pushing everything to the right by about 540px.
- Fixed: the off-center layout. Main content now sits flush next to the sidebar and the Key dates cards fill the width evenly.
- Tried but did not work: GitHub's editor find/replace shortcut (cmd+alt+f) opened the global site search instead, and earlier typing leaked into that search box. Clicking directly on the target code line, pressing End, then typing the rule worked cleanly.
- Still open: nothing for this fix. The kind column still needs adding in Supabase (alter table meetings add column kind text default 'Board';) for non-Board kinds to save, carried over from the previous entry.
- Ideas: consider a quick visual check at desktop and mobile widths after any layout change, since the mobile and desktop nav share the same flex row.

## 2026-06-09, key dates on the dashboard, meeting kinds (Claude, web session)
Context: Mike asked for a key dates section at the top of the dashboard: next board meeting, next marketing meeting, next event, and so on.
Changed:
- craig.html: added a Key dates strip at the very top of the dashboard. It shows the next upcoming date for each meeting kind (Board, Marketing, Committee, Fundraiser, Event), each as a small card with the date, the meeting title, and how far away it is, or "Nothing scheduled" if that kind has none coming up. Added a MEETING_KINDS list, upcomingMeetings() and nextOfKind() helpers, and a keyDatesHtml() builder. Meetings now carry a kind: the schedule-meeting form has a Kind selector (defaults to Board), saveMeeting stores it, and each meeting row shows a kind pill.
- approve.html: the portal now reads a meeting's kind too and labels the next-meeting block (for example "Next marketing meeting") instead of always "Next meeting".
- Existing meetings with no kind default to Board everywhere, so nothing breaks before the database column exists.
Fixed: during this work an earlier editor mishap committed a craig.html with a duplicated helper block and a missing keyDatesHtml (it threw "Illegal return statement" and the dashboard went blank). Rebuilt craig.html cleanly from the last good commit, re-applied every edit, syntax-checked it, and recommitted. The live dashboard renders the Key dates section correctly now.
Still open (needs Mike): the meetings table in Supabase does not yet have a kind column, so picking a kind other than Board when scheduling will fail to save until it is added. One line in the Supabase SQL editor fixes it: alter table meetings add column kind text default 'Board'; After that, marketing, committee, fundraiser, and event meetings will save and show up in their Key dates cards. Reading and the dashboard already work without it.
Tried but worth knowing: pasting a large file into GitHub's web editor can silently fail to register as a change (commit button stays disabled) unless you first click directly on the code text to focus the editor. Also the web editor's Restore can bring back a stale draft, which is what caused the broken commit above. Verify on the live Pages URL with a cache-busting query (for example ?v=2) since the old file can be cached.
Ideas: let staff add a freeform event kind, show the next few dates per kind rather than just the next one, and a tiny calendar view.

## 2026-06-09, go live on GitHub Pages, new fonts, landing page (Claude, web session)
Context: get craig live on GitHub Pages for testing, give it a landing page, and move the type away from the common Fraunces and Hanken Grotesk pairing.
Changed:
- index.html: new file. A public landing page that explains what craig does (the seven jobs, the staff dashboard vs board portal split, and how it is built) and links on to craig.html and approve.html. Styled with the existing design tokens (paper, pine, ochre, clay) so it matches the apps. This is now the front door GitHub Pages serves at the root.
- craig.html and approve.html: swapped the fonts. Headings and the wordmark are now Spectral (was Fraunces), body is now Public Sans (was Hanken Grotesk). Changed the Google Fonts link and every font-family in both files. The italic "signed" style in craig.html is covered by Spectral italic, which the font link now loads.
- README.md: documented the landing page in the intro, the repo layout, and the architecture notes (including the live URL). Updated the Design system fonts line to Spectral plus Public Sans, with a note on why (Public Sans is a civic typeface, fitting for a nonprofit). Added a standing rule: when a feature is added or changed in a meaningful way, update index.html too so the landing page keeps describing what craig actually does.
- Repo setting: enabled GitHub Pages, deploying from main, root folder. Live at https://mstrouss-newco.github.io/craig/ .
Fixed: nothing broken before, this was additive.
Tried but did not work: nothing notable. Note for the next agent: GitHub's web editor does not run pasted HTML in Preview, so verify rendering on the live Pages URL, not in the editor preview.
Still open: same list as the entry below (personal links in the dashboard, email sending, real file uploads, binding e signatures, tightening database access rules). The font choice is opinionated, easy to revisit with Mike if he wants a different feel.
Ideas: a small "what changed" note on the landing page, and a favicon, would both be nice touches later.

## 2026-06-09, initial build and Supabase wiring (Claude, web app session, pre Claude Code)

**Context.** Built craig from scratch with Mike over one working session, then wired it to a live Supabase backend. This entry is the starting point for the repo.

**What exists now.**
- `craig.html`: staff dashboard with seven sections (Dashboard, Meetings, Documents, Updates, Grants, Signatures, Governance). Fully wired to Supabase REST.
- `approve.html`: board member portal, full board facing. RSVP to the next meeting, approve / decline / abstain on grants, sign consents, view documents, read updates. Identifies a member by `?t=TOKEN` in the URL, or by email as a fallback. Fully wired to Supabase REST.
- `craig-setup.sql`: the full schema plus a sample board (Maple Grove Foundation, 7 directors, sample meetings, documents, grants, a consent). Idempotent enough to run on a fresh project.

**What was done, in order.**
1. Built the dashboard first as a single file HTML app using browser storage, so Mike could see and feel it immediately.
2. Added the board member portal as a second page.
3. Upgraded Grants into a real tracker: a per director "who has responded" list (Approved / Declined / Abstained / Awaiting) and a progress bar to quorum. Grants now auto approve when approve votes reach quorum, instead of needing a manual admin click.
4. Created a brand new Supabase project just for craig (Mike did this himself, West US Oregon, paid plan). Ran `craig-setup.sql`. Confirmed success.
5. Rewrote both files to drop browser storage and read / write the Supabase REST API directly with `fetch` (no SDK, to keep the no build approach). Added `loadAll`, the `act` helper, a "Working" indicator, error toasts, and a friendly fatal error screen if the connection fails.
6. Expanded the portal from grants only to everything board facing.

**Design and conventions established.** See README for the full design system and the founder preferences. The big two: warm human aesthetic (Fraunces + Hanken Grotesk, paper / pine / ochre / clay), and no em dashes anywhere.

**Tried but worth knowing.**
- The newer publishable key (`sb_publishable_...`) is used in the code. Mike also has the legacy anon JWT key (`eyJ...`) from the dashboard. If the publishable key ever fails against the REST API, swap the `SB_KEY` constant in both files to the JWT one. This was not tested against the live project from the build environment (no network access to Supabase there), so the very first browser load is the real first test.
- The HTML was syntax checked with `node --check` on the extracted script, not run against the live database.

**Still open (no particular order).**
- Personal links: the portal already accepts `?t=TOKEN`. Build the dashboard side so staff can generate and copy each director's personal link.
- Email sending for invites, links, and reminders (needs an email service).
- Real file uploads for documents (link only today). Supabase Storage is the likely path.
- Legally binding e signatures (internal record only today).
- Database access rules are open while building. Worth tightening if it ever goes live to a real board, not urgent for a side project.

**Ideas for later.** Committees, auto generated minutes, board packet PDF export, Google Calendar sync, notifications, audit log, per role permissions, term and election tracking, conflict of interest disclosures, multi org support.

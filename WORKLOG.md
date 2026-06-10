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

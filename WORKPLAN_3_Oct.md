# Frictionless work plan, Sat 3 Oct 2026 (from 11:20 CEST)

Chris's asks (11:19): every earlier goal finished; e2e-test everything, no mistakes; the app updates itself on every conversation; a common calendar (pick a day, week or month; see everything on a day); "I'm clicking on the buttons, nothing works".

## What the diagnosis found (11:20–11:40, Claude, verified)
- Chris's default browser is Chrome 154 (LaunchServices). At 11:24 his Chrome showed `localhost:8787/files/outbox/PASTE_PACK_Mon_5_Oct.html#item-5` (the paste pack opened from a task's "Pack item" link), then the app at `#decisions`.
- The server log has no POST from Chris today. Nav and links work (favicon and /files GETs).
- The pack is served from /files with a CSP `sandbox` (opaque origin, `self.origin == "null"`). Clipboard-write permission there is "denied" in the Claude pane and "granted" in headless Chrome. Copy fell back to `execCommand` and worked in both test browsers. The behaviour in Chris's own Chrome profile is not reproduced, so Copy on the pack is treated as **unreliable**.
- Mirror (claude.ai): renders "0 / nothing due" for several seconds before data arrives (no loading state). File links show as plain text there. Mirror redaction replaced the bulletin PDF path in A02 `docs[0].f`, which breaks that link.

## Contract: Calendar section (frontend agent and e2e agent both build to this)
- New section id `calendar`, label "Calendar", short "Cal", key `l` (g then l). It goes second, after Today. It replaces the week and month calendar inside Meetings: Meetings keeps its list and links to the calendar.
- State: `calView` ('day' | 'week' | 'month', default 'week'; 'day' below 760px), `calAnchor` (YYYY-MM-DD, default today), `calKinds` (all on).
- Controls: three buttons with `data-cal-view="day|week|month"` and aria-pressed. Prev `data-cal-prev`, Next `data-cal-next`, Today `data-cal-today`, and an `<input type="date" data-cal-date>` bound to calAnchor. Kind toggles carry `data-cal-kind="<kind>"` and aria-pressed.
- Item kinds, each a jump link `a[data-go][data-id]` with `data-cal-item` and `data-kind`:
  - `meeting`: start, time and duration.
  - `send`: message scheduled, draft or held, with when.at.
  - `sent`: message with sent_at.
  - `task`: open task with a due date; overdue tasks stay on their due date.
  - `done`: task done_on.
  - `date`: hard date; a range shows on every day it covers in Day and Week views.
  - `decision`: decision date.
  - All times are local.
- Day view (`[data-cal-dayview]`): heading with the full date; "All day" items (date, task, done, decision) first, then timed items (meeting, send, sent) by time. Empty day: "Nothing on <date>." with the next item after it.
- Week view: Monday to Sunday, every item, and each day header is a button `[data-cal-day="YYYY-MM-DD"]` that opens the Day view.
- Month view: a grid of whole weeks. Each cell's date button `[data-cal-day]` opens the Day view. Each cell shows up to 3 items plus a "+N more" button (`[data-cal-more]`) that also opens the Day view. Below 760px it shows a count chip only.
- Deep links: `#calendar` opens it; `#calendar-2026-10-07` opens the Day view on that date. Today panel: "Open today in the calendar" links to `#calendar-<today>`.
- Works the same in Local and Mirror mode, and in both themes.

## Contract: client error reporting
- The app sends `POST /api/client-error` with JSON `{msg, src, line, col, stack, url, ua}` (strings, each ≤2,000 chars, body ≤16 KB) on `window.error` and `unhandledrejection`, in Local mode only.
- The server checks Origin and JSON, appends one JSON line to `<home>/logs/client-errors.jsonl` (created 0600, capped at 1 MB, rotate to `.1`), and answers 204.
- The page also shows a dismissible banner, "Something went wrong: <msg>. It's logged.", so a failure is never silent.

## Contract: auto-update on every conversation (Claude Code hooks, this project)
- `UserPromptSubmit` hook: runs `fold` (Chris's app clicks into the data) and adds short context: what changed, plus the routine reminder (record new facts from this message with sources; publish; sync the mirror).
- `Stop` hook: when `frictionless/data` changed since the last publish, runs `publish`. When the mirror export differs from what was last synced, blocks the stop once with "Mirror out of date: sync it" (respects `stop_hook_active`, so it can't loop).
- Mirror sync tool (`frictionless.py mirror-plan`): computes the diff between `mirror/*.json` and a local ledger of what was last written (doc_id → content hash and version), and writes ready-to-send ArtifactData batch files (≤50 writes, ≤1 MiB, `if_version` from the ledger). `mirror-mark-synced` updates the ledger from a batch result. Facts:
  - ArtifactData `out_dir` files hold only the document data, not its version.
  - Versions appear in the listing text.
  - Every write to an existing document needs `if_version`.
  - A batch is all-or-nothing.

## Contract: spanning-bar calendar (Chris's reference screenshot, 3 Oct ~13:20)
Chris: "I want the calendar to have a similar concept."
- **The concept, from his booking calendar:**
  - continuous colour bars across days, lane-packed per week row;
  - a status badge on each bar and a tooltip on hover;
  - today as a filled circle;
  - a colour legend under the grid with an explanatory line;
  - a big title on the left, with Month/Week plus Prev/Today/Next on the right.
- **Defaults Claude chose (Chris can change them):**
  - Weeks start Monday (Poland; sprints run Tue–Mon).
  - Only hard-date ranges span days: sprints, leave and release windows. Every other kind is a one-day bar.
  - Month shows at most 4 lanes per row (2 on a phone), with "+N more" opening the day.
  - Day view and the unscheduled tray are unchanged.
- **Attributes:** the data-cal-* attributes stay. Bars add `data-cal-start`/`data-cal-end` (the clipped days in that row) and `data-span-start`/`data-span-end` (the item's full range); `cont-l`/`cont-r` mark a bar that continues across a row.
- **Amendments after Claude's screenshot review (14:05):**
  - Month hides the badge on one-day bars, as the reference does, so titles stay readable; the status stays in the tooltip, Week and Day.
  - `meeting-series` shows as "series".
  - Late and overdue items are packed first in each row, so they stay visible. A "+N more" that still hides one is shown in red.

## Checklist (status in the run log below)
1. Calendar section per contract, old Meetings calendar removed, Today link.
2. Mirror loading state: no "0 / nothing due" before data arrives.
3. "Pack item N" links open the in-app message card (Copy + Mark sent live there) in both modes; the pack page stays one click away.
4. Pack Copy: synchronous `execCommand` copy inside the click first, then the Clipboard API; if both fail, select the text and say "Press ⌘C". Re-render the pack.
5. Client error reporting (server + app).
6. Hooks + mirror-plan/mark-synced + the mirror path-redaction fix (internal-only links dropped, not rewritten).
7. E2E suite (puppeteer-core + real Chrome, isolated server copy) covering every control, both widths, both themes, deep links, every link target.
8. Goals audit: every earlier ask and every Claude-doable task progressed or done; S01 DM draft for Erik; link audit across the data.
9. Independent code + security review; e2e green; publish; mirror re-sync; CLAUDE.md + memory updated.

## Run log
- 11:40 Plan written. Launching the build workflow.
- ~12:20 Build done (6 agents). validate ok; Python 16/16; server 24/24; e2e 94/94 in real Chrome 154. Reverted an agent-added task E14 (AI-policy question on the Erik 1:1 agenda): Chris's rule is not to raise it.
- ~12:30 Independent review (11 agents, adversarially verified):
  - Critical F1: M28's bulletin sentence (WebAuthn (13) / passkeys (85)) passed the sentence-level redaction and was live on the claude.ai mirror since 03:45. Contained at ~12:35 with an ArtifactData str_replace on M28.body (now version 2).
  - Scanned the 03:42 export for bulletin vocabulary: no other specifics live.
  - High: F2 (internal_only not fail-closed), D1 (10 new people notes carry the edit's scope), D6 (S01 credential topic reaches the mirror indirectly), plus deployment steps.
- ~12:40 Fix workflow launched (Python leak-safety and ledger, frontend calendar additions and a11y, server log escaping, data rule fixes, e2e on current data + mirror stub).
- ~12:50 Fixes integrated: validate ok; Python 26/26; server 25/25; e2e 116/116; leak check 0. Claude fixed pack item 28's confirm-first condition, O08's wording and Erik's stale O01 owes line.
- 12:54 `publish --server` + LaunchAgent restart. Smoke test: /api/client-error 204.
- ~12:58 Gap found: open local pages never picked up new publishes. Added `GET /api/version` (server tests RED then GREEN, 26/26) and app polling (30 s, plus on becoming visible). New data loads in place; a new build reloads an idle page, or shows a Reload banner while a form is open. The e2e tests fail against the 12:54 app (RED) and pass now: 119/119.
- 13:00 `publish --server` + restart. The app fingerprint matches the repo (0972be18ce58). The calendar is checked visually in real Chrome (Month, October: 35 cells, 139 items, tray 31).
- ~13:05 Mirror:
  - Full real listing pulled: 672 docs, all v1 except M28 v2; no `_edited_via` mirror edits.
  - `mirror-ledger-init`, then plan (23 new, 54 changed, 3 deletes: the credential-topic Confluence docs).
  - Outgoing docs leak-scanned (0 hits); 2 pinned batches committed; marked synced; re-plan finds nothing.
  - Artifact republished as v3. Checked live: loading state, data, Calendar via keyboard.
  - Open: pointer clicks didn't register in the Claude pane on v3 (keys did; clicks worked on v2 at 11:21). Cause not confirmed: pane tool vs claude.ai. Chris to tap-test.
- 13:20–14:10 Spanning-bar calendar (Chris's reference):
  - Built by frontend + e2e agents. Integration found nothing to change.
  - Independent design/a11y review: 1 high (sideways scroll at 760–990px) and 5 medium/low findings. All fixed with tests; e2e 137/137.
  - Claude then made the 3 amendments above. The rename broke the script once: `late` was declared twice, and the suite caught it.
  - The late-first test fails without the change (RED, mutation) and passes with it.
  - Final: e2e 137/137, server 26/26, Python ok, validate ok.
  - Published 14:08 (app fingerprint 354155c6efe2); mirror artifact v4; mirror data unchanged (nothing to send).
- 14:15–15:20 Erik and Adam's instructions on the prioritisation and GTM sheets:
  - Gather, draft and review workflow: live KloudOps, Confluence, RT and Jira read-only; every transcript instruction extracted.
  - Draft in `outbox/ACS_Prioritisation_GTM_Release_3_Oct.md`; the recommendation changed to the Wed 1:1 (no Monday DM).
  - Data applied and verified: tasks E15–E18 and A12, 47 docs, the MP planned-prod date, DEC-035 priority 9, release-notes corrections.
  - Published; mirror synced to 745 docs.
- ~13:10 Hooks installed in `.claude/settings.json`. The Stop dry run returns exit 0 in 0.3 s with nothing to do. CLAUDE.md, the README folder map and memory are updated. Temp copies holding confidential data are deleted.

# SCR-009 — Notifications

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-009/ST-## -->`.

|                  |                                                                                          |
| ---------------- | ---------------------------------------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)                                        |
| **Traces to**    | REQ-028, REQ-018, NFR-004                                                                |
| **Surface**      | `apps/ui` `features/notifications` — the bell panel in the app shell, and `/notifications` |
| **Primary user** | Employee — the same person acting as Learner and Trainer on one account                  |
| **Status**       | draft — awaiting designer review                                                         |

## Purpose

The place an employee finds out that something happened to them while they were
elsewhere: a request came in, a booking was approved or rejected, a session was
cancelled, one is about to start, or a completed session is waiting on their
feedback (REQ-028). It is also the only surface on which REQ-018's auto-rejection
reaches the learner at all — nothing else tells them a request expired.

"Done" for the employee is one of two things: they see the bell is quiet and
carry on, or they open it, understand what changed, and get taken to the screen
where they can do something about it.

This screen is **two presentations of one list**, specified together because the
content, the ordering and the read/unread rules are identical:

| Presentation                        | Reached by                                          | Holds                                                |
| ----------------------------------- | --------------------------------------------------- | ---------------------------------------------------- |
| **Panel** — dropdown from the bell  | the bell in `app-header`, on any authenticated screen | the 5 most recent, newest first                     |
| **Page** — `/notifications`         | "See all" in the panel; "View all →" on the dashboard's Recent activity | the full list, filterable, paged |

They are one screen spec and not two because a second spec would mean two
descriptions of the same eight notification kinds, drifting apart at the first
change. Where the two presentations genuinely differ, the states below say so.

### What this screen deliberately does not do

Settled at design: **a notification points, it never acts.** Clicking "Arun
requested a Python session" opens SCR-007 with that request in view; Approve and
Reject stay where they already live. See the structural decisions table for why.

### Requirements this screen touches but does not fully serve

Recorded so a later audit doesn't read the manifest as "covered":

| Requirement                          | What this screen does                                                                    | Where the rest lives                                                                            |
| ------------------------------------ | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| REQ-018 (auto-reject an unanswered request) | Delivers the "and notify the learner" half — kind N5 below                        | The auto-rejected _outcome_ on SCR-007 and SCR-008, and the cutoff itself (BRD OQ-1) — not yet designed |
| REQ-006 (dashboard shows recent notifications) | Gives the dashboard's "Recent activity → View all" link a destination           | The dashboard section itself is SCR-004                                                        |
| REQ-017, REQ-020, REQ-023 (approve/reject, cancel, rate) | Announces that each happened, and routes to the screen that owns it | SCR-007 and SCR-008 own the actions                                                            |

## The eight notification kinds

REQ-028 names five events. Two of them are one-per-party (an approval notifies
the learner; a cancellation notifies whichever party did not cancel), and the
trainer's "a request arrived" is implied by REQ-017 but not written down. So the
five events resolve to eight kinds. Each has an icon, a written label, a
sentence, and a destination — never a colour alone (see accessibility).

| #  | Kind                     | Goes to | Reads                                                            | Opens              | From    |
| -- | ------------------------ | ------- | ---------------------------------------------------------------- | ------------------ | ------- |
| N1 | Request received         | Trainer | "Arun K. requested a Python session · Thu 14:00"                 | SCR-007 My Teaching | REQ-017 |
| N2 | Request sent             | Learner | "Your request to Priya M. was sent · Thu 14:00"                  | SCR-008 My Learning | REQ-028 |
| N3 | Request approved         | Learner | "Priya M. approved your Excel session · Thu 14:00"               | SCR-008 My Learning | REQ-017 |
| N4 | Request rejected         | Learner | "Priya M. declined your Excel session · Thu 14:00"               | SCR-008 My Learning | REQ-017 |
| N5 | Request expired          | Learner | "Your request to Priya M. expired without an answer · Thu 14:00" | SCR-008 My Learning | REQ-018 |
| N6 | Session cancelled        | Either  | "Arun K. cancelled your Python session · Fri 11:00"              | SCR-008 / SCR-007   | REQ-020 |
| N7 | Session reminder         | Either  | "Your Excel session with Priya M. starts in 1 hour"              | SCR-008 / SCR-007   | REQ-028 |
| N8 | Feedback prompt          | Learner | "How was your Excel session with Priya M.?"                      | SCR-008 My Learning | REQ-023 |

N5's wording says **expired**, not "rejected", even though REQ-018 calls it an
auto-rejection. A learner reading "Priya declined" when Priya never saw the
request would draw the wrong conclusion about a colleague. The system state is
rejected; the sentence describes what actually happened.

Two kinds a reasonable reader might expect are **not** designed, because no
requirement asks for them — see the open questions: a certificate-ready
notification (REQ-024 generates one silently) and a rating-received notification
for the trainer (REQ-023).

## Layout

**Panel.**

Anchored under the bell, right-aligned to it, `--shadow-lg` over the page. Fixed
width on desktop; a full-screen sheet below the breakpoint (ST-06). Header row,
list, footer link — the list is the only part that scrolls, so "Mark all read"
and "See all" are always reachable without scrolling to find them.

```
                                     ┌ bell ─┐
┌────────────────────────────────────────────────┐
│ Notifications                    Mark all read │  ← header, fixed
├────────────────────────────────────────────────┤
│ ● 📩 Request received                          │
│    Arun K. requested a Python session          │
│    Thu 14:00 · 2h ago                          │
├────────────────────────────────────────────────┤
│ ● ✅ Approved                                   │
│    Priya M. approved your Excel session        │
│    Thu 14:00 · 3h ago                          │
├────────────────────────────────────────────────┤
│   ⏰ Reminder                                   │
│    Your Excel session starts in 1 hour         │
│    yesterday                                   │
├────────────────────────────────────────────────┤
│                 See all →                      │  ← footer, fixed
└────────────────────────────────────────────────┘
```

The unread dot sits at the **start** of the row, in the reading direction,
alongside the kind's icon and its written label — so the answer to "which of
these have I not seen?" is one vertical scan down a single column.

**Page.**

The application shell, then a constrained centre column matching the dashboard's:
title, a filter row, and the list grouped under time headings.

```
┌──────────────────────────────────────────────────────────┐
│ SkillEx   [ main navigation ]                  🔔3  (JJ)▾│
├──────────────────────────────────────────────────────────┤
│                                                          │
│   Notifications                     [ Mark all read ]    │
│                                                          │
│   ( All )  ( Unread 3 )  ( Bookings )  ( Sessions )      │
│                                                          │
│   Today                                                  │
│   ┌────────────────────────────────────────────────┐     │
│   │ ● 📩 Request received                          │     │
│   │   Arun K. requested a Python session           │     │
│   │   Thu 14:00 · 2h ago                           │     │
│   └────────────────────────────────────────────────┘     │
│   ┌────────────────────────────────────────────────┐     │
│   │ ● ✅ Approved                                   │     │
│   │   Priya M. approved your Excel session         │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│   Earlier                                                │
│   ┌────────────────────────────────────────────────┐     │
│   │   ⏰ Reminder                                   │     │
│   │   Your Excel session starts in 1 hour          │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│                  [ Show older ]                          │
└──────────────────────────────────────────────────────────┘
```

**Time headings** are Today / Yesterday / Earlier. They exist so a relative
timestamp ("2h ago") never has to be decoded into a date to know roughly when
something happened.

**Filters** are All · Unread · Bookings · Sessions. Four, not eight — one per
kind would be a filter row longer than most people's list. Bookings covers
N1–N5, Sessions covers N6–N8. The filter is a presentation choice, not stored
state: leaving the page and coming back returns to All.

**Page size** is **20**, newest first, with **Show older** appending the next
20. Chosen over infinite scroll so the footer of the page stays reachable and
the keyboard user has a real control to land on.

## States

Twenty-one, split by presentation. The panel and the page each carry the full
default/loading/empty/error floor, because they load independently — opening the
page after the panel is a second request that can fail on its own.

### ST-01 Panel — default

- **When** the employee opens the bell and the list has loaded with at least one notification
- **Shows** the panel header ("Notifications" + "Mark all read", the latter present only while an unread exists); up to 5 rows newest first, each with unread dot (unread only), kind icon, kind label, sentence, and a relative timestamp; the "See all" footer. Unread rows carry the dot, a heavier sentence weight, and a tinted row background — three signals, so the tint is never load-bearing on its own.
- **Can do** open any row (→ ST-16); mark all read (→ ST-17); follow "See all" to the page; close with Escape or a click outside

### ST-02 Panel — loading

- **When** the panel has opened and the request is in flight
- **Shows** the panel at its full width with its header, and three muted skeleton rows at the height a real row occupies. The panel is sized before the data lands so it does not resize under the cursor.
- **Can do** close the panel. Nothing inside is interactive. "Mark all read" is absent, not disabled — there is nothing yet to mark.

### ST-03 Panel — nothing yet

- **When** the list loaded and the employee has never received a notification
- **Shows** the panel header and, in place of rows, a centred line — bell icon + "No notifications yet." + one line of secondary text: "We'll tell you here when someone books, approves, or cancels a session." No "See all" footer: a page of nothing is not worth a click.
- **Can do** close the panel

### ST-04 Panel — could not load

- **When** the panel's request fails
- **Shows** the panel header and, in place of rows, alert icon + "Couldn't load notifications." + a **Try again** button. The panel stays open; the page behind it is untouched and fully usable.
- **Can do** retry (→ ST-02); close the panel

### ST-05 Panel — everything read

- **When** the list loaded, notifications exist, and none is unread
- **Shows** the same rows as ST-01 with no dots and no tint, and **no "Mark all read"** — a control that would do nothing is removed rather than disabled. The bell carries no badge.
- **Can do** open any row; follow "See all"; close

### ST-06 Panel — narrow width

- **When** the viewport is below the desktop breakpoint and the bell is tapped (NFR-004)
- **Shows** the list as a full-screen sheet rather than a dropdown: its own title bar with a close (×) at the trailing edge, "Mark all read" beneath it, the same rows at full width, "See all" pinned at the bottom. A 320px-wide dropdown anchored to a bell near the screen edge is unreadable and hard to dismiss; a sheet is neither.
- **Can do** everything ST-01 allows; close returns focus to the bell
- **Note** this is a presentation change only. Every other panel state renders inside the sheet at this width.

### ST-07 Bell — unread count at the cap

- **When** unread notifications number **10 or more**
- **Shows** the badge reading **9+** rather than a growing numeral, with the accessible name giving the true figure — "Notifications, 12 unread" — so the exact number is available to a screen reader without the badge changing width and shifting the header layout.
- **Can do** open the panel as normal
- **Note** below the cap the badge shows the numeral, which `app-header` already specifies. Only the capped render is new here.

### ST-08 Page — default

- **When** `/notifications` has loaded with at least one notification
- **Shows** the app shell; the title row with "Mark all read" (present only while an unread exists); the four filters with All selected and Unread carrying its count; rows grouped under Today / Yesterday / Earlier, newest first, up to 20; **Show older** when more exist. Rows render exactly as in the panel, at full column width.
- **Can do** open a row (→ ST-16); switch filter (→ ST-12); mark all read (→ ST-17); show older (→ ST-14)

### ST-09 Page — loading

- **When** the page has mounted and its request is in flight
- **Shows** the app shell in full, the title, and skeleton rows under a single skeleton time heading. The filter row is rendered but inert, with no counts — the count is not known yet.
- **Can do** use the header

### ST-10 Page — nothing yet

- **When** the page loaded and the employee has never received a notification
- **Shows** the app shell, the title, **no filter row** — nothing to filter — and the ST-03 empty panel content at page scale, with a **Find a trainer** link as the one thing worth doing next.
- **Can do** follow that link; use the header

### ST-11 Page — could not load

- **When** the page's request fails
- **Shows** the app shell as normal and, in place of the list, alert icon + "We couldn't load your notifications." + **Try again**. Header stays usable so the employee is never trapped.
- **Can do** retry (→ ST-09); use the header

### ST-12 Page — filtered, with results

- **When** a filter other than All is selected and at least one notification matches
- **Shows** that filter visibly selected — a filled background **and** `aria-pressed="true"`, never colour alone — the list reduced to matches, still under its time headings, and the result count read out politely. Time headings with no matching rows are not rendered.
- **Can do** switch to another filter; clear back to All; open a row

### ST-13 Page — filtered, nothing matches

- **When** a filter is selected and no notification matches — most often Unread, after everything has been read
- **Shows** the filter row intact with the selection still visible, and a single line in place of the list: "Nothing here." + "No unread notifications." for the Unread filter, "No booking notifications." for Bookings, and so on, plus a **Show all** link.
- **Can do** clear the filter; pick another
- **Why it is not ST-10:** "you have nothing" and "nothing matches this filter" are different facts, and offering "Find a trainer" to someone who just filtered would be answering a question they did not ask.

### ST-14 Page — loading older

- **When** **Show older** has been pressed and the next page is in flight
- **Shows** the button replaced in place by a spinner and the word "Loading…". Everything already on the page stays fully interactive.
- **Can do** open any already-loaded row; nothing on the button itself
- **On failure** the button is restored with an inline line beneath it: "Couldn't load older notifications. Try again." — rows already on screen are never removed because a later page failed.

### ST-15 Page — end of the list

- **When** the last page has loaded and nothing older exists
- **Shows** no **Show older** button, and a quiet closing line — "That's everything." A button that yields nothing is worse than no button.
- **Can do** everything ST-08 allows

### ST-16 Opening a notification

- **When** a row is clicked or activated from the keyboard, in either presentation
- **Shows** that row losing its unread dot and tint immediately — optimistically, before the navigation resolves — and the bell count decrementing. The panel closes. The destination screen (per the kinds table) opens with the referenced item scrolled into view and briefly highlighted, so the connection between what was read and what is now on screen is visible.
- **Can do** whatever the destination screen allows
- **On failure of the read-marking call** the navigation still happens — it is the thing the employee asked for — and the unread mark is silently restored. Nothing is announced: a failed bookkeeping call is not the employee's problem, and they are now on a different screen.

### ST-17 Marking all read — in flight

- **When** "Mark all read" is pressed in either presentation
- **Shows** the control replaced by a spinner and disabled. Rows stay as they are — they change on success, not on click, because this control clears the whole list and a mistaken optimistic clear is expensive to undo.
- **Can do** nothing on that control; rows remain openable

### ST-18 Marked all read

- **When** the call succeeds
- **Shows** every dot and tint removed together, the bell badge gone, and "Mark all read" removed (→ the surface is now ST-05 or the all-read form of ST-08). A polite announcement: "All notifications marked as read." If the Unread filter is active on the page, the surface becomes ST-13.
- **Can do** everything the read state allows

### ST-19 Mark all read — failed

- **When** the call fails
- **Shows** the control restored and enabled, with an inline error beside it: alert icon + "Couldn't mark those as read." Dots and the badge are untouched, so what the employee sees still matches the truth.
- **Can do** retry; carry on reading rows individually

### ST-20 Something arrives while the surface is open

- **When** a new notification is raised while the panel or page is open
- **Shows** the bell count incrementing, and the new row appearing at the top of the list — the panel dropping its oldest row to stay at 5. On the page, the row inserts under Today. The insertion is announced politely, once, naming the count rather than the content: "1 new notification." The list does **not** scroll or move focus.
- **Can do** everything the underlying state allows
- **Why the row inserts rather than a "1 new" bar appearing:** at this scale notifications arrive minutes apart, not seconds. A row sliding in at the top of a list nobody is mid-scroll through is legible; a deferred-load bar is machinery for a volume this product does not have.
- **Note** whether notifications arrive without a reload at all is a build question, not a design one — see open question OQ-D4. If they do not, this state does not occur and no other state changes.

### ST-21 What it points at is gone

- **When** a row is opened whose target no longer exists — a session cancelled and cleared, a request removed
- **Shows** the destination screen rendering its own not-found handling, and the row, on return, marked read and carrying one line of muted secondary text: "This session is no longer available." The row is never silently deleted: an employee who remembers being told something must still be able to find the thing they were told.
- **Can do** everything else the list allows; the row itself is no longer a link and is not focusable as one

## Components

| Component             | Preview                                                        | States covered                                                                     |
| --------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `notification-row`    | `inception/design/components/notification-row/preview.html`    | all eight kinds, unread and read, opened, stale — ST-01, ST-05, ST-08, ST-16, ST-21 |
| `notification-panel`  | `inception/design/components/notification-panel/preview.html`  | panel default, loading, empty, error, all-read, narrow sheet, mark-all in flight / done / failed, arrival, bell cap, page filters and paging — ST-02, ST-03, ST-04, ST-06, ST-07, ST-09, ST-10, ST-11, ST-12, ST-13, ST-14, ST-15, ST-17, ST-18, ST-19, ST-20 |
| `app-header`          | `inception/design/components/app-header/preview.html`          | the bell that opens the panel, with and without a count                            |
| `empty-state`         | `inception/design/components/empty-state/preview.html`         | the page-scale empty behind ST-10                                                  |
| `alert`               | `inception/design/components/alert/preview.html`               | the error variants behind ST-04, ST-11, ST-19                                      |
| `button`              | `inception/design/components/button/preview.html`              | Try again, Show older, Mark all read                                               |
| `spinner`             | `inception/design/components/spinner/preview.html`             | the in-flight indicator for ST-14 and ST-17                                        |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Opening the panel:** the bell is a button with `aria-expanded` and an accessible name carrying the unread count ("Notifications, 3 unread"). Enter or Space opens it and moves focus to the panel's first focusable element. The panel is a `role="dialog"` labelled by its heading, with focus trapped inside it.
- **Closing the panel:** Escape, or a click outside, closes it and returns focus to the bell — never to `<body>`. The narrow-width sheet (ST-06) closes the same way, plus its × button.
- **Keyboard within the list:** each row is a single link — one tab stop per notification, not one per element inside it. Tab order is header controls → rows in visual order → footer link. Time headings are not focusable.
- **Focus after an action:** ST-18 removes the "Mark all read" control, so focus moves to the panel heading rather than being dropped. ST-14 restores focus to the **Show older** button when the next page fails, and to the first newly-loaded row when it succeeds.
- **Focus:** every row, filter and button shows the `--focus-ring` outline on keyboard focus. Rows are focusable as a whole, so the ring outlines the entire row.
- **Non-colour signalling (NFR-003 in spirit, and the framework's standing rule):** unread is a dot **and** a heavier weight **and** a tint — remove the colour and two signals remain. Each of the eight kinds carries an icon **and** a written label; nothing is distinguished by hue alone, which matters most for approved (N3) versus rejected (N4) versus expired (N5), the three that sit next to each other and mean opposite things. A selected filter is filled **and** `aria-pressed`.
- **Announcements:** the list is a `role="feed"`-style region labelled "Notifications". ST-02 and ST-09 skeletons are `aria-busy="true"`. ST-18's confirmation, ST-12's result count and ST-20's arrival are `aria-live="polite"`; ST-04 and ST-11 are `aria-live="assertive"`. ST-20 announces a count, never the notification's text, so an employee mid-task is not read a sentence they did not ask for.
- **Timestamps:** the visible text is relative ("2h ago"); the exact local time sits in a `title` and in the accessible name, because "2h ago" is ambiguous to anyone returning after a break.
- **Responsive (NFR-004):** the panel becomes a sheet below the breakpoint (ST-06). The page's filter row scrolls horizontally rather than wrapping to two lines. Rows reflow to full width; the timestamp moves under the sentence rather than being truncated.
- **Motion:** ST-20's insertion is a short fade, suppressed under `prefers-reduced-motion`, where the row simply appears.

## Structural decisions

| Decision                                                                       | Rationale                                                                                                                                                                                                                                                      | Alternative rejected                                                                                                                                                     |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A panel for the glance and a page for the record, both specified here          | Designer's decision, 2026-08-28. The panel keeps the common case — "anything for me?" — on the screen the employee is already using; the page gives them somewhere to look back and filter. One spec, because the rows and rules are identical                 | Page only (every glance costs a navigation and loses the current screen); panel only (nowhere to filter or look back, and cramped on a phone)                            |
| A notification points to the owning screen; it never carries Approve or Reject | Designer's decision, 2026-08-28. Each action then exists in exactly one place, with one set of in-flight, success and failure states to design, build and test. Inline actions would put Approve in the panel, the dashboard and SCR-007 — three implementations of one rule | Inline actions on the row — one click faster on the common path, at the cost of triplicating every booking action and its failure modes                        |
| A notification is read only when opened, plus an explicit "Mark all read"      | Designer's decision, 2026-08-28. Nothing is cleared behind the employee's back; the badge stays honest, and there is a one-click escape hatch when it stops being useful                                                                                       | Marking everything read on opening the panel — a tidier badge, but a glance made while walking silently discards the thing they meant to return to                     |
| The list is one shared component across panel and page                        | The row is where the eight kinds, the read/unread treatment and the accessibility rules live. Two row implementations would drift on the first change to a kind                                                                                                 | Separate compact and full rows — more control over each presentation, and two places to fix every future kind                                                            |
| Four filters (All, Unread, Bookings, Sessions), not one per kind               | Eight filters is a filter row longer than most POC-scale lists. These four match the questions people actually ask: what's new, what's about bookings, what's about sessions                                                                                    | A filter per kind; or a search box — search over one screen of your own notifications is machinery without a job                                                        |
| Filter selection is not remembered between visits                             | Returning to a page still filtered from last week and seeing "Nothing here" reads as a broken product, not as a filter                                                                                                                                          | Persisting the last filter                                                                                                                                              |
| "Show older" in pages of 20, not infinite scroll                              | The page footer stays reachable, and a keyboard user gets a real control instead of a scroll position that loads more                                                                                                                                           | Infinite scroll — smoother on a phone, but no reachable end and nothing to focus                                                                                        |
| The unread badge caps at 9+                                                   | A growing numeral changes the badge's width and shifts the header layout around it. The exact figure stays available in the accessible name                                                                                                                     | An uncapped count; or a plain dot with no number, which answers "something happened" but not "how much"                                                                  |
| N5 reads "expired without an answer", not "rejected"                          | The system state is rejected (REQ-018), but the trainer never saw it. Wording it as a rejection would tell a learner something untrue about a colleague                                                                                                        | Reusing the rejection wording and icon for both, which is what the requirement's own language would produce                                                             |
| A row whose target is gone is kept and annotated, never removed               | An employee who was told something must be able to find that they were told it. Silent deletion makes the product look like it lied                                                                                                                             | Dropping stale rows from the list; or leaving them as live links that lead to a not-found screen with no explanation                                                     |
| The panel shows 5 items                                                       | Matches the dashboard's per-section cap (SCR-004), so the two summaries feel like one product. It is a starting default and the designer's to change                                                                                                            | 10 in the panel — a taller dropdown that on a small laptop reaches the bottom of the viewport                                                                            |

## Conflicts

No requirement conflicts on this screen. REQ-028, REQ-018 and NFR-004 are all
satisfiable together as specified above.

| #   | Conflict | Between | Owner | Status |
| --- | -------- | ------- | ----- | ------ |
| —   | none     | —       | —     | —      |

## Open questions

Each has a working default, so none blocks drawing the frames — but each is
someone else's call to confirm, and the answer may change a kind, a threshold or
a piece of copy.

| #     | Question                                                                                                                                                                                            | Working default                                                                                                              | Owner          | Status                     |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------------- | -------------------------- |
| OQ-D1 | REQ-028 says "booking confirmation" without saying whose. It could mean the learner's "your request was sent", the trainer's "a request arrived", or the confirmed-session moment (which REQ-017's approval already covers) | Both readings are designed as separate kinds — N2 to the learner, N1 to the trainer                                          | BA (`/ba`)     | open                       |
| OQ-D2 | REQ-024 generates a certificate automatically, and REQ-028 does not list it as a notifiable event. Should the learner be told their certificate is ready?                                            | No certificate notification. It surfaces on SCR-008's history, where REQ-026 puts it                                          | BA (`/ba`)     | open                       |
| OQ-D3 | REQ-023's rating is submitted by the learner. Should the trainer be notified that a rating arrived? REQ-028 does not list it                                                                         | No rating-received notification. It appears in the trainer's history (SCR-007, REQ-027)                                       | BA (`/ba`)     | open                       |
| OQ-D4 | REQ-028 does not say whether a notification must appear without a page reload. That decides whether ST-20 can happen at all                                                                          | Notifications refresh when a screen loads and when the panel is opened. ST-20 is specified so live arrival is not a redesign if it is built | Architect (`/architect`) | open |
| OQ-D5 | How long before a session should the reminder (N7) be raised? REQ-028 says "session reminders" without a time                                                                                        | One reminder, 1 hour before start — consistent with SCR-004's 15-minute "starting soon" promotion being a separate, later signal | Business owner | open                       |
| OQ-D6 | BRD OQ-1 (the auto-reject cutoff) is unresolved. N5's sentence is written to work at any cutoff, but if the value is one employees should anticipate, a pending request may also need a warning notification | N5 announces the expiry after the fact only; no advance warning notification                                                  | Business owner | open (tracked as BRD OQ-1) |
| OQ-D7 | Do notifications ever expire or get cleared out, or does the list grow for the life of the account?                                                                                                 | The list is kept indefinitely and paged. At POC scale nobody reaches a volume where this matters                              | BA (`/ba`)     | open                       |
| OQ-D8 | ~~The eight kinds' sentences are product copy — tone, whether colleagues are named, first name versus full name~~ | **Resolved 2026-08-28 — first name plus surname initial.** "Priya M. approved your request." Matches the rows on SCR-004 and SCR-007, so one colleague reads the same way everywhere in the portal, and two colleagues sharing a first name stay distinguishable | Designer | closed |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via
Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the
numbering is the checklist. ST-01 and ST-08 are each worth two frames — desktop
and mobile width — and ST-06 is the mobile sheet in its own right, since NFR-004
is what the reflow has to prove. The eight kinds in the table above are one
frame between them: a single row rendered eight times.

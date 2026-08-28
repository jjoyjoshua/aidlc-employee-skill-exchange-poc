# SCR-006 — Find a trainer

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-006/ST-## -->`.

|                  |                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)                                     |
| **Traces to**    | REQ-012, REQ-013, REQ-015, REQ-025, NFR-004                                           |
| **Surface**      | `apps/ui` `features/trainers` — `/trainers`, reached from "Find a trainer" in the app-header (SCR-004) |
| **Primary user** | Employee acting as a **Learner** — looking for a colleague who can teach them something |
| **Status**       | draft — awaiting designer review                                                      |

## Purpose

The screen where an employee answers _"who here can teach me this?"_ and then asks for a time. It lists every colleague who has declared a teachable skill (REQ-013), narrows that list by skill, department, proficiency level and rating (REQ-012), shows each trainer's average star rating (REQ-025), and — in a panel opened from a result — lets the learner request one of that trainer's published time slots (REQ-015).

"Done" for the learner is one of two things: they have sent a booking request and know it was received, or they have established that nobody currently teaches what they need — and are told so in a way that names their next step rather than leaving them on an empty page.

### Requirements this screen touches but does not fully serve

Recorded so a later audit doesn't read the manifest as "covered":

| Requirement                           | What this screen does                                                                   | Where the rest lives                                                                                    |
| ------------------------------------- | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| REQ-014 (trainer publishes slots)     | **Reads** published slots in the panel and renders them                                 | Publishing, editing and withdrawing slots is a trainer-side screen — not yet designed                   |
| REQ-016 (learner sees request status) | Confirms the request was sent, and marks that slot "Requested" for the rest of the visit | Ongoing status (Pending / Approved / Rejected) lives on SCR-004 and a bookings screen — not yet designed |
| REQ-021 (one learner per slot)        | Shows an already-booked slot as unavailable and non-requestable (ST-13)                 | The rule itself is enforced server-side; the screen renders it, it does not guarantee it                |
| REQ-028 (notifications)               | Tells the learner the trainer has been notified (ST-15)                                 | Raising and displaying the notification itself — not yet designed                                       |
| REQ-023 (rating comments)             | Shows the numeric average and how many ratings it came from                             | Where written comments surface is undecided — see OQ-S5                                                 |

## Layout

One application shell (inherited unchanged from SCR-004), then a single constrained centre column: page title, search box, filter row, result count, result list. Selecting a result slides a panel in from the right **over** the results; the list stays visible behind it at desktop width and is fully covered at phone width.

```
DESKTOP — ST-01
┌────────────────────────────────────────────────────────────────┐
│ SkillEx   [ main navigation ]                        🔔3  (JJ)▾│  ← app-header
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   Find a trainer                                               │
│   ┌──────────────────────────────────────────────┐             │
│   │ 🔍 Search a skill…                           │             │
│   └──────────────────────────────────────────────┘             │
│   [ Department ▾ ] [ Level ▾ ] [ Rating ▾ ]                    │
│                                                                │
│   8 colleagues teaching                                        │
│   ┌──────────────────────────────────────────────────────┐     │
│   │ (PM)  Priya M.                                       │     │
│   │       Finance · Financial Analyst                    │     │
│   │       ★★★★☆ 4.6 · 12 ratings                         │     │
│   │       Teaches  [Excel · Expert]  [SQL · Intermediate]│     │
│   │                              [ View availability ]   │     │
│   └──────────────────────────────────────────────────────┘     │
│   ┌──────────────────────────────────────────────────────┐     │
│   │ (AK)  Arun K.                                        │     │
│   │       Engineering · Senior Developer                 │     │
│   │       ☆ Not rated yet                                │     │
│   │       Teaches  [Python · Expert]                     │     │
│   └──────────────────────────────────────────────────────┘     │
└────────────────────────────────────────────────────────────────┘

DESKTOP — ST-11, panel open over the results
┌──────────────────────────────┬─────────────────────────────────┐
│ ░ results stay visible ░     │ (PM) Priya M.               ✕   │
│ ░ (dimmed, inert)      ░     │ Finance · Financial Analyst     │
│ ░                      ░     │ ★★★★☆ 4.6 · 12 ratings          │
│ ░                      ░     │ ─────────────────────────────── │
│ ░                      ░     │ Teaches                         │
│ ░                      ░     │  [Excel · Expert]               │
│ ░                      ░     │  [SQL · Intermediate]           │
│ ░                      ░     │ ─────────────────────────────── │
│ ░                      ░     │ Available times                 │
│ ░                      ░     │  Thu 3 Sep · 14:00 · 45 min     │
│ ░                      ░     │                    [ Request ]  │
│ ░                      ░     │  Fri 4 Sep · 11:00 · 60 min     │
│ ░                      ░     │                    [ Request ]  │
│ ░                      ░     │  Mon 7 Sep · 09:00 · 45 min     │
│ ░                      ░     │           🔒 Already booked     │
└──────────────────────────────┴─────────────────────────────────┘

PHONE — same panel, full-height sheet (ST-19)
┌────────────────────┐   ┌────────────────────┐
│ 🔍 Search a skill… │   │ ← (PM) Priya M.  ✕ │
│ [Dept▾][Level▾] →  │   │ Finance · Analyst  │
│ 8 colleagues       │   │ ★★★★☆ 4.6 · 12     │
│ ┌────────────────┐ │ → │ ────────────────── │
│ │ (PM) Priya M.  │ │   │ Teaches            │
│ │ ★★★★☆ 4.6      │ │   │  [Excel · Expert]  │
│ │ [Excel·Expert] │ │   │ ────────────────── │
│ └────────────────┘ │   │ Available times    │
│ ┌────────────────┐ │   │  Thu 3 Sep · 14:00 │
│ │ (AK) Arun K.   │ │   │       [ Request ]  │
│ └────────────────┘ │   │                    │
└────────────────────┘   └────────────────────┘
```

**Reading order, and why.** Search first, filters second, count third, results fourth. The count sits between the controls and the list on purpose: it is the feedback that a filter did something, and it is the only place a learner learns that unrated colleagues were hidden by a rating filter (see the conflicts table).

**No pagination.** The POC is under 10 employees (BRD §1), so every match is rendered. That is a scale assumption, not a design principle — recorded in the decisions table so whoever crosses that threshold knows to revisit it.

## States

### ST-01 Default — everyone who teaches

- **When** the screen has loaded, no search term or filter is set, and at least one colleague has declared a teachable skill
- **Shows** the app-header; the page title; an empty search box reading "Search a skill…"; three filter buttons each reading its own name (Department, Level, Rating) with no value; the count line ("8 colleagues teaching"); and one result card per trainer, ordered alphabetically by name. Each card carries: avatar, name, department and designation, the star rating with its numeric average and rating count (REQ-025), and a chip per teachable skill showing skill name and proficiency (REQ-010). The whole card is one link; a "View availability" action repeats that target explicitly.
- **Can do** type a search; open any filter; open a trainer's panel (→ ST-10)
- **Note** the signed-in employee never appears in their own results — see the decisions table.

### ST-02 First load

- **When** the screen has mounted and the trainer list request is in flight
- **Shows** the app-header and the page title in full — neither needs this data — the search box and filter buttons rendered but disabled, no count line (the number is not yet known), and four placeholder cards: muted skeleton blocks at the height a real card occupies.
- **Can do** navigate away using the header. The search box and filters are inert, so nothing typed can be lost when the real data replaces the skeleton.
- **Why skeletons, not a page spinner:** the page frame is known before the data lands, so nothing re-flows when it does.

### ST-03 Refining

- **When** the learner has changed the search term or any filter, and the updated result request is in flight
- **Shows** the search box and filters staying **fully live and fully populated** with what the learner just chose. The count line is replaced by a quiet "Searching…" with a small spinner. The previous results remain on screen, dimmed and marked `aria-busy`, rather than being cleared.
- **Can do** keep typing, change another filter, or clear everything — each supersedes the in-flight request. Result cards are not clickable while dimmed.
- **Why the old results stay:** blanking the list on every keystroke makes the page flicker between empty and full, and a learner mid-refinement loses the context they were comparing against. Search input is debounced, so one refinement is one request rather than one per character (NFR-003).

### ST-04 Nobody teaches anything yet

- **When** the list loaded successfully, no filter is applied, and **no** colleague in the company has declared a teachable skill
- **Shows** the search box and filters hidden — there is nothing to search — and a single panel in place of the list: one line explaining that no one has listed a skill to teach yet, and one primary action, **Add a skill you can teach**, pointing at SCR-005. Nobody is blamed, and the learner is given the one move that fixes it — REQ-013 means their own listing takes effect immediately.
- **Can do** follow that action; use the header
- **Why the controls disappear here but not in ST-05:** filtering an empty set is a control that cannot do anything. Hiding it removes a dead end rather than presenting one.

### ST-05 Nothing matches

- **When** the list loaded successfully, at least one filter or search term is set, and no trainer matches it
- **Shows** the search box and filters staying exactly as the learner left them — never reset — with the count line reading "No matches", and beneath it a short panel: "No one is currently teaching Excel in Finance." plus two actions, **Clear filters** (primary) and **Add it to what you want to learn** (secondary, → SCR-005). The sentence names the terms actually in force, so the learner can see which one to relax.
- **Can do** clear filters (→ ST-01); change any filter; add the skill to their learning list
- **Special case** when the only reason for zero results is the rating filter hiding unrated colleagues, this state also carries the reveal line described in the conflicts table.

### ST-06 Couldn't load

- **When** the trainer list request fails — timeout, server error, no connection
- **Shows** the app-header and title as normal, and in place of the search, filters and list, one error panel: alert icon + "We couldn't load the list of trainers." + a **Try again** button. The header stays usable so the learner is never trapped.
- **Can do** retry (→ ST-02); use the header

### ST-07 A filter open

- **When** the learner activates the Department, Level or Rating button
- **Shows** a menu anchored under that button, over the page. **Department** and **Level** are multi-select checkbox lists — departments from the fixed IT-maintained list (REQ-008), levels Beginner / Intermediate / Expert (REQ-010). **Rating** is single-select — Any rating / 3★ and up / 4★ and up / 4.5★ and up — because "at least 3 **and** at least 4" is not something a person means. Each option shows how many trainers it would leave. The button stays visible and marked as expanded.
- **Can do** toggle options — each takes effect immediately (→ ST-03), with no Apply button; close with Escape or by clicking away
- **Note** with no skill searched, **Level** means "is at this level in at least one skill they teach". With a skill searched, it means their level in that skill. The menu states which reading is in force in one line of helper text, because a control that silently answers two different questions is how a filter gets mistrusted.

### ST-08 Filters in force

- **When** at least one filter or search term is set and results have loaded
- **Shows** each active filter button carrying its value and count ("Department: Finance", "Level: Expert +1"), marked active with a **filled dot as well as** the colour change; a **Clear all** text button appearing at the end of the row only when something is set; and the count line reading the result against the total ("3 of 8 colleagues").
- **Can do** clear one filter from its own menu; clear everything at once; open a result
- **Why "3 of 8":** a bare "3 colleagues" hides the fact that five were filtered out, which is exactly the fact a learner needs in order to decide whether to relax something.

### ST-09 A colleague with no ratings yet

- **When** a trainer in the results has never been rated — the normal state of anyone who has just declared a skill (REQ-013)
- **Shows** that card's rating line reading "☆ Not rated yet" in secondary text, in the same position the star average occupies on every other card. Not an empty gap, not zero stars, and not a five-star default: "0" would read as a bad trainer and a blank would read as a broken page.
- **Can do** everything ST-01 allows — being unrated never limits what a learner can do with the card
- **Note** whether these colleagues survive an active rating filter is the open conflict below; the working default is described there.

### ST-10 Opening a trainer

- **When** the learner activates a result card or its "View availability" action
- **Shows** the panel sliding in from the right (a full-height sheet at phone width) already carrying everything the result card knew — name, department, designation, rating, skills — with only the "Available times" section in a skeleton state while the slots are fetched. The results behind are dimmed and made inert; a scrim covers them.
- **Can do** close the panel (Escape, the ✕, or the scrim), which returns focus to the card that opened it
- **Why the panel opens populated:** the identity data is already on screen. A wholly empty panel would briefly hide information the learner is looking at right now.

### ST-11 Trainer panel — loaded

- **When** the trainer's published slots have loaded and at least one is available (REQ-014)
- **Shows** the panel in full: header with avatar, name, department, designation and a close control; the rating summary with numeric average and rating count (REQ-025); the "Teaches" section as skill chips with proficiency; then "Available times" as one row per slot — day, date, start time, duration — each with a **Request** button (REQ-015). Slots are ordered soonest first, shown as day-and-clock rather than relative time.
- **Can do** request any available slot (→ ST-14); close the panel

### ST-12 Trainer panel — no times published

- **When** the panel loaded successfully and this trainer has published no bookable slots, or every one has passed
- **Shows** the whole panel as in ST-11, with "Available times" replaced by one line: "Priya hasn't published any times yet." Identity, skills and rating stay fully visible — the learner still learns the right person exists.
- **Can do** close the panel
- **Note** there is deliberately no "notify me" or "ask anyway" action: no requirement defines a second request path, and inventing one would be inventing a business rule. Raised as OQ-S4.

### ST-13 Trainer panel — a slot already taken

- **When** a slot in the list has already been booked by another learner (REQ-021)
- **Shows** that row still listed, its time in muted text with a strikethrough, a lock icon and the words "Already booked" where its Request button would be. No button at all, so there is nothing to click and be refused.
- **Can do** request any other slot; close the panel
- **Why it stays listed rather than disappearing:** a slot vanishing between the learner reading it and reaching for it is the confusing case. Showing it as taken explains what happened.

### ST-14 Requesting a slot

- **When** the learner has activated Request on a slot and the call is in flight
- **Shows** that row's button disabled with a spinner in place of its label ("Requesting…"), and **every other slot's Request button also disabled** for the duration. The rest of the panel stays readable; the close control stays live.
- **Can do** close the panel — the request completes regardless, and its outcome is waiting on the dashboard
- **Why the other buttons lock too:** these are requests against one person's calendar. Three fired in parallel from one panel is a mess the learner did not intend to create.

### ST-15 Request sent

- **When** the booking request is accepted (REQ-015)
- **Shows** that slot's row replaced in place by a confirmation: tick icon + "Requested — Priya has been notified." (REQ-028). The remaining slots' buttons re-enable. A line under the panel header reads "Waiting for Priya to approve. You'll see it on your dashboard." — naming where the status lives (REQ-016), because the panel is not where the learner will come back to check.
- **Can do** request another slot; close the panel

### ST-16 Request failed

- **When** the booking request call fails
- **Shows** that row restored to its requestable form with an inline error beneath it: alert icon + "Couldn't send that request. Please try again." All buttons re-enable. The wording blames the sending, never the request's validity, so nobody reads it as a refusal from the trainer.
- **Can do** retry the same slot; pick a different one; close the panel
- **Special case** if the failure is specifically that the slot was taken while the panel was open, the row becomes ST-13 carrying the line "Someone booked this one just now." A different fact deserves a different sentence.

### ST-17 Already requested

- **When** the learner has already sent a request for this slot — earlier in this visit, or on a previous one
- **Shows** that row with a "Requested" status (clock icon + word) in place of its Request button, so the same slot cannot be asked for twice.
- **Can do** request a different slot; close the panel

### ST-18 Trainer panel — couldn't load

- **When** the slot request for an opened trainer fails
- **Shows** the panel keeping the identity, rating and skills it opened with, and "Available times" replaced by an inline error plus a **Try again** button scoped to that section. The panel is not dismissed and the results behind it are not disturbed.
- **Can do** retry; close the panel

### ST-19 Panel at phone width

- **When** any panel state is shown below the desktop breakpoint (NFR-004)
- **Shows** the same panel as a full-height sheet covering the results entirely, with a back arrow at the left of its header alongside the ✕, and the panel scrolling internally so the trainer's name stays pinned while the slot list scrolls.
- **Can do** everything the equivalent desktop panel state allows; the browser Back gesture closes the sheet rather than leaving the screen
- **Note** this is one state covering the responsive form of ST-10 through ST-18, not a separate flow. Content and actions are identical; only the frame changes.

## Components

| Component           | Preview                                                      | States covered                                                                                    |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| `trainer-card`      | `inception/design/components/trainer-card/preview.html`      | populated, skeleton, unrated, dimmed — ST-01, ST-02, ST-09                                        |
| `search-filter-bar` | `inception/design/components/search-filter-bar/preview.html` | idle, menu open, active with values, searching — ST-03, ST-07, ST-08                              |
| `rating-stars`      | `inception/design/components/rating-stars/preview.html`      | every average, unrated, the desaturation check                                                    |
| `trainer-panel`     | `inception/design/components/trainer-panel/preview.html`     | opening, loaded, no times, load failure, phone sheet — ST-10, ST-11, ST-12, ST-18, ST-19          |
| `slot-row`          | `inception/design/components/slot-row/preview.html`          | available, taken, requesting, sent, failed, already requested — ST-13, ST-14, ST-15, ST-16, ST-17 |
| `empty-state`       | `inception/design/components/empty-state/preview.html`       | nobody teaching, nothing matches — ST-04, ST-05                                                   |
| `alert`             | `inception/design/components/alert/preview.html`             | the list-level load failure — ST-06                                                               |
| `app-header`        | `inception/design/components/app-header/preview.html`        | the shell inherited from SCR-004, unchanged                                                       |
| `skill-chip`        | `inception/design/components/skill-chip/preview.html`        | the teach chips on a card and in the panel                                                        |
| `button`            | `inception/design/components/button/preview.html`            | primary, secondary, loading, disabled                                                             |
| `spinner`           | `inception/design/components/spinner/preview.html`           | the in-flight indicator behind ST-03 and ST-14                                                    |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Keyboard — the page:** header tab order first (unchanged from SCR-004), then search box → Department → Level → Rating → Clear all (when present) → each result card → each card's "View availability". A filter button opens on Enter or Space, moves focus into its menu, moves between options with arrow keys, and closes on Escape returning focus to the button.
- **Keyboard — the panel:** opening moves focus to the panel's heading and **traps** it inside until close. Escape closes. Closing returns focus to the exact card that opened the panel, never to `<body>` — a learner three cards down the list must not be thrown back to the top.
- **Focus:** every interactive element shows the `--focus-ring` outline, including the whole-card link and each slot row's Request button.
- **Non-colour signalling (NFR-003):** the star rating always carries its numeric average as text ("4.6"), never stars alone. "Not rated yet" is words, not an absence. "Already booked", "Requested" and "Requesting…" each carry an icon **and** a word. An active filter is marked with a filled dot as well as a colour change. Proficiency is spelled out (Beginner / Intermediate / Expert), never a colour or a count of dots.
- **Announcements:** the result count line is `aria-live="polite"` and is the single place a filter change is announced — "3 of 8 colleagues" — so a screen-reader user learns a filter worked without tabbing into the list. The dimmed list in ST-03 is `aria-busy="true"`. The panel is `role="dialog" aria-modal="true"`, labelled by the trainer's name. ST-15's confirmation and ST-16's error are announced politely; ST-06's panel is assertive.
- **Screen-reader wording on a card:** the rating is read as "Rated 4.6 out of 5, from 12 ratings", not as a row of star glyphs.
- **Responsive (NFR-004):** the centre column goes full width below the breakpoint; the filter row scrolls horizontally rather than wrapping into a block that pushes results off-screen; the panel becomes the ST-19 sheet; result cards keep their vertical order at every width.
- **Motion:** the panel slide and the ST-03 dim are transitions; under `prefers-reduced-motion` the panel appears without sliding and the dim is instant.

## Structural decisions

| Decision                                                                        | Rationale                                                                                                                                                                                                                                                       | Alternative rejected                                                                                                                                                            |
| ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The full list of teaching colleagues is shown on arrival, before any search      | Designer's decision, 2026-08-28. At POC scale the whole company fits on one screen, so browsing beats guessing, and a learner who doesn't know the catalog's wording can still find someone. It also makes REQ-013 visible — a new trainer is on the page immediately | A skill picker showing nothing until a skill is chosen — a blank first screen, and it hides the catalog from the people least able to guess it                                   |
| A result opens a **side panel over the results**, not a separate trainer page    | Designer's decision, 2026-08-28. The list stays on screen, so comparing two colleagues costs no navigation and no lost filter state                                                                                                                              | A full trainer page (loses the result set on every look); an inline expanding card (no room for slots plus identity plus rating without pushing the list apart)                  |
| The booking request is completed **inside** the panel                            | Designer's decision, 2026-08-28. One unbroken flow from "who teaches Excel" to "I've asked for Thursday"; a separate booking screen would add a navigation step to the portal's core purpose                                                                    | A read-only panel linking out to a booking screen                                                                                                                               |
| Filters are a row of dropdown buttons under the search box, applying instantly   | Designer's decision, 2026-08-28. One design at every width — the row scrolls sideways on a phone instead of needing a second drawer-based layout (NFR-004) — and instant application makes the result count continuous feedback                                | A left filter sidebar (costs horizontal space, needs a separate mobile drawer); all filters behind one button (two extra clicks per filter)                                      |
| Rating is single-select; department and level are multi-select                   | "At least 4 stars and at least 3 stars" is not a question anyone asks; "Finance or Engineering" is                                                                                                                                                               | Uniform multi-select everywhere, for consistency's sake                                                                                                                         |
| The count line reads "3 of 8", not "3"                                           | The number filtered **out** is what tells a learner whether to relax a filter. It is also the one place the hidden-unrated reveal can live                                                                                                                       | A bare result count                                                                                                                                                             |
| Results are ordered alphabetically by name, with no sort control                 | Neutral and stable. Ordering by rating would push every newly-declared trainer to the bottom permanently — the rich-get-richer loop REQ-013 exists to prevent — and at POC scale ordering is a weak signal anyway. Revisit via OQ-S3                            | Highest-rated first (fights REQ-013); most-recently-active first (no requirement defines activity)                                                                              |
| Every match is rendered — no pagination, no result cap                           | Under 10 employees (BRD §1), the whole set is a short page. A cap would be scaffolding for a scale this POC does not have                                                                                                                                        | A "show more" cap mirroring SCR-004's 5-item sections — right for a summary dashboard, wrong for a search whose job is completeness                                              |
| The search box matches **skill names only**, not colleague names                  | REQ-012 says "filterable by skill name". Matching people's names is a capability nobody has asked for, and this is not the place to invent one — so it is raised as OQ-S2 rather than quietly added                                                              | A combined "skill or person" search — probably wanted, but not stated                                                                                                           |
| The signed-in employee is excluded from their own results                        | Booking yourself is not a workflow the BRD contains, and a self card carrying a Request button invites a dead end. Working default, confirmable by the BA (OQ-S1)                                                                                                | Showing themselves marked "(you)" and non-requestable — more to build and more to explain, for a case with no value                                                             |
| Old results stay on screen, dimmed, while a refinement loads (ST-03)             | Clearing on every keystroke strobes the page between empty and full and destroys the comparison the learner was making                                                                                                                                           | Replacing the list with skeletons on every filter change — right on first load, disorienting on the twentieth refinement                                                        |
| A taken slot stays in the list, marked (ST-13), rather than disappearing         | The confusing case is a row vanishing between reading it and reaching for it. "Already booked" explains; absence does not                                                                                                                                        | Filtering booked slots out of the panel entirely                                                                                                                                |
| Requesting one slot disables every other Request button until it settles (ST-14) | These are requests against one person's calendar; three fired in parallel from one panel is a mess the learner did not mean to create                                                                                                                            | Independent per-row locking                                                                                                                                                     |
| Unrated trainers read "Not rated yet" — never 0 stars, never a blank             | "0" reads as a bad trainer; a blank reads as a broken page. Both misrepresent someone whose only fact is that they are new (REQ-013)                                                                                                                             | An implicit neutral default such as 3 stars — fabricates a rating nobody gave                                                                                                   |

## Conflicts

Blocks approval of this screen until resolved.

| #   | Conflict                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Between            | Owner      | Status |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | ---------- | ------ |
| C-1 | **A rating filter permanently hides every new trainer.** REQ-013 puts a colleague into search results the moment they declare a skill, with no gatekeeping. REQ-012 lets a learner filter by trainer rating. A newly-declared trainer has no rating, so any "3★ and up" filter excludes them — and they can only earn a rating by being booked, which cannot happen while they are filtered out. The open, self-declared model REQ-013 exists to create is undone by the filter REQ-012 requires. This is a rule about who is eligible, not a visual choice, so it is the BA's to settle rather than mine. **Working default pending that decision:** unrated colleagues are excluded when a minimum rating is set, and the count line says so and offers the reveal — "3 of 8 colleagues · 2 not-yet-rated colleagues hidden by this filter. **Show them**" — one click that brings them back, listed after the rated ones under a "Not rated yet" subheading. | REQ-012 ↔ REQ-013 | BA (`/ba`) | open   |

## Open questions

Each has a working default, so none blocks drawing the frames — but each is someone else's call to confirm, and the answer may change a rule or a line of copy.

| #     | Question                                                                                                                                                                        | Working default                                                          | Owner         | Status                     |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ------------- | -------------------------- |
| OQ-S1 | Should an employee see themselves in trainer results?                                                                                                                            | No — excluded entirely                                                   | BA (`/ba`)    | open                       |
| OQ-S2 | REQ-012 names skill, department, level and rating. Should the search box also match a colleague's **name**? In a company this size it is the obvious way to find a known person   | Skill names only, as REQ-012 states                                      | BA (`/ba`)    | open                       |
| OQ-S3 | ~~Should learners be able to change the result order (by rating, or by soonest availability)?~~ | **Resolved 2026-08-28 — no sort control.** Alphabetical by name. At POC scale a search returns a handful of colleagues, where a sort control is furniture; and sorting by rating would mostly order colleagues who have no rating yet (REQ-013), which ranks people nobody has ranked. Revisit when the list is long enough for order to carry information | Designer + BA | closed |
| OQ-S4 | A learner opens a trainer with no published times (ST-12). Should they be able to register interest, or is the dead end intended?                                                | Dead end — no requirement defines a second request path                  | BA (`/ba`)    | open                       |
| OQ-S5 | REQ-023 lets a learner leave a written comment with their rating, but no requirement says where comments are read. Should the panel show recent ones?                            | Numeric average and rating count only; no comments on this screen        | BA (`/ba`)    | open                       |
| OQ-S6 | ~~The rating filter's thresholds~~ | **Resolved 2026-08-28 — four steps.** Any / 3★ and up / 4★ and up / 4.5★ and up, with "Any" the default. 4.5 earns its place as the only way to ask for the very best without implying the portal can separate 3.9 from 4.1. **This settles the thresholds only** — whether unrated colleagues survive a threshold is C-1, and remains the BA's | Designer | closed |
| OQ-S7 | Slot rows show a duration ("45 min"). No requirement says whether a slot has a fixed or a trainer-chosen length; this depends on how REQ-014 is specified                        | Duration is shown as the trainer published it; the screen does not constrain it | BA (`/ba`) | open                       |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist. ST-01 and ST-11 are each worth two frames — desktop and phone width — since NFR-004 is what the reflow has to prove, and ST-19 is the phone form of the whole panel family rather than a frame of its own.

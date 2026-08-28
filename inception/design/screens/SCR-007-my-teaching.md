# SCR-007 — My teaching

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-007/ST-## -->`.

|                  |                                                                                                     |
| ---------------- | ----------------------------------------------------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)                                                   |
| **Traces to**    | REQ-014, REQ-017, REQ-021, NFR-004                                                                  |
| **Surface**      | `apps/ui` `features/teaching` — `/teaching`, reached from "My teaching" in the app-header (SCR-004) |
| **Primary user** | Employee acting as a **Trainer** — offering their time and deciding who gets it                     |
| **Status**       | draft — awaiting designer review                                                                    |

## Purpose

The other half of the booking that SCR-006 starts. This is where an employee says _when I am free to teach_ (REQ-014) and then answers the requests that arrive against those times (REQ-017), including the case where two colleagues have asked for the same one and only one can have it (REQ-021).

"Done" for the trainer is one of two things: nobody is waiting on them and their calendar shows the times they meant to offer, or they have cleared the decisions that were waiting — each one recorded and the colleague notified — without leaving the page.

This screen is also where the dashboard's "View all" on Booking requests lands. SCR-004 handles the first five requests inline because that is the fast path; this is the full list, plus the availability that produces it.

### Requirements this screen touches but does not fully serve

Recorded so a later audit doesn't read the manifest as "covered":

| Requirement                            | What this screen does                                                                    | Where the rest lives                                                        |
| -------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| REQ-018 (auto-reject stale requests)   | Shows the trainer's side of it — a request that expired before they answered (ST-20)     | The cutoff itself and the learner's notification — BRD OQ-1, not yet designed |
| REQ-019 (video-meeting link)           | Surfaces the link as "Join meeting" on a booked time (ST-14)                             | Link generation and a session detail view — not yet designed                 |
| REQ-020 (either party cancels)         | Puts "Cancel session" on a booked time, and stops at the confirm step — see OQ-T5         | The cancellation flow, its copy and its notification — not yet designed      |
| REQ-022 (trainer marks complete)       | Shows a past booked time as awaiting completion (ST-15), with no action on it yet         | The completion flow — not yet designed                                       |
| REQ-027 (teaching history)             | Nothing — resolved requests and finished sessions leave this screen                       | A history screen — not yet designed                                          |
| REQ-028 (in-app notifications)         | Every approve, reject and publish states that the colleague has been notified            | The notification surface itself — not yet designed                           |

## Layout

The app shell from SCR-004, then a single constrained centre column holding two section cards, in this order:

1. **Requests waiting for you** — someone else is blocked on this
2. **My available times** — the trainer's own calendar

Requests come first for the same reason they outrank skills on the dashboard: one section holds a decision another person is waiting on, the other holds a list only the trainer cares about. One column, identical order at every width (NFR-004).

```
┌──────────────────────────────────────────────────────────┐
│ SkillEx  Find a trainer  My teaching  My skills   🔔3 (JJ)▾│ ← app-header
├──────────────────────────────────────────────────────────┤
│                                                          │
│   My teaching                                            │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Requests waiting for you (2)                   │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Priya M. wants to learn Excel              │ │     │
│   │ │ Thu 4 Sep, 14:00 · 45 min · asked 2h ago   │ │     │
│   │ │                  [ Approve ]  [ Reject ]   │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ ⚠ 2 people asked for Thu 4 Sep, 14:00.     │ │     │
│   │ │   Approving one declines the other.        │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ My available times (3)         [ + Add a time ]│     │
│   │  Thu 4 Sep                                     │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ 14:00 · 45 min · Excel                     │ │     │
│   │ │ ⏳ 2 requests waiting          [ See them ]│ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ 16:00 · 45 min · Excel                     │ │     │
│   │ │ ✓ Booked — Arun K.  [Join meeting] [Cancel]│ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │  Fri 5 Sep                                     │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ 11:00 · 60 min · Python          [ Remove ]│ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

**Times are grouped by day**, soonest first, with the day as a subheading rather than repeated on every row. A trainer publishing three slots on a Thursday should read "Thursday" once.

**No item cap here.** The dashboard caps each section at 5 because it is a summary; this screen is the "View all" destination, so capping it would leave the full list with nowhere to live. Requests are ordered oldest-asked first — the person who has waited longest is answered first.

## States

### ST-01 Default

- **When** the screen has loaded, the employee teaches at least one skill, and at least one section has content
- **Shows** the app-header; the page title; the two section cards in the order above, each with its title and count. Each request row carries the learner's name, the skill they want to learn, the slot's day, date, start time and duration, how long ago they asked, and **Approve** / **Reject**. Each availability row carries start time, duration and skill, plus its status — nothing, "N requests waiting", or "Booked — <learner>" — and the actions that status allows.
- **Can do** approve or reject any request (→ ST-16); add a time (→ ST-07); remove an unrequested time (→ ST-21); join or cancel a booked session; everything the header allows

### ST-02 Loading

- **When** the screen has mounted and the teaching data request is in flight
- **Shows** the app-header and page title in full — neither needs this data — and both section cards as placeholders: title visible without its count (the count is not yet known), body showing muted skeleton rows at the height the real rows occupy. The **Add a time** button renders but is disabled, because the form needs the trainer's skill list, which is part of this request.
- **Can do** use the header. Nothing inside a card is interactive.
- **Why skeletons rather than one page spinner:** consistent with SCR-004 and SCR-006 — the frame is known before the data lands, so the page does not visibly re-flow when it does.

### ST-03 Couldn't load

- **When** the teaching data request fails — timeout, server error, no connection
- **Shows** the app-header and title as normal and, in place of both cards, one error panel: alert icon + "We couldn't load your teaching page." + a **Try again** button. The header stays usable so the trainer is never trapped.
- **Can do** retry (→ ST-02); use the header
- **Why one error and not two:** the screen loads as a single request, so exactly one thing can fail. Two failure panels would specify a failure mode the design cannot produce.

### ST-04 You don't teach anything yet

- **When** the data loaded successfully and the employee has declared **no** teachable skill
- **Shows** both section cards replaced by a single panel: one line explaining that publishing times starts with saying what you can teach, and one primary action, **Add a skill you can teach**, pointing at SCR-005. No **Add a time** button anywhere on the screen.
- **Can do** follow that action; use the header
- **Why the whole screen and not an empty availability card:** a trainer with no teachable skill cannot publish a time _and_ cannot receive a request, so both sections are structurally empty rather than incidentally empty. Offering "Add a time" here would open a form whose first field has nothing in it.
- **Note** the moment they list a teachable skill they are in trainer search results (REQ-013), so this state is the only thing standing between them and a request arriving.

### ST-05 No requests waiting

- **When** the data loaded, the employee teaches something, and no request is pending — but they have published at least one time
- **Shows** the Requests card keeping its title (count reads 0) with its rows replaced by one line: "No requests waiting." No action — there is nothing the trainer can do to make someone ask.
- **Can do** everything else on the screen
- **Why the card stays:** removing it would change the screen's shape between visits, so the trainer could never learn where requests appear.

### ST-06 No times published yet

- **When** the data loaded, the employee teaches something, and they have published no times — or every one they published has passed
- **Shows** the Availability card keeping its title and its **Add a time** button, with its rows replaced by one line: "You haven't published any times yet. Colleagues can find you, but they can't book you." plus the same **Add a time** action repeated as a link. The sentence names the consequence, because "no times" is otherwise a fact without a stake.
- **Can do** add a time (→ ST-07)

### ST-07 Add a time — the form

- **When** the trainer activates **Add a time**
- **Shows** a dialog over the page, the page behind inert under a scrim, containing four fields in this order and a pair of actions:

  | Field  | Control                                                                          | Default        |
  | ------ | -------------------------------------------------------------------------------- | -------------- |
  | Skill  | Select, listing **only skills this employee has declared teachable** (see OQ-T3) | their only teachable skill if they have exactly one, otherwise empty |
  | Date   | Date field, today or later                                                       | empty          |
  | Start  | Time field                                                                       | empty          |
  | Length | Select — 30 / 45 / 60 / 90 minutes (see OQ-T2)                                   | 45 minutes     |

  Actions: **Cancel** (secondary) and **Publish** (primary). Publish is enabled only when all four fields hold a value.

- **Can do** fill the form; publish (→ ST-09); cancel or press Escape, which closes without saving and returns focus to the **Add a time** button
- **Why a dialog and not an inline row:** the form has four fields and a validation story of its own. Inline, it would push the whole list down every time it opened and share the page's error space with request errors.

### ST-08 Add a time — the form won't accept it

- **When** the trainer submits with a value the form can reject without asking the server: a date in the past, a start time already gone by today, or a time that overlaps one they have already published (see OQ-T4)
- **Shows** the dialog staying open with everything the trainer typed intact, the offending field marked with an icon **and** a message beneath it — never a red border alone:
  - date in the past → "Pick today or a later date."
  - start time gone by → "That time has already passed today."
  - overlapping an existing time → "You already have a time at 14:00 that day. A session is one-on-one, so times can't overlap."
  The message names the existing time, so the trainer knows which one to look at.
- **Can do** correct the field and publish again; cancel

### ST-09 Publishing

- **When** the trainer has submitted a valid form and the call is in flight
- **Shows** all four fields disabled at the values entered, **Cancel** disabled, and **Publish** showing a spinner in place of its label. The dialog cannot be dismissed while a write is in flight — closing it would leave the trainer unsure whether the time exists.
- **Can do** nothing until it resolves

### ST-10 Time published

- **When** the publish call succeeds (REQ-014)
- **Shows** the dialog closing; the new row appearing in **My available times** in its correct day group and time order, briefly carrying a tick icon and "Published" beside it; the section count incrementing; and focus moving to the new row so a keyboard user is left where the change happened. If the new time is the first ever, the card leaves ST-06.
- **Can do** everything ST-01 allows
- **Note** no page-level success banner — consistent with SCR-004 ST-07 and SCR-006 ST-15, confirmation belongs where the change landed. The time is bookable immediately; no approval step exists (REQ-013's model).

### ST-11 Couldn't publish

- **When** the publish call fails
- **Shows** the dialog staying open with every value intact, the fields and both buttons re-enabled, and an inline error above the actions: alert icon + "Couldn't publish that time. Please try again." The wording blames the saving, never the time itself, so nobody reads it as a refusal.
- **Can do** publish again; cancel
- **Special case** if the failure is specifically that the same time was published from somewhere else in the meantime, the message becomes "You already have a time at 14:00 that day." and the dialog offers **Close** rather than a retry that will fail identically.

### ST-12 Add a time — at phone width

- **When** any of ST-07 to ST-11 is reached below the desktop breakpoint (NFR-004)
- **Shows** the same form as a full-height sheet rising from the bottom of the screen rather than a centred dialog: fields at full width, one per line, and the **Publish** / **Cancel** pair pinned to the bottom of the sheet so they are reachable without scrolling past the fields.
- **Can do** everything ST-07 to ST-11 allow at that step
- **Why a sheet:** matches SCR-006's trainer panel at phone width, so the portal has one way of showing an overlay on a phone rather than two.

### ST-13 A time with requests waiting on it

- **When** one or more learners have requested a published, unbooked time (REQ-015)
- **Shows** that availability row carrying an hourglass icon and "1 request waiting" / "N requests waiting" as a text label, linking up to the Requests section above. Its **Remove** action is **not** offered — see the decisions table.
- **Can do** follow the label to the request(s); nothing destructive
- **Why Remove disappears rather than greying out:** a disabled button invites a click and then explains itself. There is a defined path — answer the request — and the label points at it.

### ST-14 A booked time

- **When** the trainer approved a request for that time and the session is confirmed and still in the future (REQ-017, REQ-021)
- **Shows** that row with a tick icon and "Booked — Arun K." as a text label, the learner's name spelled out; a **Join meeting** action (REQ-019); and **Cancel session** as a quiet, non-primary action (REQ-020). No Remove — a booked time belongs to two people now, and removing it silently is not one of the things REQ-020 permits.
- **Can do** join the meeting; start a cancellation (→ OQ-T5); nothing else
- **Note** exactly one learner appears here, ever (REQ-021, BR-003.1). There is no design for a second name because the system does not produce one.

### ST-15 A time that has passed

- **When** a published time's start has gone by
- **Shows**
  - **unbooked** — the row is dropped from the list entirely, and the section count reflects that. An expired offer is not a fact the trainer needs to act on, and keeping it would grow the list without bound.
  - **booked** — the row stays, moved into a "Finished" group beneath the upcoming days, in muted text with a clock icon and "Session finished — awaiting completion". No action is offered on it yet.
- **Can do** nothing on either
- **Note** "awaiting completion" is deliberately a status and not a button: REQ-022 gives the trainer a way to mark a session complete, and that flow is not designed. Naming the state here without inventing its action is the honest version. Raised as OQ-T6.

### ST-16 Responding to a request

- **When** the trainer has activated Approve or Reject and the call is in flight (REQ-017)
- **Shows** that request row only: both its buttons disabled, the clicked one showing a spinner in place of its label. If other requests exist **for the same time**, theirs are disabled too for the duration — see ST-19. Every unrelated row and the whole availability section stay fully interactive.
- **Can do** nothing on the affected rows; everything else on the screen still works

### ST-17 Response recorded

- **When** the approve or reject call succeeds
- **Shows** the row replaced in place by a confirmation line — tick icon + "Approved — Priya has been notified." or "Rejected — Priya has been notified." (REQ-028) — held for a few seconds, after which the row leaves the list and the count decrements. On approval the matching availability row below becomes ST-14 in the same beat, so the trainer sees where the decision landed.
- **Can do** everything ST-01 allows

### ST-18 Response failed

- **When** the approve or reject call fails
- **Shows** the row restored to its interactive form with an inline error beneath it: alert icon + "Couldn't save that. Please try again." Both buttons re-enabled. The wording says the _saving_ failed, so nobody reads it as the request being invalid.
- **Can do** retry the same action, or take the other one

### ST-19 Two requests for the same time

- **When** more than one learner has a pending request against a single published time — possible because SCR-006 lets any learner request any unbooked slot, while REQ-021 allows exactly one booking
- **Shows** those requests kept adjacent in the list regardless of ask-order, under a shared notice: warning icon + "2 people asked for Thu 4 Sep, 14:00. Approving one declines the other." Each row keeps its own Approve and Reject. On approving one: that row goes to ST-17, and the sibling rows are replaced by "Declined — the time was given to Priya. Arun has been notified." rather than vanishing.
- **Can do** approve one; reject any individually; the shared notice is not itself actionable
- **Blocked on** whether the losing requests auto-decline at all is a business rule nobody has stated. This is conflict **C-1** below, and the behaviour described here is the working default, not a decision I get to make.

### ST-20 A request that expired

- **When** a pending request passes the auto-reject cutoff before the trainer answers it (REQ-018)
- **Shows** the row in muted text with a clock icon and "Expired — Priya was told you didn't get to this in time.", its Approve and Reject removed rather than disabled. It stays for the rest of the visit and is gone on the next load. If the trainer clicks Approve at the moment it expires, ST-18's error is replaced by this row and the line "This request expired while you were looking at it." — a different fact deserves a different sentence.
- **Can do** nothing on the row
- **Why it is shown rather than silently removed:** a request disappearing between the trainer reading it and reaching for it is the confusing case, and the wording tells them the colleague already knows. Same reasoning as SCR-006 ST-13.

### ST-21 Removing a published time

- **When** the trainer activates **Remove** on a published time that nobody has requested
- **Shows** the row's actions replaced in place by a confirm pair — "Remove this time?" with **Remove** (danger emphasis) and **Keep** — rather than a modal dialog, since nothing is lost that a colleague is depending on. On confirm: the button shows a spinner, then the row fades out and the count decrements. On failure: the row is restored with "Couldn't remove that time. Please try again."
- **Can do** confirm, or keep and return the row to ST-01
- **Note** whether a trainer may unpublish at all is not stated by any requirement — REQ-014 says publish and stops. Working default is yes-while-unrequested; raised as OQ-T1.

## Components

| Component          | Preview                                                       | States covered                                                          |
| ------------------ | ------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `availability-row` | `inception/design/components/availability-row/preview.html`   | published, just-published, requests waiting, booked, finished, removing — ST-01, ST-10, ST-13, ST-14, ST-15, ST-21 |
| `slot-form`        | `inception/design/components/slot-form/preview.html`          | dialog, field errors, publishing, failed, phone sheet — ST-07, ST-08, ST-09, ST-11, ST-12 |
| `request-row`      | `inception/design/components/request-row/preview.html`        | responding, recorded, failed, competing pair, expired — ST-16, ST-17, ST-18, ST-19, ST-20 |
| `card`             | `inception/design/components/card/preview.html`               | loading skeleton, empty section — ST-02, ST-05                          |
| `empty-state`      | `inception/design/components/empty-state/preview.html`        | not a trainer yet, no times published — ST-04, ST-06                    |
| `alert`            | `inception/design/components/alert/preview.html`              | the page-level load failure — ST-03                                     |
| `app-header`       | `inception/design/components/app-header/preview.html`         | the shell inherited from SCR-004, with "My teaching" added as a destination |
| `skill-chip`       | `inception/design/components/skill-chip/preview.html`         | the skill named on an availability row                                  |
| `button`           | `inception/design/components/button/preview.html`             | primary, secondary, danger, loading, disabled                           |
| `spinner`          | `inception/design/components/spinner/preview.html`            | the in-flight indicator behind ST-09, ST-16 and ST-21                   |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Keyboard:** header tab order unchanged from SCR-004. Then the page: **Add a time** is reached before the availability rows, since it is the section's primary action. Within a request row, Approve precedes Reject in both visual and tab order. Within an availability row, the actions follow the status text.
- **Dialog (ST-07 to ST-12):** focus moves to the Skill field on open and is trapped inside until it closes. Escape cancels — except during ST-09, where the write is in flight and Escape does nothing. On close, focus returns to the control that opened it.
- **Focus after an action:** ST-10 moves focus to the new row; ST-17 moves focus to the Requests section heading once the row leaves; ST-21 moves focus to the Availability section heading. Focus is never dropped to `<body>`.
- **Focus ring:** every interactive element shows the `--focus-ring` outline on keyboard focus, including whole-row targets.
- **Non-colour signalling (BRD §5, NFR-004's sibling commitment carried from SCR-004):** "Booked", "N requests waiting", "Expired", "Session finished", "Published" and every field error each carry an icon **and** a word. Nothing on this screen is distinguishable by colour alone — including the danger emphasis on Remove, which also carries the word "Remove" and a confirm step.
- **Announcements:** each section card is a landmark region labelled by its heading. ST-02's skeleton region is `aria-busy="true"`. The dialog is `role="dialog"` `aria-modal="true"` labelled by its title. ST-17's confirmation, ST-10's "Published" and ST-21's removal are announced `aria-live="polite"` — the trainer caused them and is looking at them. ST-03's panel and ST-19's shared notice are `aria-live="assertive"`, the latter because it changes what the trainer's next click will do.
- **Time and date:** every time is shown as day, date and clock time in the trainer's local zone, never as a bare relative offset — a trainer publishing availability needs the actual clock time. "asked 2 hours ago" on a request row is the one relative reading, and it is supplementary to the slot's absolute time, not a replacement for it.
- **Responsive (NFR-004):** the single column reflows to full width below the breakpoint; request-row actions move from beside the text to a full-width pair beneath it; the dialog becomes a sheet (ST-12); day subheadings become sticky while scrolling a long list.
- **Motion:** ST-17's and ST-21's row removals are fades, suppressed under `prefers-reduced-motion`.

## Structural decisions

| Decision                                                                                  | Rationale                                                                                                                                                                                                                                 | Alternative rejected                                                                                                                                                     |
| ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requests and availability on one screen, not two                                          | Designer's decision, 2026-08-28. At POC scale both lists are short, they reference each other constantly (a request _is_ a request for one of these times), and the dashboard already routes to both. One destination in the top bar, not two | Two destinations — cleaner conceptual split, but it puts a navigation step between a request and the time it is about, which is the one thing a trainer needs to see together |
| Times are added through a four-field form, listed by day — not drawn on a week grid       | Designer's decision, 2026-08-28. One layout that works identically on a phone and a monitor, which is what NFR-004 asks for, and far less to build for a POC                                                                              | A Mon–Sun calendar grid — better for thinking about a week, but it needs a second design for phone width; and a grid-plus-list hybrid, which is two of everything          |
| Requests section above availability                                                       | One section holds a decision a colleague is blocked on; the other holds a list only the trainer cares about. Same priority logic as SCR-004's section order                                                                               | Availability first, on the argument that it is the cause and requests are the effect — true, and irrelevant to what needs attention now                                   |
| No item cap on either section                                                             | This screen is the dashboard's "View all" destination. Capping it would leave the complete list with nowhere to live                                                                                                                      | Matching SCR-004's cap of 5 — which would mean a trainer with eight published times could not see them anywhere                                                           |
| Availability rows grouped under a day subheading                                          | A trainer publishing three times on Thursday reads "Thursday" once instead of three times, and the eye finds a day before it finds an hour                                                                                                | A flat list repeating the full date on every row                                                                                                                          |
| The skill picker lists only skills the trainer has declared teachable                     | Offering a time to teach something you have not said you can teach makes you invisible in the search that would surface it (REQ-012, REQ-013) — the slot could never be found. Restricting the field is the design following the data, not a new rule | A free skill picker over the whole IT-maintained catalog — publishable, unfindable                                                                                       |
| An unbooked time that has passed leaves the list; a booked one stays under "Finished"     | An expired offer needs no action and would grow the list without bound. A finished session does need action eventually (REQ-022), so it stays visible even though its action is not yet designed                                          | Keeping everything with a "Past" filter — a control that exists to hide rows that need not have been kept                                                                 |
| Remove is withdrawn from a time once someone has requested it (ST-13), not greyed out      | A disabled control invites a click and then explains itself. There is a real path — answer the request — so the row points at it instead                                                                                                  | A disabled Remove with a tooltip; or allowing removal and silently killing the pending request, which decides C-1 by accident                                             |
| Remove confirms in the row (ST-21), not in a modal                                        | Nothing another person depends on is lost — by definition, since Remove is only offered on unrequested times. An in-row confirm is enough friction for a reversible act                                                                    | A modal confirm dialog, which is the weight this action would carry if it could affect a colleague — and it cannot                                                        |
| Competing requests for one time are shown together with a shared notice (ST-19)           | REQ-021 means only one can win. A trainer who approves the second of two without knowing the first existed has made a decision they did not know they were making                                                                         | Showing them in plain ask-order like any other requests, and letting the system resolve the loser silently                                                                |
| An expired request is shown and explained (ST-20) rather than removed on sight            | A row vanishing between reading and clicking is the confusing case; and the trainer should know the colleague was already told. Same reasoning as SCR-006 ST-13                                                                           | Dropping expired requests at load, which makes REQ-018 invisible to the person whose inaction triggered it                                                                |
| "Session finished — awaiting completion" is a status, not a button                        | REQ-022 exists but its flow is not designed. Naming the state without inventing its action keeps the screen honest and leaves the completion design free                                                                                  | Putting a "Mark complete" button here and designing REQ-022 in passing                                                                                                    |

## Conflicts

Blocks approval of this screen until resolved.

| #   | Conflict                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Between                       | Owner      | Status |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- | ---------- | ------ |
| C-1 | **Nothing says what happens to the other requests for a time once one is approved.** REQ-015 lets any learner request any published, unbooked slot, so a popular time can collect several pending requests. REQ-021 and BR-003.1 allow exactly one booking. REQ-017 gives the trainer approve and reject. No requirement says whether approving one request auto-rejects its rivals, or leaves them pending against a slot they can no longer win. BR-003.1's wording — "when a learner **books** a slot, no other learner may book that same slot" — does not distinguish requesting from confirming, which is precisely the distinction needed here. This decides what a colleague is told, so it is the BA's to settle, not mine. **Working default pending that decision:** approving one request auto-declines the others for that time, each replaced by "Declined — the time was given to Priya. Arun has been notified." (ST-19), and the trainer is warned before they choose. | REQ-015 ↔ REQ-017 ↔ REQ-021 | BA (`/ba`) | open   |

## Open questions

Each has a working default, so none blocks drawing the frames — but each is someone else's call to confirm, and the answer may change a rule or a line of copy.

| #     | Question                                                                                                                                                            | Working default                                                                                          | Owner          | Status                     |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | -------------- | -------------------------- |
| OQ-T1 | May a trainer unpublish a time? REQ-014 says publish and stops                                                                                                      | Yes while nobody has requested it (ST-21); never once requested or booked                                | BA (`/ba`)     | open                       |
| OQ-T2 | Session length — is it fixed by the portal, or chosen per time by the trainer? SCR-006 raised the same gap from the learner's side (OQ-S7)                          | Trainer chooses per time, from 30 / 45 / 60 / 90 minutes                                                 | BA (`/ba`)     | open — paired with OQ-S7   |
| OQ-T3 | Can a trainer publish a time for a skill they have not declared teachable?                                                                                          | No — the picker lists only their teachable skills                                                        | BA (`/ba`)     | open                       |
| OQ-T4 | May two of a trainer's own published times overlap? Sessions are one-on-one (REQ-021), so a trainer cannot be in both                                               | Blocked at the form with an explanatory message (ST-08)                                                  | BA (`/ba`)     | open                       |
| OQ-T5 | REQ-020 lets either party cancel a confirmed session. This screen offers the trainer's entry point (ST-14) but the flow, its copy and its notification are undesigned | "Cancel session" opens nothing yet; the flow is designed with the learner's bookings screen              | BA + Designer  | open                       |
| OQ-T6 | REQ-022 lets a trainer mark a session complete. Is this the place they do it?                                                                                       | ST-15 names the state and offers no action; the completion flow is designed separately                   | Designer + BA  | open — your call at review |
| OQ-T7 | Should rejecting a request require or offer a reason?                                                                                                               | No reason field — REQ-017 asks for a decision, not an explanation                                        | BA (`/ba`)     | open                       |
| OQ-T8 | How far ahead may a trainer publish?                                                                                                                                | No limit; the date field rejects only dates already past                                                 | BA (`/ba`)     | open                       |
| OQ-T9 | BRD OQ-1 (the auto-reject cutoff) is unresolved, and it decides whether a pending request should warn the trainer that it is about to expire                        | No countdown until BRD OQ-1 is answered; a request reads "asked 2 hours ago" and nothing more            | Business owner | open (tracked as BRD OQ-1) |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist. ST-01 is worth two frames — desktop and phone width — since NFR-004 is what the reflow has to prove, and ST-12 is the phone form of the whole ST-07…ST-11 family rather than a frame of its own.

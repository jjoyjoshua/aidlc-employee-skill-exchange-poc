# SCR-007 — My teaching

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-007/ST-## -->`.

|                  |                                                                                                     |
| ---------------- | ----------------------------------------------------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)                                                   |
| **Traces to**    | REQ-014, REQ-017, REQ-021, REQ-022, REQ-027, NFR-004                                                |
| **Surface**      | `apps/ui` `features/teaching` — `/teaching`, reached from "My teaching" in the app-header (SCR-004) |
| **Primary user** | Employee acting as a **Trainer** — offering their time and deciding who gets it                     |
| **Status**       | draft — awaiting designer review                                                                    |

## Purpose

The trainer's half of the portal, end to end. This is where an employee says _when I am free to teach_ (REQ-014), answers the requests that arrive against those times (REQ-017) — including the case where two colleagues have asked for the same one and only one can have it (REQ-021) — and then, once a session is over, confirms that it happened (REQ-022) and keeps the record of who they taught and how it was received (REQ-027).

"Done" for the trainer is one of three things, depending on when they arrive: nobody is waiting on them and their calendar shows the times they meant to offer; they have cleared the decisions that were waiting, each one recorded and the colleague notified, without leaving the page; or they have closed the loop on a session that happened — confirmed, and the learner's certificate on its way.

The past lives on this screen rather than on a separate Teaching History destination because the designer chose one destination per role over a separate filing system (2026-08-28, recorded in the decisions table). It is the same choice SCR-008 made for the learner, and it settles that screen's OQ-L10: there is no History destination in the top bar.

This screen is also where the dashboard's "View all" on Booking requests lands. SCR-004 handles the first five requests inline because that is the fast path; this is the full list, plus the availability that produces it.

### Requirements this screen touches but does not fully serve

Recorded so a later audit doesn't read the manifest as "covered":

| Requirement                            | What this screen does                                                                    | Where the rest lives                                                        |
| -------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| REQ-018 (auto-reject stale requests)   | Shows the trainer's side of it — a request that expired before they answered (ST-20)     | The cutoff itself and the learner's notification — BRD OQ-1, not yet designed |
| REQ-019 (video-meeting link)           | Surfaces the link as "Join meeting" on a booked time (ST-14)                             | Link generation and a session detail view — not yet designed                 |
| REQ-020 (either party cancels)         | Puts "Cancel session" on a booked time, and stops at the confirm step — see OQ-T5         | The cancellation flow, its copy and its notification — not yet designed      |
| REQ-023 (learner rates the session)    | **Fires** it — marking a session complete is what prompts the learner (ST-25) — and shows the rating that comes back (ST-28) | The rating form itself is SCR-008 ST-22; the trainer never rates and never replies |
| REQ-024 (certificate)                  | **Fires** it — the same action generates the learner's certificate with no further step (ST-25) | The certificate belongs to the learner and is opened from SCR-008; no copy is offered here |
| REQ-025 (trainer's average rating)     | Nothing — a taught row carries one session's rating, not an average                       | SCR-005's profile and SCR-006's results. Whether the trainer sees their own average here is OQ-T12 |
| REQ-028 (in-app notifications)         | Every approve, reject, publish and completion states that the colleague has been notified | The notification surface itself — not yet designed                           |

## Layout

The app shell from SCR-004, then a single constrained centre column holding three section cards, in this order:

1. **Requests waiting for you** — someone else is blocked on this
2. **My available times** — the trainer's own calendar
3. **Sessions you've taught** — what became of those times, and the one action a finished session still carries

Requests come first for the same reason they outrank skills on the dashboard: the first section holds a decision another person is blocked on, the second holds a list only the trainer cares about, the third holds the past. Blocked-on-me, then mine, then done. One column, identical order at every width (NFR-004).

```
┌──────────────────────────────────────────────────────────┐
│ SkillEx   [ main navigation ]                  🔔3  (JJ)▾│ ← app-header
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
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Sessions you've taught (4)                     │     │
│   │                  1 waiting for you to confirm  │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Priya M. · Excel · Thu 4 Sep               │ │     │
│   │ │ Did this happen?     [ Mark as complete ]  │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Arun K. · SQL · 8 Aug                      │ │     │
│   │ │ ★★★★☆ 4 out of 5 · "Very clear."           │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Meera S. · Python · 29 Jul                 │ │     │
│   │ │ — No rating left                           │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

**Times are grouped by day**, soonest first, with the day as a subheading rather than repeated on every row. A trainer publishing three slots on a Thursday should read "Thursday" once.

**Taught sessions are listed flat, most recent first** — no day grouping, because rows there rarely share a date and a subheading over a single row is noise. One exception to the ordering: a session still waiting to be confirmed sorts above the confirmed ones regardless of date, since it is the only row in that card carrying an action. The card header states the count — "1 waiting for you to confirm" — so the exception is announced rather than merely performed. SCR-008's Finished card makes the same exception for an unrated session, for the same reason.

**No item cap here.** The dashboard caps each section at 5 because it is a summary; this screen is the "View all" destination, so capping it would leave the full list with nowhere to live. Requests are ordered oldest-asked first — the person who has waited longest is answered first. Sessions you've taught is uncapped only because the POC is under 10 employees (BRD §1): that is a scale assumption, not a principle, and it is the first section on this screen that will need paging when the assumption breaks.

## States

### ST-01 Default

- **When** the screen has loaded, the employee teaches at least one skill, and at least one section has content
- **Shows** the app-header; the page title; the three section cards in the order above, each with its title and count. Each request row carries the learner's name, the skill they want to learn, the slot's day, date, start time and duration, how long ago they asked, and **Approve** / **Reject**. Each availability row carries start time, duration and skill, plus its status — nothing, "N requests waiting", or "Booked — <learner>" — and the actions that status allows. Each taught row carries the learner's name, the skill taught and the date it happened (REQ-027), plus either the rating that came back, the words "No rating left", or — where the session has not been confirmed yet — "Did this happen?" and **Mark as complete**.
- **Can do** approve or reject any request (→ ST-16); add a time (→ ST-07); remove an unrequested time (→ ST-21); join or cancel a booked session; confirm a session that has happened (→ ST-23); everything the header allows

### ST-02 Loading

- **When** the screen has mounted and the teaching data request is in flight
- **Shows** the app-header and page title in full — neither needs this data — and all three section cards as placeholders: title visible without its count (the count is not yet known), body showing muted skeleton rows at the height the real rows occupy. The **Add a time** button renders but is disabled, because the form needs the trainer's skill list, which is part of this request.
- **Can do** use the header. Nothing inside a card is interactive.
- **Why skeletons rather than one page spinner:** consistent with SCR-004 and SCR-006 — the frame is known before the data lands, so the page does not visibly re-flow when it does.

### ST-03 Couldn't load

- **When** the teaching data request fails — timeout, server error, no connection
- **Shows** the app-header and title as normal and, in place of all three cards, one error panel: alert icon + "We couldn't load your teaching page." + a **Try again** button. The header stays usable so the trainer is never trapped.
- **Can do** retry (→ ST-02); use the header
- **Why one error and not three:** the screen loads as a single request, so exactly one thing can fail. Three failure panels would specify a failure mode the design cannot produce.

### ST-04 You don't teach anything yet

- **When** the data loaded successfully and the employee has declared **no** teachable skill
- **Shows** all three section cards replaced by a single panel: one line explaining that publishing times starts with saying what you can teach, and one primary action, **Add a skill you can teach**, pointing at SCR-005. No **Add a time** button anywhere on the screen.
- **Can do** follow that action; use the header
- **Why the whole screen and not an empty availability card:** a trainer with no teachable skill cannot publish a time, cannot receive a request, and cannot have taught anything — so all three sections are structurally empty rather than incidentally empty. Offering "Add a time" here would open a form whose first field has nothing in it.
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
  - **unbooked** — the row is dropped from the availability list entirely, and the section count reflects that. An expired offer is not a fact the trainer needs to act on, and keeping it would grow the list without bound.
  - **booked** — the row leaves the availability list too, and reappears in **Sessions you've taught** as a session waiting to be confirmed (→ ST-22). The availability card holds what is still on offer; the third card holds what became of it.
- **Can do** nothing in the availability card in either case. The booked one carries its action in its new home, and only there
- **Note** this replaces the "Finished" group that used to sit at the foot of the availability card. A session that has already happened is not a time on offer, and REQ-022 gives it an action that belongs beside the record it produces rather than beside next Thursday's free hour. The learner is looking at the same moment from the other side, as SCR-008 ST-15.

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

### ST-22 A session waiting for you to confirm

- **When** a confirmed session's start time has passed and the trainer has not yet marked it complete (REQ-022)
- **Shows** the row at the top of **Sessions you've taught**, above the confirmed ones regardless of date, carrying the learner's name, the skill taught, and the day and date it happened. In place of a rating: a clock icon and "Did this happen?", with **Mark as complete** as the row's action. The card header carries the count — "1 waiting for you to confirm" — so the sort is announced rather than merely performed.
- **Can do** mark it complete (→ ST-23); nothing else
- **Note** this is the same moment SCR-008 ST-15 shows the learner as "Waiting for Priya to confirm this happened." The learner has no action there, so this row is the only place the state can be resolved. What happens if the trainer never acts — or if the session did not in fact happen — is **conflict C-2**, and the copy here promises nothing about either.

### ST-23 Marking complete — the confirm step

- **When** the trainer activates **Mark as complete**
- **Shows** the row's action replaced in place by a confirm pair — "Mark this as complete? Priya gets her certificate and a chance to rate the session." with **Yes, it happened** and **Not yet** — rather than a modal dialog. Same in-row treatment, and the same visual weight, as ST-21's Remove confirm.
- **Can do** confirm (→ ST-24); back out, returning the row to ST-22
- **Why a confirm at all, and why not a modal:** nothing is taken away — the action _gives_ a colleague a certificate (REQ-024) and a prompt to rate (REQ-023) — so the weight a modal implies is wrong here. But no requirement offers an undo (OQ-T10), and one line of copy naming both consequences before the trainer commits is the cheapest possible guard against an irreversible click.
- **Why the copy names the certificate and the rating:** they are the two things that happen to somebody else. BR-005.1 makes the certificate automatic and BR-004.1 makes the rating optional, so neither is a promise the trainer has to keep — but both are consequences they should know they are causing.

### ST-24 Marking complete — in flight

- **When** the trainer has confirmed and the call is in flight
- **Shows** that row only: **Not yet** disabled, and **Yes, it happened** showing a spinner in place of its label. Every other row, and both sections above, stay fully interactive.
- **Can do** nothing on the affected row; everything else on the screen still works

### ST-25 Marked complete

- **When** the call succeeds (REQ-022)
- **Shows** the row replaced in place by a confirmation line — tick icon + "Marked complete — Priya has her certificate and can rate the session." (REQ-024, REQ-023, REQ-028) — held for a few seconds, after which the row settles into its no-rating-yet form (ST-27) and re-sorts into date order among the confirmed sessions. The header's "waiting for you to confirm" count decrements, and disappears entirely when it reaches zero. Focus stays on the row.
- **Can do** everything ST-01 allows
- **Note** no page-level success banner — consistent with ST-10, ST-17, SCR-006 ST-15 and SCR-008 ST-24: confirmation belongs where the change landed.

### ST-26 Couldn't mark it complete

- **When** the call fails
- **Shows** the row restored to its confirm-step form (ST-23) with both controls re-enabled, and an inline error beneath it: alert icon + "Couldn't save that. Please try again." The wording says the _saving_ failed, so nobody reads it as the session being ineligible.
- **Can do** confirm again; back out

### ST-27 A taught session with no rating yet

- **When** the session is complete and the learner has not rated it — which REQ-023 permits indefinitely, since submitting is optional (BR-004.1)
- **Shows** the row with the learner's name, the skill taught and the date it happened (REQ-027), and in place of stars a muted line reading "No rating left". No stars, no zero, no prompt. No action of any kind.
- **Can do** nothing
- **Why words and not five empty stars:** on the learner's screen five empty stars _are_ the control that invites a rating (SCR-008 ST-21). Reusing that glyph here would render a control that does nothing, on a screen where the trainer cannot rate themselves — and an empty star row reads as "rated zero", which is a score the learner never gave.
- **Why no nudge:** there is nothing for the trainer to click that would produce a rating, and asking a colleague to rate you is not a thing this portal does. BR-004.1 makes the silence a legitimate answer, so the copy states it flatly rather than treating it as an omission.

### ST-28 A taught session the learner rated

- **When** the learner submitted a rating for a completed session (REQ-027 — "rating received")
- **Shows** the row with the learner's name, the skill taught, the date, the stars filled to the submitted value with that value spelled out beside them ("4 out of 5"), and the learner's comment beneath in quotation marks where one was left. Read-only: no reply, no dispute, no acknowledgement.
- **Can do** nothing
- **Note** whether the written comment reaches the trainer at all, and whether it is attributed by name, is **OQ-T11** — SCR-008 raised the same gap from the learner's side as OQ-L7. It is rendered here on that working default, not on a decision anyone has made. If the answer is "anonymous", this row loses the name above the comment and nothing else changes.
- **Note** no certificate action. REQ-024 generates the certificate for the _learner_; nothing in the BRD gives the trainer a copy, and inventing one here would invent a document.

### ST-29 Nothing taught yet

- **When** the data loaded, the employee teaches at least one skill, and no session of theirs has finished or been confirmed
- **Shows** the Sessions you've taught card keeping its title (count reads 0) with its rows replaced by one line: "No sessions taught yet. Once a session you've hosted is over, it'll appear here for you to confirm." No action — nothing the trainer can click creates a session.
- **Can do** everything else on the screen
- **Why the card holds its place:** same reasoning as ST-05 — a trainer learns where finished sessions will appear before they have one, and the sentence tells them what will put a row there.

## Components

| Component          | Preview                                                       | States covered                                                          |
| ------------------ | ------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `availability-row` | `inception/design/components/availability-row/preview.html`   | published, just-published, requests waiting, booked, finished, removing — ST-01, ST-10, ST-13, ST-14, ST-15, ST-21 |
| `slot-form`        | `inception/design/components/slot-form/preview.html`          | dialog, field errors, publishing, failed, phone sheet — ST-07, ST-08, ST-09, ST-11, ST-12 |
| `request-row`      | `inception/design/components/request-row/preview.html`        | responding, recorded, failed, competing pair, expired — ST-16, ST-17, ST-18, ST-19, ST-20 |
| `card`             | `inception/design/components/card/preview.html`               | loading skeleton, empty sections — ST-02, ST-05, ST-29                  |
| `empty-state`      | `inception/design/components/empty-state/preview.html`        | not a trainer yet, no times published — ST-04, ST-06                    |
| `alert`            | `inception/design/components/alert/preview.html`              | the page-level load failure — ST-03                                     |
| `app-header`       | `inception/design/components/app-header/preview.html`         | the shell inherited from SCR-004, with "My teaching" added as a destination |
| `skill-chip`       | `inception/design/components/skill-chip/preview.html`         | the skill named on an availability row                                  |
| `button`           | `inception/design/components/button/preview.html`             | primary, secondary, danger, loading, disabled                           |
| `history-row`      | `inception/design/components/history-row/preview.html`        | waiting to be confirmed, confirming, confirmed, no rating left, rated — ST-22, ST-23, ST-24, ST-25, ST-26, ST-27, ST-28 |
| `rating-stars`     | `inception/design/components/rating-stars/preview.html`       | the read-only star display reused inside a rated taught row             |
| `spinner`          | `inception/design/components/spinner/preview.html`            | the in-flight indicator behind ST-09, ST-16, ST-21 and ST-24            |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Keyboard:** header tab order unchanged from SCR-004. Then the page: **Add a time** is reached before the availability rows, since it is the section's primary action. Within a request row, Approve precedes Reject in both visual and tab order. Within an availability row, the actions follow the status text. Within a taught row, **Mark as complete** is the only interactive element; its confirm pair (ST-23) replaces it in place, so nothing before or after the row shifts in the tab order.
- **Dialog (ST-07 to ST-12):** focus moves to the Skill field on open and is trapped inside until it closes. Escape cancels — except during ST-09, where the write is in flight and Escape does nothing. On close, focus returns to the control that opened it.
- **Focus after an action:** ST-10 moves focus to the new row; ST-17 moves focus to the Requests section heading once the row leaves; ST-21 moves focus to the Availability section heading. Focus is never dropped to `<body>`. ST-25 keeps focus on the row it happened to, since the row survives the action and changes meaning rather than leaving.
- **Focus ring:** every interactive element shows the `--focus-ring` outline on keyboard focus, including whole-row targets.
- **Non-colour signalling (BRD §5, NFR-004's sibling commitment carried from SCR-004):** "Booked", "N requests waiting", "Expired", "Published", "Did this happen?", "No rating left", "Marked complete" and every field error each carry an icon **and** a word. Nothing on this screen is distinguishable by colour alone — including the danger emphasis on Remove, which also carries the word "Remove" and a confirm step. A rating received is always accompanied by its numeric value in text, so a taught row is legible with no colour and no glyph rendering at all.
- **Announcements:** each section card is a landmark region labelled by its heading. ST-02's skeleton region is `aria-busy="true"`. The dialog is `role="dialog"` `aria-modal="true"` labelled by its title. ST-17's confirmation, ST-10's "Published" and ST-21's removal are announced `aria-live="polite"` — the trainer caused them and is looking at them. ST-03's panel and ST-19's shared notice are `aria-live="assertive"`, the latter because it changes what the trainer's next click will do. ST-25's "Marked complete" line is announced `aria-live="polite"` for the same reason as ST-17's: the trainer caused it and is looking at it.
- **Time and date:** every time is shown as day, date and clock time in the trainer's local zone, never as a bare relative offset — a trainer publishing availability needs the actual clock time. "asked 2 hours ago" on a request row is the one relative reading, and it is supplementary to the slot's absolute time, not a replacement for it. A session in Sessions you've taught shows its date only — the clock time of something that is over is noise, which is the rule SCR-008's Finished card already follows.
- **Responsive (NFR-004):** the single column reflows to full width below the breakpoint; request-row actions move from beside the text to a full-width pair beneath it; the dialog becomes a sheet (ST-12); day subheadings become sticky while scrolling a long list. a taught row's **Mark as complete**, and the confirm pair that replaces it, move from beside the text to full width beneath it, the safe action (**Not yet**) first;
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
| An unbooked time that has passed leaves the availability list; a booked one moves to Sessions you've taught | An expired offer needs no action and would grow the list without bound. A finished session does need an action (REQ-022) and produces a record (REQ-027), and both belong beside the history they create rather than inside the calendar of what is still on offer | Keeping everything with a "Past" filter — a control that exists to hide rows that need not have been kept; or leaving finished sessions at the foot of the availability card, which puts an action about last Tuesday under the heading "My available times" |
| Remove is withdrawn from a time once someone has requested it (ST-13), not greyed out      | A disabled control invites a click and then explains itself. There is a real path — answer the request — so the row points at it instead                                                                                                  | A disabled Remove with a tooltip; or allowing removal and silently killing the pending request, which decides C-1 by accident                                             |
| Remove confirms in the row (ST-21), not in a modal                                        | Nothing another person depends on is lost — by definition, since Remove is only offered on unrequested times. An in-row confirm is enough friction for a reversible act                                                                    | A modal confirm dialog, which is the weight this action would carry if it could affect a colleague — and it cannot                                                        |
| Competing requests for one time are shown together with a shared notice (ST-19)           | REQ-021 means only one can win. A trainer who approves the second of two without knowing the first existed has made a decision they did not know they were making                                                                         | Showing them in plain ask-order like any other requests, and letting the system resolve the loser silently                                                                |
| An expired request is shown and explained (ST-20) rather than removed on sight            | A row vanishing between reading and clicking is the confusing case; and the trainer should know the colleague was already told. Same reasoning as SCR-006 ST-13                                                                           | Dropping expired requests at load, which makes REQ-018 invisible to the person whose inaction triggered it                                                                |
| Teaching history is a third section on this screen, not a destination of its own            | Designer's decision, 2026-08-28. A trainer's question is "what happened with Priya?", and the answer moves from requested to booked to taught without them meaning to change screens. One destination per role, mirroring the choice SCR-008 made for the learner — and it settles SCR-008's OQ-L10: there is no History destination in the top bar | A separate History destination holding both teaching and learning history — a cleaner filing system, and it splits one session across two screens for each of the two people who were in it; also a Teaching History page of its own, which would still need a "Mark as complete" button back on this screen |
| "Awaiting completion" became an action (ST-22), replacing the bare status this screen shipped with | Designer's decision, 2026-08-28, closing OQ-T6. The status was honest while REQ-022's flow was undesigned; leaving it there once the flow exists would strand the learner, whose certificate and rating both wait on this one click                                                    | Keeping the status and putting the action somewhere else — which is the arrangement that produced SCR-008's C-2 in the first place |
| Marking complete confirms in the row (ST-23), not in a modal                                | The action gives rather than takes: a certificate (REQ-024) and a rating prompt (REQ-023) for the learner. A modal is the weight this portal reserves for taking a committed time away from a colleague (SCR-008 ST-10). But it is irreversible (OQ-T10), so a confirm step naming both consequences earns its place | No confirm at all, on the grounds that nothing is destroyed — true, and it makes an irreversible act a single stray click; or a modal, which would rank confirming a session that already happened above cancelling one that has not |
| **Mark as complete** appears only once the session's start time has passed                  | REQ-022 says "after it has occurred". A button offered before then invites a trainer to certify something that has not happened, and BR-005.1 would issue a real certificate for it                                                                                                     | Offering it on any booked session, and validating the timing server-side only — which puts the error message where the design could have prevented it |
| A taught row shows no certificate action                                                    | REQ-024 generates the certificate for the learner and names them on it. Nothing in the BRD gives the trainer a copy, so offering one would invent a document                                                                                                                            | A "View certificate" action mirroring SCR-008's — symmetrical, and it hands out a record of somebody else's achievement |
| An unrated taught session reads "No rating left", in words, with no stars                   | Five empty stars are the _control_ on the learner's screen (SCR-008 ST-21); rendered here they would be a control that does nothing, and an empty star row reads as a score of zero that the learner never gave. BR-004.1 makes silence a legitimate answer, so the copy states it flatly | Empty stars for visual symmetry with the rated rows; or hiding the rating area entirely, which makes "not rated" and "rated" differ by a missing element rather than by a statement |
| A cancelled session never appears in Sessions you've taught                                 | REQ-027 defines Teaching History as sessions taught. A session that did not happen has no rating to receive and nothing to confirm, so the row would be a record of nothing. Identical to SCR-008's rule for Finished, so the two histories agree about what counts                     | A "Cancelled" group inside the section — complete, and it turns a record of teaching into an activity log |
| No item cap on the third section either                                                     | Uncapped only because the POC is under 10 employees (BRD §1) — a scale assumption, not a principle, and the first one on this screen to revisit. SCR-008 records the same assumption for the same reason                                                                                 | Capping it at 5 with a "View all" — which would need a destination that this screen has just decided not to create |

## Conflicts

Blocks approval of this screen until resolved.

| #   | Conflict                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Between                       | Owner      | Status |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- | ---------- | ------ |
| C-1 | **Nothing says what happens to the other requests for a time once one is approved.** REQ-015 lets any learner request any published, unbooked slot, so a popular time can collect several pending requests. REQ-021 and BR-003.1 allow exactly one booking. REQ-017 gives the trainer approve and reject. No requirement says whether approving one request auto-rejects its rivals, or leaves them pending against a slot they can no longer win. BR-003.1's wording — "when a learner **books** a slot, no other learner may book that same slot" — does not distinguish requesting from confirming, which is precisely the distinction needed here. This decides what a colleague is told, so it is the BA's to settle, not mine. **Working default pending that decision:** approving one request auto-declines the others for that time, each replaced by "Declined — the time was given to Priya. Arun has been notified." (ST-19), and the trainer is warned before they choose. | REQ-015 ↔ REQ-017 ↔ REQ-021 | BA (`/ba`) | open   |
| C-2 | **A session that did not happen has no exit.** REQ-022 gives the trainer exactly one action — mark it complete — and nothing covers the learner who did not turn up, the session both parties abandoned, or the trainer who simply never comes back to this screen. REQ-023, REQ-024 and REQ-027 all fire "on completion", so an unconfirmed session yields no certificate, no rating and no history entry for either party, permanently. Adding ST-22 puts the one available action in front of the trainer, which is as far as design can carry this: the missing case is a rule, not a button. This is the trainer-side half of SCR-008's C-2 and the two must be answered together — whatever answer is given ("it didn't happen" as a second action, an auto-complete after N days, an expiry) lands as new states on both screens. **Working default pending that decision:** the row sits at the top of Sessions you've taught under "Did this happen?" indefinitely, and the design offers no way to say no. | REQ-022 ↔ REQ-023 ↔ REQ-024 ↔ REQ-027 | BA (`/ba`) | open   |

## Open questions

Each has a working default, so none blocks drawing the frames — but each is someone else's call to confirm, and the answer may change a rule or a line of copy.

| #     | Question                                                                                                                                                            | Working default                                                                                          | Owner          | Status                     |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | -------------- | -------------------------- |
| OQ-T1 | May a trainer unpublish a time? REQ-014 says publish and stops                                                                                                      | Yes while nobody has requested it (ST-21); never once requested or booked                                | BA (`/ba`)     | open                       |
| OQ-T2 | Session length — is it fixed by the portal, or chosen per time by the trainer? SCR-006 raised the same gap from the learner's side (OQ-S7)                          | Trainer chooses per time, from 30 / 45 / 60 / 90 minutes                                                 | BA (`/ba`)     | open — paired with OQ-S7   |
| OQ-T3 | Can a trainer publish a time for a skill they have not declared teachable?                                                                                          | No — the picker lists only their teachable skills                                                        | BA (`/ba`)     | open                       |
| OQ-T4 | May two of a trainer's own published times overlap? Sessions are one-on-one (REQ-021), so a trainer cannot be in both                                               | Blocked at the form with an explanatory message (ST-08)                                                  | BA (`/ba`)     | open                       |
| OQ-T5 | REQ-020 lets either party cancel a confirmed session. This screen offers the trainer's entry point (ST-14); SCR-008 has since designed the dialog, its copy and its two failure paths from the learner's side (ST-10 to ST-13, ST-16), so what is left is pointing this entry point at that pattern | "Cancel session" opens nothing yet. Next design pass on this screen adopts SCR-008's confirm-dialog verbatim, with the learner named in place of the trainer | Designer     | open — narrowed, no longer blocked on SCR-008 |
| OQ-T6 | ~~REQ-022 lets a trainer mark a session complete. Is this the place they do it?~~                                                                                   | **Resolved 2026-08-28 — yes.** Sessions you've taught (ST-22 to ST-29) is where a session is confirmed and where the record of it lives                  | Designer + BA  | closed                     |
| OQ-T7 | Should rejecting a request require or offer a reason?                                                                                                               | No reason field — REQ-017 asks for a decision, not an explanation                                        | BA (`/ba`)     | open                       |
| OQ-T8 | How far ahead may a trainer publish?                                                                                                                                | No limit; the date field rejects only dates already past                                                 | BA (`/ba`)     | open                       |
| OQ-T9 | BRD OQ-1 (the auto-reject cutoff) is unresolved, and it decides whether a pending request should warn the trainer that it is about to expire                        | No countdown until BRD OQ-1 is answered; a request reads "asked 2 hours ago" and nothing more            | Business owner | open (tracked as BRD OQ-1) |
| OQ-T10 | May a trainer undo "mark complete"? By then REQ-024 has already produced the learner's certificate and REQ-023 has already prompted them, so an undo has to say what happens to both                          | No — irreversible, which is why ST-23 confirms first and names both consequences                         | BA (`/ba`)     | open                       |
| OQ-T11 | Does the learner's written comment reach the trainer, and is it attributed by name? REQ-027 gives the trainer "rating received" and does not mention comments at all                                             | Shown, attributed — following SCR-008's working default. If the answer is "anonymous", ST-28 drops the name above the comment and nothing else changes   | BA (`/ba`)     | open — paired with OQ-L7   |
| OQ-T12 | ~~Should the trainer see their own average rating (REQ-025) on this screen? REQ-025 scopes the average to their profile and to search results; REQ-027 names four facts and an average is not one of them~~ | **Resolved 2026-08-28 — not shown here.** A trainer sees their own average on their profile (SCR-005, OQ-E8 closed the same day), where every other viewer sees it too. My teaching is where a colleague publishes times and answers requests; leading it with a score changes what the screen is for, and puts one number in two places to keep in step | BA + Designer | closed |
| OQ-T13 | Is there a deadline after which a session can no longer be marked complete? The question only bites if C-2 is answered with an expiry                                                                             | No deadline; the row waits indefinitely                                                                  | BA (`/ba`)     | open — depends on C-2      |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist. ST-01 is worth two frames — desktop and phone width — since NFR-004 is what the reflow has to prove, and ST-12 is the phone form of the whole ST-07…ST-11 family rather than a frame of its own. In the new third section, ST-22, ST-27 and ST-28 are the three shapes a taught row ever takes and are worth drawing carefully; ST-23 to ST-26 are that first shape mid-action and can be drawn as a strip on one frame.

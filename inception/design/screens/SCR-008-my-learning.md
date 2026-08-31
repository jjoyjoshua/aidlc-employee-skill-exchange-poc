# SCR-008 — My learning

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-008/ST-## -->`.

|                  |                                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)                                                     |
| **Traces to**    | REQ-016, REQ-019, REQ-020, REQ-023, REQ-024, REQ-026, NFR-004                                         |
| **Surface**      | `apps/ui` `features/learning` — `/learning`, reached from "My learning" in the app-header (SCR-004)   |
| **Primary user** | Employee acting as a **Learner** — waiting on an answer, attending a session, or looking back at one  |
| **Status**       | draft — awaiting designer review                                                                      |

## Purpose

The learner's half of the portal, end to end. SCR-006 is where an employee asks for a time; this is where they find out what happened to that ask (REQ-016), join the session it became (REQ-019), call it off if they must (REQ-020), and — once it is over — rate the colleague who taught them (REQ-023) and keep the certificate and the record it produced (REQ-024, REQ-026).

"Done" for the learner is one of three things, depending on when they arrive: they know where every request they sent currently stands, they are in the meeting, or they have closed the loop on a session that happened — rating left, certificate in hand.

The past lives on this screen rather than on a separate History destination because the designer chose one learner destination over two (2026-08-28, recorded in the decisions table). A learner's question is almost always "where is my Python session?", and the answer changes from _requested_ to _booked_ to _finished_ without the learner ever meaning to change screens.

### Requirements this screen touches but does not fully serve

Recorded so a later audit doesn't read the manifest as "covered":

| Requirement                            | What this screen does                                                                    | Where the rest lives                                                              |
| -------------------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| REQ-015 (learner submits a request)    | **Reads** the requests already sent and reports where each one stands                    | Sending one is SCR-006's trainer panel                                            |
| REQ-017 (trainer approves or rejects)  | Renders the trainer's decision from the receiving end (ST-18, ST-19)                     | The decision itself is SCR-007                                                    |
| REQ-018 (auto-reject stale requests)   | Shows the learner's side — a request that ran out of time (ST-20)                        | The cutoff itself is BRD OQ-1, unresolved                                         |
| REQ-021 (one learner per session)      | Never renders a second learner, because the system does not produce one                  | Enforced server-side; the screen renders the rule, it does not guarantee it       |
| REQ-022 (trainer marks a session done) | Shows a session waiting on that action (ST-15), with nothing the learner can do about it | The trainer's completion action — not yet designed; see conflict C-2              |
| REQ-025 (trainer's average rating)     | Nothing — a finished row names the trainer, it does not score them                       | SCR-006's result cards and trainer panel                                          |
| REQ-027 (teaching history)             | Nothing — this screen is the learner's side only                                         | The trainer's own history, expected inside SCR-007 — not yet designed             |
| REQ-028 (in-app notifications)         | Every cancellation states that the other party has been notified                         | The notification surface itself — not yet designed                                |

## Layout

The app shell from SCR-004, then a single constrained centre column holding three section cards, in this order:

1. **Coming up** — a session may be starting in ten minutes
2. **Requests you've sent** — waiting on somebody else
3. **Finished** — the record, plus the one action it still carries

```
┌──────────────────────────────────────────────────────────┐
│ SkillEx  Find a trainer  My learning  My teaching  🔔3(JJ)▾│ ← app-header
├──────────────────────────────────────────────────────────┤
│                                                          │
│   My learning                                            │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Coming up (2)                                  │     │
│   │  Today                                         │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Excel · with Priya M.                      │ │     │
│   │ │ 14:00 · 45 min                             │ │     │
│   │ │            [ Join meeting ]  [ Cancel ]    │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │  Fri 5 Sep                                     │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Python · with Arun K. · 11:00 · 60 min     │ │     │
│   │ │ ⏳ Meeting link on its way    [ Cancel ]    │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Requests you've sent (3)                       │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ SQL · Meera S. · Mon 8 Sep, 09:00          │ │     │
│   │ │ ⏳ Pending — asked 2h ago                   │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Figma · Dan R. · Wed 3 Sep, 15:00          │ │     │
│   │ │ ✕ Not this time — Dan couldn't make it     │ │     │
│   │ │                       [ Find another time ]│ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Finished (4)         1 waiting for your rating │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ SQL · with Arun K. · 8 Aug                 │ │     │
│   │ │ ☆ How did it go? ☆☆☆☆☆     [ Certificate ] │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Excel · with Priya M. · 12 Aug             │ │     │
│   │ │ ★★★★☆ 4 out of 5 · "Very clear."           │ │     │
│   │ │                            [ Certificate ] │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

**Sessions are grouped by day**, soonest first, with the day as a subheading — the same grouping SCR-007 uses for availability, for the same reason: the eye finds a day before it finds an hour, and "Today" is the word that matters most on this screen.

**Requests are not grouped.** They are ordered by the requested slot's start time, soonest first, so the one the learner is about to lose patience over sits at the top. Their own ask-time is supplementary ("asked 2h ago"), exactly as it is on the trainer's side.

**Finished is ordered most-recent-first**, with one exception: a session still waiting for a rating sorts above the rated ones regardless of date, because it is the only row in that card carrying an action. The card's header states the count — "1 waiting for your rating" — so the exception is announced rather than merely performed.

**No item cap.** Coming up and Requests are the "View all" destinations for SCR-004's first two sections, so capping them would leave the full list with nowhere to live. Finished is uncapped for the POC's scale (under 10 employees, BRD §1) and is the first section that will need paging when that assumption breaks — recorded in the decisions table.

## States

### ST-01 Default

- **When** the screen has loaded and at least one of the three sections has content
- **Shows** the app-header; the page title; the three section cards in the order above, each with its title and count. A **Coming up** row carries the skill, the trainer's name, the day, date, start time and duration, plus **Join meeting** and **Cancel**. A **Requests** row carries the skill, the trainer, the requested slot's date and time, how long ago it was asked, and a status with an icon and a word — never colour alone. A **Finished** row carries the skill, the trainer, the date it happened, the rating the learner left or an invitation to leave one, and **Certificate**.
- **Can do** join a session; cancel a session (→ ST-10); rate a finished session (→ ST-22); open a certificate (→ ST-27); everything the header allows
- **Note** nothing on this screen is an action taken on somebody else's behalf. Every decision that belongs to the trainer — approve, reject, mark complete — is read here and taken on SCR-007.

### ST-02 Loading

- **When** the screen has mounted and the learning data request is in flight
- **Shows** the app-header and page title in full — neither needs this data — and all three section cards as placeholders: title visible without its count (the count is not yet known), body showing muted skeleton rows at the height the real rows occupy.
- **Can do** use the header. Nothing inside a card is interactive.
- **Why skeletons rather than one page spinner:** consistent with SCR-004, SCR-006 and SCR-007 — the frame is known before the data lands, so the page does not visibly re-flow when it does.

### ST-03 Couldn't load

- **When** the learning data request fails — timeout, server error, no connection
- **Shows** the app-header and title as normal and, in place of all three cards, one error panel: alert icon + "We couldn't load your learning page." + a **Try again** button. The header stays usable so the learner is never trapped.
- **Can do** retry (→ ST-02); use the header
- **Why one error and not three:** the screen loads as a single request, so exactly one thing can fail. Three failure panels would specify a failure mode the design cannot produce. Same reasoning as SCR-007 ST-03.

### ST-04 You haven't started learning yet

- **When** the data loaded successfully and the employee has no sessions, no requests and no finished sessions — the true first visit
- **Shows** all three cards replaced by a single panel: one line explaining that learning here starts by finding a colleague who teaches what you want, and one primary action, **Find a trainer**, pointing at SCR-006. A secondary line points at SCR-005 — "Or add what you want to learn to your profile, so colleagues know" — because REQ-009 lets an employee list skills they want to learn, and a learner who does so is visible in a way an empty profile is not.
- **Can do** follow either action; use the header
- **Why the whole screen and not three empty cards:** three simultaneously empty cards is a wall of nothing. The same decision SCR-004 ST-04 makes about its five sections, for the same reason.

### ST-05 Nothing coming up

- **When** the data loaded, the learner has history or open requests, but no confirmed future session
- **Shows** the Coming up card keeping its title (count reads 0) with its rows replaced by one line: "Nothing booked yet." and, when at least one request is pending, a second clause: "One request is still waiting on a reply." No action — the next step depends on which of those is true, and both are already on the screen below.
- **Can do** everything else on the screen
- **Why the card stays:** removing it would change the screen's shape between visits, so the learner could never learn where sessions appear. Same rule as SCR-007 ST-05.

### ST-06 No open requests

- **When** the data loaded and no request is pending, recently answered, or expired
- **Shows** the Requests card keeping its title (count reads 0) with its rows replaced by one line: "No requests open." plus **Find a trainer** as a quiet link, since that is the only way to create one.
- **Can do** follow the link; everything else on the screen

### ST-07 Nothing finished yet

- **When** the data loaded and no session has been marked complete
- **Shows** the Finished card keeping its title (count reads 0), no "waiting for your rating" clause in its header, and its rows replaced by one line: "Nothing finished yet. Your certificates will appear here." The sentence names what the section is _for_ (REQ-024, REQ-026), because an empty history is otherwise a fact with no promise attached.
- **Can do** everything else on the screen

### ST-08 A confirmed session

- **When** a booking request was approved (REQ-017) and the session's start time is still in the future
- **Shows** that row with a tick icon and the trainer's name spelled out — "with Priya M." — the skill as a chip, the day, date, start time and duration; **Join meeting** as the primary action (REQ-019); and **Cancel** as a quiet, non-primary action (REQ-020). Exactly one trainer appears, ever (REQ-021).
- **Can do** join the meeting, which opens the link in a new tab (see OQ-L8); start a cancellation (→ ST-10)
- **Note** no requirement defines a join window, so **Join meeting** is offered for the whole life of the row rather than unlocking near the start time. Raised alongside OQ-L3; the working default is deliberately the permissive one, since a learner who is early is not a problem the portal needs to solve.

### ST-09 A confirmed session with no meeting link yet

- **When** the session is confirmed but REQ-019's link has not been produced — generation is pending, or the integration failed (NFR-005, BRD OQ-2)
- **Shows** the same row with **Join meeting** replaced by an hourglass icon and the text "Meeting link on its way." **Cancel** stays. If the row is still in this state within the hour before the session starts, the text becomes "No meeting link yet — message Priya directly." naming the trainer, because at that point the learner needs a person and not a status.
- **Can do** cancel; nothing else on the row
- **Why a status and not a dead button:** a **Join meeting** button that opens nothing is worse than no button, and REQ-019 makes the link something the system owes the learner rather than something they can retry into existence.
- **Note** what the portal does when link generation fails permanently is not specified by any requirement. The wording above is a working default, and "message Priya directly" points outside the portal rather than inventing a messaging feature.

### ST-10 Cancelling — the confirm step

- **When** the learner activates **Cancel** on a confirmed session (REQ-020)
- **Shows** a dialog over the page, the page behind inert under a scrim: the title "Cancel this session?", a line naming exactly what is being cancelled — "Excel with Priya M., today at 14:00" — and a second line naming the consequence: "Priya will be notified. This can't be undone." (REQ-028). Actions: **Keep the session** (secondary, focused by default) and **Cancel the session** (danger emphasis).
- **Can do** confirm (→ ST-11); keep, or press Escape, which closes without cancelling and returns focus to the **Cancel** button on the row
- **Why a dialog and not the in-row confirm SCR-007 uses for Remove:** SCR-007's own reasoning draws the line — an in-row confirm is enough friction for something no colleague depends on. This one takes a time away from a colleague who has committed it, so it carries the weight of a modal, and the safe action holds focus.
- **Blocked on** what happens to the trainer's slot afterwards — see conflict **C-1**. The dialog deliberately makes no promise about it either way.

### ST-11 Cancelling — in flight

- **When** the learner has confirmed and the call is in flight
- **Shows** both dialog buttons disabled, **Cancel the session** showing a spinner in place of its label. The dialog cannot be dismissed while a write is in flight — closing it would leave the learner unsure whether the session still exists.
- **Can do** nothing until it resolves

### ST-12 You cancelled

- **When** the cancel call succeeds
- **Shows** the dialog closing; the row replaced in place by a confirmation line — tick icon + "Cancelled — Priya has been notified." (REQ-028) — in muted text, held for the rest of the visit and gone on the next load; the section count decrementing. Focus moves to that line. If it was the only session, the card enters ST-05.
- **Can do** everything ST-01 allows
- **Why the row stays as a line rather than vanishing:** a row disappearing at the moment of a decision leaves the learner unsure the decision landed, and this one has a consequence for someone else that they should see stated. Same reasoning as SCR-006 ST-13 and SCR-007 ST-20.
- **Note** a cancelled session never reaches **Finished**. REQ-026 defines Learning History as sessions that were completed, and a session that did not happen has no certificate (REQ-024) and nothing to rate (REQ-023).

### ST-13 Couldn't cancel

- **When** the cancel call fails
- **Shows** the dialog staying open, both buttons re-enabled, and an inline error above the actions: alert icon + "Couldn't cancel that session. Please try again." The wording blames the saving, never the session.
- **Can do** try again; keep the session
- **Special case** if the failure is specifically that the session was already cancelled from the other side while the dialog was open, the message becomes "Priya has already cancelled this session." and the dialog offers a single **Close** action, which on dismissal leaves the row in ST-14 rather than restoring it. A different fact deserves a different sentence, and a retry that will fail identically is not an offer.

### ST-14 The trainer cancelled

- **When** the trainer cancelled a confirmed session (REQ-020, from SCR-007's side) and the learner is seeing the result
- **Shows** the row in muted text with an alert icon and "Cancelled by Priya M." — the trainer named, because "cancelled" without a subject reads as though the learner did it — and beneath it **Find another time** as a quiet link to that trainer on SCR-006. No Join, no Cancel. The row stays for the rest of the visit and is gone on the next load.
- **Can do** follow the link; nothing else on the row
- **Note** whether the trainer gave a reason is not shown, because no requirement captures one — see OQ-L4.

### ST-15 A session whose time has passed

- **When** a confirmed session's start time has gone by and the trainer has not yet marked it complete (REQ-022)
- **Shows** the row moved into a **"Waiting to be confirmed"** group at the bottom of Coming up, in muted text with a clock icon and "Waiting for Priya to confirm this happened." No action of any kind. It does not appear in **Finished**, and no certificate exists for it yet (REQ-024).
- **Can do** nothing
- **Why it is named rather than hidden:** the learner's certificate and their chance to rate both depend on an action only the trainer can take. Hiding the row would make an unexplained absence out of a state the learner is entitled to understand. What happens if the trainer never acts is **conflict C-2**, and this copy is honest precisely because it makes no promise about that.

### ST-16 Cancelling at phone width

- **When** ST-10, ST-11 or ST-13 is reached below the desktop breakpoint (NFR-004)
- **Shows** the same dialog as a sheet rising from the bottom of the screen rather than a centred dialog: the two lines of copy at full width, and the **Keep the session** / **Cancel the session** pair stacked and pinned to the bottom of the sheet, the safe action on top.
- **Can do** everything ST-10 to ST-13 allow at that step
- **Why a sheet:** matches SCR-006's trainer panel and SCR-007's slot form at phone width, so the portal has one way of showing an overlay on a phone rather than three.

### ST-17 A request still pending

- **When** a request has been sent and the trainer has not answered it (REQ-016)
- **Shows** the row with an hourglass icon and the label "Pending", the requested slot's day, date and start time, the trainer's name, the skill as a chip, and "asked 2h ago" as supplementary text. No action — see OQ-L1.
- **Can do** nothing on the row
- **Note** no countdown to the auto-reject cutoff is shown, because BRD OQ-1 has not fixed one. Raised as OQ-L9 and paired with SCR-007's OQ-T9, so both sides of the booking move together.

### ST-18 A request that was approved

- **When** the trainer approved the request (REQ-017) and it has become a confirmed session
- **Shows** the row with a tick icon and "Approved — it's in Coming up", the last four words being a link that moves focus to the matching session row above. Held for the rest of the visit; gone on the next load, because the session itself is the durable record and two rows for one booking is one row too many.
- **Can do** follow the link
- **Why it appears here at all:** REQ-016 names Approved as one of the three statuses a learner must be able to see. Dropping the request the instant it succeeds would satisfy the booking and quietly fail the requirement.

### ST-19 A request that was rejected

- **When** the trainer rejected the request (REQ-017)
- **Shows** the row with a cross icon and "Not this time — Dan couldn't make it", the trainer named, plus **Find another time** as a quiet link back to that trainer on SCR-006. The row stays until the slot's start time passes (see OQ-L2).
- **Can do** follow the link
- **Why the copy avoids the word "rejected":** the status a learner reads about themselves should describe the outcome, not grade them. No reason is shown because REQ-017 asks the trainer for a decision and not an explanation (SCR-007's OQ-T7) — so there is nothing to display, and adding a field here would decide that question by accident.

### ST-20 A request that ran out of time

- **When** a pending request passed the auto-reject cutoff before the trainer answered it (REQ-018)
- **Shows** the row in muted text with a clock icon and "Ran out of time — Meera didn't get to this before the session.", the trainer named, plus **Find another time** as a quiet link. Distinct copy from ST-19 because the cause is distinct: nobody decided against this learner.
- **Can do** follow the link
- **Note** the cutoff itself is BRD OQ-1, unresolved. This state renders REQ-018's effect without asserting when it fires.

### ST-21 A finished session you haven't rated

- **When** the trainer marked the session complete (REQ-022) and the learner has not submitted a rating
- **Shows** the row at the top of **Finished**, carrying the skill, the trainer, the date it happened, and an invitation — "How did it go?" beside five empty stars that are themselves the control (REQ-023). **Certificate** sits at the end of the row, available immediately and independently: the certificate is generated on completion with no manual step (REQ-024), so it is never withheld pending a rating.
- **Can do** rate (→ ST-22); open the certificate (→ ST-27)
- **Note** there is no **Skip** or **Dismiss**. REQ-023 makes submitting optional, and an optional thing does not need declining — leaving it alone is the decline. A dismiss action would also have to remember the dismissal, which is state no requirement defines.

### ST-22 Rating — the form

- **When** the learner activates a star, or the "How did it go?" invitation, on an unrated finished session
- **Shows** the row expanding in place — not a dialog — to reveal: the five stars filled to the value being hovered or focused, with the chosen value spelled out beside them ("4 out of 5"); an optional multi-line comment field labelled "Anything you'd like to add? (optional)"; and a **Submit rating** / **Cancel** pair. **Submit rating** is enabled as soon as a star is chosen, since REQ-023 makes only the star value required.
- **Can do** change the star value; type a comment; submit (→ ST-23); cancel, which collapses the row back to ST-21 and discards what was typed
- **Why in-row and not a dialog:** the rating is about the row it sits on and carries one required field. A dialog would separate the rating from the session it describes, and the confirm-step weight a dialog implies belongs to actions that affect a colleague's calendar, not to optional feedback.

### ST-23 Rating — submitting

- **When** the learner has submitted and the call is in flight
- **Shows** the stars and comment field disabled at the values entered, **Cancel** disabled, and **Submit rating** showing a spinner in place of its label.
- **Can do** nothing until it resolves

### ST-24 Rating saved

- **When** the submit call succeeds (REQ-023)
- **Shows** the row collapsing to its rated form (ST-26) with a tick icon and "Thanks — your rating has been saved." beside it briefly; the "waiting for your rating" count in the card header decrementing, and disappearing entirely when it reaches zero; the row re-sorting into date order among the rated sessions. Focus stays on the row.
- **Can do** everything ST-01 allows
- **Note** no page-level success banner — consistent with SCR-004 ST-07, SCR-006 ST-15 and SCR-007 ST-10: confirmation belongs where the change landed. The trainer's average (REQ-025) updates wherever it is shown; this screen does not display it.

### ST-25 Couldn't save the rating

- **When** the submit call fails
- **Shows** the form staying open with the star value and any comment intact, controls re-enabled, and an inline error above the actions: alert icon + "Couldn't save your rating. Please try again."
- **Can do** submit again; cancel, which discards it

### ST-26 A finished session you've rated

- **When** the learner has submitted a rating for a completed session (REQ-026 — "feedback they submitted")
- **Shows** the row with the skill, the trainer, the date, the stars filled to the submitted value with that value spelled out beside them, the comment beneath in quotation marks when one was left, and **Certificate**. The rating is read-only — see OQ-L6.
- **Can do** open the certificate (→ ST-27)

### ST-27 The certificate

- **When** the learner activates **Certificate** on any completed session (REQ-024)
- **Shows** the certificate opening in a new browser tab as a printable page carrying the four things REQ-024 names — the learner's name, the skill, the trainer, and the date completed — set in the portal's own typography and colour tokens. The row itself is unchanged; the action shows a brief spinner in place of its label while the tab is prepared.
- **Can do** print or save it from the browser; return to this tab, which is untouched
- **Note** the medium — a printable page rather than a generated PDF or a shareable link — is a working default. REQ-024 specifies the contents and that generation is automatic; it says nothing about the format. Raised as OQ-L5.

### ST-28 The certificate isn't available

- **When** the certificate request fails, or the session is complete but its certificate has not yet been produced
- **Shows** the **Certificate** action replaced in place by an alert icon and "Certificate isn't ready yet. Try again in a moment." with **Try again** beside it. The rest of the row is unaffected, and the rating remains available — the two are independent.
- **Can do** retry; everything else the row allows
- **Why this state exists at all:** REQ-024 promises the certificate is generated automatically with no manual step, which makes its absence a system failure rather than a missing user action — so the copy blames the generation, offers a retry, and never suggests the learner do something to earn it.

## Components

| Component        | Preview                                                   | States covered                                                                                                                        |
| ---------------- | --------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `session-row`    | `inception/design/components/session-row/preview.html`    | confirmed, no meeting link, cancelled by you, cancelled by trainer, waiting to be confirmed — ST-01, ST-08, ST-09, ST-12, ST-14, ST-15 |
| `confirm-dialog` | `inception/design/components/confirm-dialog/preview.html` | confirm, in flight, failed, phone sheet — ST-10, ST-11, ST-13, ST-16                                                                  |
| `request-row`    | `inception/design/components/request-row/preview.html`    | pending, approved, rejected, ran out of time — ST-17, ST-18, ST-19, ST-20                                                             |
| `history-row`    | `inception/design/components/history-row/preview.html`    | unrated, rated, certificate opening, certificate unavailable — ST-21, ST-26, ST-27, ST-28                                              |
| `rating-form`    | `inception/design/components/rating-form/preview.html`    | form, submitting, saved, failed — ST-22, ST-23, ST-24, ST-25                                                                          |
| `card`           | `inception/design/components/card/preview.html`           | loading skeleton, three empty sections — ST-02, ST-05, ST-06, ST-07                                                                   |
| `empty-state`    | `inception/design/components/empty-state/preview.html`    | nothing at all yet — ST-04                                                                                                            |
| `alert`          | `inception/design/components/alert/preview.html`          | the page-level load failure — ST-03                                                                                                   |
| `rating-stars`   | `inception/design/components/rating-stars/preview.html`   | the read-only star display reused inside a rated row                                                                                  |
| `app-header`     | `inception/design/components/app-header/preview.html`     | the shell inherited from SCR-004, with "My learning" added as a destination                                                           |
| `skill-chip`     | `inception/design/components/skill-chip/preview.html`     | the skill named on every row on this screen                                                                                           |
| `button`         | `inception/design/components/button/preview.html`         | primary, secondary, danger, loading, disabled                                                                                         |
| `spinner`        | `inception/design/components/spinner/preview.html`        | the in-flight indicator behind ST-11, ST-23 and ST-27                                                                                 |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Keyboard:** header tab order unchanged from SCR-004. Then the page, top to bottom, section by section. Within a Coming up row, **Join meeting** precedes **Cancel** in both visual and tab order — the safe, frequent action first. Within a Finished row, the star control precedes **Certificate**.
- **The star control (ST-21, ST-22):** a radio group of five, not five buttons, labelled "Rate this session". Arrow keys move between values, Space or Enter selects, and the chosen value is spelled out as text beside the glyphs so it never depends on counting filled shapes. The glyphs are `aria-hidden`; the number is the information — the same division `rating-stars` already makes for the read-only case (REQ-025).
- **Dialog (ST-10 to ST-13, ST-16):** focus moves to **Keep the session** on open — the safe action — and is trapped inside until it closes. Escape cancels, except during ST-11 where a write is in flight and Escape does nothing. On close, focus returns to the **Cancel** button that opened it.
- **Focus after an action:** ST-12 moves focus to the confirmation line that replaced the row; ST-18's link moves focus to the matching session row above; ST-24 keeps focus on the row. Focus is never dropped to `<body>`.
- **Focus ring:** every interactive element shows the `--c-focus-ring` outline on keyboard focus, including whole-row targets and each star in the group.
- **Non-colour signalling (NFR-003):** "Pending", "Approved", "Not this time", "Ran out of time", "Cancelled by Priya M.", "Waiting to be confirmed", "Meeting link on its way" and every error each carry an icon **and** a word. The danger emphasis on **Cancel the session** also carries its full verb and a confirm step. A submitted rating is always accompanied by its numeric value in text, so the row is legible with no colour and no glyph rendering at all.
- **Announcements:** each section card is a landmark region labelled by its heading. ST-02's skeleton region is `aria-busy="true"`. The dialog is `role="dialog"` `aria-modal="true"` labelled by its title. ST-12's confirmation, ST-24's "Thanks" and ST-28's failure are announced `aria-live="polite"` — the learner caused them and is looking at them. ST-03's panel is `aria-live="assertive"`.
- **Time and date:** every session and requested slot is shown as day, date and clock time in the learner's local zone; "Today" and "Tomorrow" replace the weekday name where they apply, because those two are the words that change behaviour. "asked 2h ago" is the one relative reading and it is supplementary to the slot's absolute time, never a replacement for it. A finished session shows its date only — the clock time of something that is over is noise.
- **Responsive (NFR-004):** the single column reflows to full width below the breakpoint; row actions move from beside the text to a full-width stack beneath it; the dialog becomes a sheet (ST-16); day subheadings become sticky while scrolling; the rating form's five stars stay on one line at every width, since each must remain individually tappable.
- **New tabs:** **Join meeting** (ST-08) and **Certificate** (ST-27) both open a new tab and both say so with the conventional external-link affordance, so a learner mid-flow is never surprised by losing this page.
- **Motion:** ST-22's expansion, ST-24's collapse and ST-12's replacement are short transitions, suppressed under `prefers-reduced-motion`.

## Structural decisions

| Decision                                                                                     | Rationale                                                                                                                                                                                                                                                                        | Alternative rejected                                                                                                                                                             |
| -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Upcoming, requests **and** finished sessions on one screen                                   | Designer's decision, 2026-08-28. A learner's question is "where is my Python session?", and the answer moves from requested to booked to finished without them meaning to change screens. One destination follows the thing, not its lifecycle stage                             | A separate History destination holding both learning and teaching history — a cleaner filing system, and it splits one learner's single question across two screens               |
| Coming up first, then requests, then finished                                                | Time-critical, then blocked-on-someone-else, then reference. Identical to SCR-004's section order, which this screen is the "View all" destination for                                                                                                                            | Requests first, mirroring SCR-007 — right there because a colleague is blocked on the trainer; wrong here, because the learner is blocked on nobody and a session may be starting |
| Sessions grouped by day; requests and finished sessions listed flat                          | Grouping earns its keep where several rows share a day, which is true of a calendar and rarely true of a request list. Matches SCR-007's availability grouping                                                                                                                   | Grouping all three sections for visual symmetry — subheadings over one-row groups                                                                                                 |
| An unrated finished session sorts above rated ones, and the card header says how many        | It is the only row in that section carrying an action, so it earns the top. Announcing the count in the header keeps the sort from looking like a bug                                                                                                                             | Strict date order — consistent, and it buries the only actionable row under the archive                                                                                           |
| No dismissible "rate your session" banner at the top of the screen                           | REQ-023 makes submitting optional, and a banner that must be dismissed contradicts that. The prompt REQ-023 asks for is served by the notification (REQ-028) plus a visible affordance on the row itself                                                                          | A top-of-page prompt card — more prominent, and it needs a dismissal state no requirement defines, plus a second place to render the same rating                                  |
| Cancelling opens a modal (ST-10); rating expands in the row (ST-22)                          | The two actions carry different weight. Cancelling takes a committed time away from a colleague; rating is optional feedback about a row you are already looking at. SCR-007 draws the same line between its in-row Remove confirm and its slot-form dialog                       | One convention for both — a modal for the rating separates it from the session it describes; an in-row confirm for the cancel is too little friction for something that costs someone else their time |
| An approved request is shown as a resolved row for the visit, not dropped on success (ST-18) | REQ-016 names Approved as a status the learner must be able to see. Dropping it the instant it succeeds satisfies the booking and quietly fails the requirement                                                                                                                   | Removing approved requests immediately, on the argument that the session row above says the same thing — it does, but only to someone who already knows to look                   |
| A rejected or expired request stays until its slot's start time passes                       | Derivable from data already on the row, so no arbitrary retention window is invented. Once the slot has gone by there is nothing left to act on                                                                                                                                   | A fixed number of days, which is a business rule I would be making up; or keeping them forever, which turns the section into a second history                                     |
| A cancelled session never appears in Finished                                                | REQ-026 defines Learning History as completed sessions. A session that did not happen has no certificate (REQ-024) and nothing to rate (REQ-023), so a row in Finished would be a record of nothing with two empty columns                                                       | A "Cancelled" group inside Finished — complete, and it turns the history into a log rather than the record of achievement REQ-024 and REQ-026 describe                            |
| A passed-but-unconfirmed session stays in Coming up under its own subheading (ST-15)         | It is not finished, so Finished would be a lie; and dropping it would hide the fact that the learner's certificate is waiting on somebody. Naming the state without inventing an action keeps C-2 open honestly                                                                  | Moving it into Finished with no certificate; or removing it and letting the certificate appear one day with no explanation of the gap                                             |
| The certificate is offered independently of the rating (ST-21)                               | REQ-024 makes generation automatic with no manual step, and REQ-023 makes rating optional. Gating the certificate on a rating would make an optional thing mandatory by the back door                                                                                             | "Rate to unlock your certificate" — effective at collecting ratings, and a direct contradiction of two requirements                                                               |
| No Skip or Dismiss on the rating invitation                                                  | An optional action does not need declining; leaving it alone is the decline. A dismissal would also have to be remembered, which is state no requirement defines                                                                                                                  | A "No thanks" action that hides the invitation permanently                                                                                                                       |
| A missing meeting link shows a status, never a disabled button (ST-09)                       | REQ-019 makes the link something the system owes the learner, not something they can retry into existence. A dead **Join meeting** invites a click and then explains itself — the same reasoning SCR-007 applies to withdrawing Remove rather than greying it out                | A disabled Join button with a tooltip; or hiding the row until its link exists, which loses the session from the learner's calendar                                               |
| "Not this time" rather than "Rejected" (ST-19)                                                | The status a learner reads about themselves should name the outcome, not grade them — and REQ-017 gives the trainer no field in which to explain, so there is nothing further to show                                                                                            | The literal requirement wording, which is correct in a database and unkind on a screen                                                                                            |
| No item cap on any section                                                                   | Coming up and Requests are the dashboard's "View all" destinations, so capping them would leave the full list nowhere to live. Finished is uncapped only because the POC is under 10 employees (BRD §1) — a scale assumption, not a principle, and the first one to revisit      | Matching SCR-004's cap of 5 — right for a summary, wrong for the destination the summary points at                                                                                |

## Conflicts

Blocks approval of this screen until resolved.

| #   | Conflict                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Between                       | Owner      | Status |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- | ---------- | ------ |
| C-1 | **Nothing says what happens to the trainer's time slot when a confirmed session is cancelled.** REQ-020 lets either party cancel before the session starts and stops there. REQ-014 gives the trainer a published slot; REQ-015 lets any learner request an unbooked one; REQ-021 allows exactly one booking. Nobody has said whether cancelling returns that slot to the pool — bookable again by anyone, including the learners whose competing requests were declined when it was first approved (SCR-007's C-1) — or retires it. This decides a line of copy in the cancel dialog (ST-10), whether the slot reappears on SCR-006, and what SCR-007's availability row does next. It is a rule about the booking model, not a visual choice. **Working default pending that decision:** the dialog promises nothing about the slot beyond notifying the trainer, and this screen renders neither outcome.        | REQ-014 ↔ REQ-015 ↔ REQ-020 | BA (`/ba`) | open   |
| C-2 | **The learner's certificate and their chance to rate both depend on an action only the trainer can take, with no fallback.** REQ-022 lets a trainer mark a session complete. REQ-023 and REQ-024 both fire "on completion". REQ-026 promises the learner a Learning History containing certificates. A session that happened but was never marked complete therefore produces no certificate, no rating and no history entry — permanently — and no requirement gives the learner, or anyone else, a way to resolve it. ST-15 is where that state lands, and its copy is deliberately non-committal because the design cannot answer this. **Working default pending that decision:** the row sits under "Waiting to be confirmed" indefinitely with no action offered.                                                                                                                                            | REQ-022 ↔ REQ-024 ↔ REQ-026 | BA (`/ba`) | open   |

## Open questions

Each has a working default, so none blocks drawing the frames — but each is someone else's call to confirm, and the answer may change a rule or a line of copy.

| #      | Question                                                                                                                                                                                     | Working default                                                                                                                                  | Owner          | Status                     |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | -------------------------- |
| OQ-L1  | May a learner withdraw a request that is still pending? REQ-020 covers confirmed sessions only, and says nothing about un-asking                                                              | No withdraw action; a pending request can only be answered or expire                                                                             | BA (`/ba`)     | open                       |
| OQ-L2  | How long does a rejected or expired request stay on the screen?                                                                                                                              | Until the requested slot's start time passes — derivable from the row, no retention window invented                                              | BA (`/ba`)     | open                       |
| OQ-L3  | Is there a cutoff before the start time after which a session can no longer be cancelled? REQ-020 says "before it starts" and no more. The same gap decides whether **Join meeting** unlocks near the start | Cancellable at any point before the start time; **Join meeting** offered for the whole life of the row                                           | BA (`/ba`)     | open                       |
| OQ-L4  | Must, or may, the party cancelling give a reason? REQ-028 says the other party is notified but not what the notification carries                                                             | No reason field on either side — mirrors SCR-007's OQ-T7 for rejections                                                                          | BA (`/ba`)     | open — paired with OQ-T7   |
| OQ-L5  | What _is_ a certificate — a printable page in the portal, a generated PDF, or a shareable link? REQ-024 names its four contents and that generation is automatic; it names no medium         | A printable page opening in a new tab, set in the portal's tokens                                                                                 | BA + Designer  | open                       |
| OQ-L6  | Can a learner change or withdraw a rating after submitting it? REQ-023 says submitting is optional; it does not say it is final                                                              | Final — ST-26 is read-only                                                                                                                       | BA (`/ba`)     | open                       |
| OQ-L7  | Is the learner's written comment shown to the trainer, and is it attributed by name? REQ-023 collects it; REQ-027 gives the trainer "rating received" and does not mention comments          | The comment reaches the trainer attributed to the learner — but that is a privacy commitment nobody has made, so it is raised rather than assumed | BA (`/ba`)     | open                       |
| OQ-L8  | REQ-019 shares a meeting link; NFR-005 leaves the platform undecided (BRD OQ-2). Does **Join meeting** simply open a URL, or does the portal do more?                                        | Opens the link in a new tab; no in-portal embed, no calendar integration                                                                          | IT + BA        | open (tracked as BRD OQ-2) |
| OQ-L9  | BRD OQ-1 (the auto-reject cutoff) is unresolved, and it decides whether a pending request should warn the learner it is about to expire                                                      | No countdown; a pending request reads "asked 2h ago" and nothing more                                                                             | Business owner | open (tracked as BRD OQ-1) |
| OQ-L10 | SCR-004's header lists a **History** destination. With learning history now inside this screen — and teaching history expected inside SCR-007 — does a separate History destination survive? | It does not; the header reads Find a trainer · My learning · My teaching · My skills                                                              | Designer       | open — your call at review |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist. ST-01 is worth two frames — desktop and phone width — since NFR-004 is what the reflow has to prove. ST-16 is the phone form of the whole ST-10…ST-13 family rather than a frame of its own, and ST-22…ST-25 are four steps of one expanding row, best drawn as a strip.

# SCR-004 — Dashboard

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-004/ST-## -->`.

|                  |                                                          |
| ---------------- | -------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)        |
| **Traces to**    | REQ-004, REQ-005, REQ-006, REQ-017, NFR-004              |
| **Surface**      | `apps/ui` `features/dashboard` — `/dashboard` (the post-login landing route, REQ-004) |
| **Primary user** | Employee — the same person acting as Learner and Trainer on one account |
| **Status**       | draft — awaiting designer review                         |

## Purpose

The first screen an employee sees after signing in (REQ-004), and the place they return to between tasks. It answers one question at a glance — _what needs me today?_ — across the five sections REQ-006 names: sessions coming up, booking requests still pending, skills they teach, skills they are learning, and recent notifications.

"Done" is one of two things: the employee sees that nothing needs them and leaves satisfied, or they clear the one thing that did — approving a booking request (REQ-017) or joining a session that is starting — without navigating anywhere.

This screen also introduces the **application shell**: the top bar carrying the portal identity, the destinations, the notification bell, and Log out (REQ-005). Every authenticated screen designed after this one inherits that shell; it is specified here because this is the first screen to have one.

### Requirements this screen touches but does not fully serve

Recorded so a later audit doesn't read the manifest as "covered":

| Requirement                           | What this screen does                                    | Where the rest lives                                                      |
| ------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------------- |
| REQ-016 (learner sees booking status) | Shows the employee's own _pending_ requests               | A bookings screen must show approved/rejected history — not yet designed  |
| REQ-019 (video-meeting link)          | Surfaces the link as a "Join meeting" action on a session | Link generation and a session detail view — not yet designed              |
| REQ-028 (in-app notifications)        | Shows the five most recent, plus an unread count on the bell | A full notifications surface — not yet designed                        |
| REQ-009/010/011 (skills)              | Lists what the employee teaches and learns, read-only     | Adding, editing and removing skills — not yet designed                    |

## Layout

One header, then a single-column stack of section cards on a constrained centre column. The card _order_ is the priority order: what needs a decision first, what is read-only last.

Chosen over a two-column dashboard grid because the priority stays unambiguous on every device — the same order top to bottom on a phone and on a monitor, so there is one layout to design, one to build, and no question of which column the eye reads first (NFR-004).

```
┌──────────────────────────────────────────────────────────┐
│ SkillEx   Find a trainer  My skills  History   🔔3  (JJ)▾│  ← app-header
├──────────────────────────────────────────────────────────┤
│                                                          │
│   Good morning, Joy                                      │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Upcoming sessions (2)              View all →  │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Excel Pivot Tables · with Priya M.         │ │     │
│   │ │ Today 14:00 · 45 min       [ Join meeting ]│ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Booking requests (3)               View all →  │     │
│   │  Waiting for you                               │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ Priya M. · Excel · Thu 14:00               │ │     │
│   │ │              [ Approve ]  [ Reject ]       │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   │  Waiting for them                              │     │
│   │ ┌────────────────────────────────────────────┐ │     │
│   │ │ You asked Arun K. · Python · Fri 11:00     │ │     │
│   │ │              ⏳ Pending                     │ │     │
│   │ └────────────────────────────────────────────┘ │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│   ┌───────────────────────┐  ┌───────────────────────┐   │
│   │ Skills I teach  Edit →│  │ Skills I'm learning → │   │
│   │ [Excel · Expert]      │  │ [Python · Beginner]   │   │
│   └───────────────────────┘  └───────────────────────┘   │
│      (side by side on desktop, stacked on mobile)        │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Recent activity                    View all →  │     │
│   │ • Arun approved your Python session   2h ago   │     │
│   └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

**Section order, and why.** Upcoming sessions first (time-critical, one may be starting now) → Booking requests (someone is waiting on a decision) → Skills, teach and learn (reference, changes rarely) → Recent activity (informational). The two skills cards are the only pair sitting side by side, because they are the shortest and are read as a pair.

**Item cap.** Each section shows at most **5** items, soonest or newest first, with a "View all (N)" link when there are more. A dashboard that grows without bound stops being a summary. 5 is a starting default and is the designer's to change.

## States

### ST-01 Default

- **When** the employee has signed in, the dashboard data has loaded, and at least one section has content
- **Shows** the app-header (portal name, destinations, bell with unread count, avatar); a greeting line with the employee's first name; the five sections in the order above, each with its title, its item count, and up to 5 items. Approved upcoming sessions each carry a "Join meeting" action. Requests waiting on this employee carry **Approve** and **Reject**; requests they sent show a "Pending" status with an icon and a text label.
- **Can do** approve or reject a request in place (→ ST-06); join a session; open the avatar menu; follow any "View all" or "Edit" link; open the notification bell

### ST-02 Loading

- **When** the screen has mounted and the dashboard data request is in flight
- **Shows** the app-header rendered in full — it needs no dashboard data, since identity and destinations are already known — the greeting line, and each section card in a placeholder state: title visible, body showing muted skeleton rows at the height the real rows will occupy. Section titles appear without counts, because the count is not yet known.
- **Can do** use the header (navigate away, log out). Nothing inside a card is interactive.
- **Why skeletons rather than one page spinner:** the header and section titles are known before the data arrives, so showing them immediately means the page does not visibly re-flow when content lands.

### ST-03 Nothing yet (first visit)

- **When** the data loaded successfully and **every** section is empty — a brand-new employee who has just set their password and signed in for the first time
- **Shows** the app-header and greeting, then a single full-width welcome panel _in place of_ the five empty cards: one line explaining what the portal is for, and two clearly ranked actions — **Add a skill** (primary) and **Find a trainer** (secondary). Five simultaneously empty cards is a wall of nothing; one panel with a next action is an onboarding moment.
- **Can do** follow either action; use the header
- **Note** this state is defined by _all_ sections being empty. As soon as any one has content the screen is ST-01, with per-section empties handled by ST-04.

### ST-04 A section with nothing in it

- **When** the data loaded and at least one section has content, but a given section has none
- **Shows** that section's card keeping its title (count reads 0 or is omitted) and replacing its rows with one line of section-specific copy plus, where there is an obvious next step, a single link:

  | Section             | Copy                                                | Action          |
  | ------------------- | --------------------------------------------------- | --------------- |
  | Upcoming sessions   | "No sessions booked."                               | Find a trainer  |
  | Booking requests    | "No requests waiting."                              | —               |
  | Skills I teach      | "You haven't listed anything to teach yet."         | Add a skill     |
  | Skills I'm learning | "You haven't listed anything you want to learn."    | Add a skill     |
  | Recent activity     | "Nothing yet."                                      | —               |

- **Can do** follow the section's action where one exists
- **Why the card stays:** removing an empty section would change the dashboard's shape between visits, so an employee could never learn where things are.

### ST-05 Could not load

- **When** the dashboard data request fails — timeout, server error, no connection
- **Shows** the app-header as normal and, in place of the section stack, a single error panel: alert icon + "We couldn't load your dashboard." + a **Try again** button. The header stays usable so the employee is never trapped — they can still navigate or log out.
- **Can do** retry (→ ST-02); use the header
- **Why one error and not five:** the dashboard loads as a single request, so there is one thing that can fail. Per-section failure states would specify a failure mode the design cannot produce.

### ST-06 Responding to a request

- **When** the employee has clicked Approve or Reject on a request and the call is in flight
- **Shows** that request row only: both its buttons disabled, the clicked button showing a spinner in place of its label. Every other row and section stays fully interactive — one slow request must not freeze the dashboard.
- **Can do** nothing on that row; everything else on the screen still works

### ST-07 Response recorded

- **When** the approve or reject call succeeds (REQ-017)
- **Shows** the row replaced in place by a confirmation line — tick icon + "Approved — Priya has been notified." or "Rejected — Priya has been notified." — held for a few seconds, after which the row leaves the list and the section count decrements. On approval the session appears in Upcoming sessions. A page-level success banner is deliberately not used; the confirmation belongs where the action happened.
- **Can do** everything ST-01 allows

### ST-08 Response failed

- **When** the approve or reject call fails
- **Shows** the row restored to its interactive form with an inline error line beneath it: alert icon + "Couldn't save that. Please try again." Both buttons are re-enabled. The wording says the _saving_ failed, so nobody reads it as the request itself being invalid.
- **Can do** retry the same action, or take the other one

### ST-09 Session starting soon

- **When** an approved upcoming session's start time falls inside the "starting soon" window (default **15 minutes** — see OQ-D3)
- **Shows** that session promoted to the top of Upcoming sessions and visually raised: an accent left border, a "Starting soon" label with a clock icon (never colour alone), and the **Join meeting** button changed from secondary to primary emphasis. The time reads relatively — "in 8 minutes" — rather than as a clock time.
- **Can do** join the meeting; everything ST-01 allows
- **Note** the Join action exists on every approved upcoming session, not only this one. This state changes prominence, not availability.

### ST-10 Signing out

- **When** the employee opens the avatar menu and chooses Log out (REQ-005)
- **Shows** that menu item replaced by a spinner, the menu held open while the session ends, and the page beneath inert. No confirmation dialog — nothing is lost by logging out, and a confirm on a safe action trains people to dismiss dialogs unread.
- **Can do** nothing until it completes; the employee is then returned to SCR-001 in its ST-01 default carrying the "You've been signed out" notice, and the portal is not reachable again without signing in (REQ-005)
- **On failure** the menu item is restored with an inline "Couldn't sign out. Please try again." The employee is never told they are signed out when they are not.

## Components

| Component     | Preview                                                | States covered                                                                       |
| ------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| `app-header`  | `inception/design/components/app-header/preview.html`  | default, unread bell, avatar menu open, mobile collapsed, signing out — ST-01, ST-10 |
| `card`        | `inception/design/components/card/preview.html`        | populated, loading skeleton, empty, error, starting-soon — ST-02, ST-04, ST-05, ST-09 |
| `request-row` | `inception/design/components/request-row/preview.html` | incoming, outgoing pending, responding, decided, failed — ST-06, ST-07, ST-08        |
| `skill-chip`  | `inception/design/components/skill-chip/preview.html`  | teach and learn variants at each proficiency level                                   |
| `empty-state` | `inception/design/components/empty-state/preview.html` | first-visit welcome panel — ST-03                                                    |
| `button`      | `inception/design/components/button/preview.html`      | primary, secondary, loading, disabled                                                |
| `alert`       | `inception/design/components/alert/preview.html`       | error and success variants behind ST-05 and ST-08                                    |
| `spinner`     | `inception/design/components/spinner/preview.html`     | the in-flight indicator for ST-06 and ST-10                                          |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Keyboard:** header tab order is portal name → each destination → bell → avatar. Then the page: the greeting is not focusable; within each section, "View all" first, then each row's actions in visual order. The avatar menu opens on Enter or Space, moves focus to its first item, cycles with arrow keys, and closes on Escape returning focus to the avatar.
- **Focus after an action:** when ST-07 removes a request row, focus moves to that section's heading — never lost to `<body>`, which would throw a keyboard user back to the top of the page.
- **Focus:** every interactive element shows the `--focus-ring` outline on keyboard focus, including rows that are clickable as a whole.
- **Non-colour signalling:** "Pending", "Approved", "Rejected" and "Starting soon" each carry an icon **and** a text label. Proficiency on a skill chip is spelled out (Beginner / Intermediate / Expert), never encoded as a colour or a count of dots. The unread bell shows a numeral, not only a coloured dot.
- **Announcements:** each section card is a landmark region labelled by its heading. ST-02's skeleton region is `aria-busy="true"`. The ST-07 confirmation and the ST-08 error are announced `aria-live="polite"` — polite rather than assertive, because the employee caused the action and is already looking at it. ST-05's panel is `aria-live="assertive"`.
- **Responsive (NFR-004):** the single column reflows to full width below the desktop breakpoint; the two skills cards stack; header destinations collapse behind a menu button while the bell and avatar stay visible, so logging out and reading notifications are never buried in a menu.
- **Motion:** ST-07's row removal is a fade, suppressed under `prefers-reduced-motion`.

## Structural decisions

| Decision                                                                              | Rationale                                                                                                                                                                                                                                                    | Alternative rejected                                                                                                                                                                                          |
| ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Booking requests can be approved or rejected directly on the dashboard                | Designer's decision, 2026-08-28. It is the action a trainer performs most often and the one most likely to be blocking someone else; routing it through a second screen adds two clicks to the portal's most frequent decision                              | A read-only dashboard where every card links out; and a full control-panel dashboard where skills are edited inline too — the first is slow on the common path, the second duplicates screens still to design |
| Persistent top bar as the application shell, with Log out inside an avatar menu       | Designer's decision, 2026-08-28. Works unchanged from phone to monitor, costs the least vertical space on a phone, and puts Log out where people already look. Specified here because this is the first authenticated screen; every later screen inherits it | A left sidebar — more room for destinations, but it consumes horizontal space on laptops and needs a second, drawer-based design for mobile                                                                   |
| Single column of full-width sections, not a two-column grid                           | The section order _is_ the priority order, and one column preserves that order identically at every width — one layout to design and build, satisfying NFR-004 with no separate mobile design                                                                | A 2×3 dashboard grid; on mobile it collapses to a column anyway, so the grid exists only on desktop and makes the reading order ambiguous there                                                               |
| Booking requests are one section split into "Waiting for you" and "Waiting for them"  | REQ-006 says "pending booking requests" without saying whose. One account is both learner and trainer, so both readings are real. Two labelled groups in one section shows both without inventing a rule about which matters more, and makes the actionable ones obvious | Two separate cards — pushes everything else down for a distinction most employees have one or two of; or picking one reading and silently dropping the other                                            |
| Skills sections are read-only here, linking out to edit                               | Editing a skill involves a catalog picker and a proficiency choice (REQ-009, REQ-010) that need their own screen. Read-only keeps the dashboard a summary                                                                                                    | Inline add and edit — a second, smaller implementation of a screen that has to be built properly anyway                                                                                                       |
| One dashboard-wide error state (ST-05), no per-section failure                        | The dashboard loads in one request, so exactly one thing can fail. Specifying five failure states would design a behaviour the system cannot produce                                                                                                         | Per-card error states — plausible-looking, but untestable against this data flow                                                                                                                              |
| An approve/reject confirmation appears in the row (ST-07), not as a page banner       | The employee is looking at the row they clicked; a banner at the top of the page moves the feedback away from the action, and off-screen entirely on a phone                                                                                                 | A success banner above the section stack, matching SCR-001's ST-06 — right for a page transition, wrong for an in-place action                                                                                |
| A whole-dashboard empty state (ST-03) replacing the cards, not five empty cards       | A first-time employee's screen should propose a first action, not present five statements of absence                                                                                                                                                         | Rendering all five sections in their ST-04 empty form on first visit                                                                                                                                          |
| No confirmation dialog on Log out                                                     | Nothing is lost, and the action is one click to undo; a confirm here trains people to dismiss dialogs without reading them                                                                                                                                   | "Are you sure you want to log out?"                                                                                                                                                                           |
| Every section capped at 5 items with "View all (N)"                                   | Keeps the dashboard a summary at any data volume, and gives every section a defined maximum height to design against                                                                                                                                         | Unbounded lists — fine with today's under-10-user POC, unusable the moment it isn't                                                                                                                          |

## Conflicts

No requirement conflicts on this screen. REQ-004, REQ-005, REQ-006, REQ-017 and NFR-004 are all satisfiable together as specified above.

| #   | Conflict | Between | Owner | Status |
| --- | -------- | ------- | ------- | ------ |
| —   | none     | —       | —     | —      |

## Open questions

Each has a working default, so none blocks drawing the frames — but each is someone else's call to confirm, and the answer may change a piece of copy or a threshold.

| #     | Question                                                                                                                                                                                                     | Working default                                                                                     | Owner          | Status                        |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | -------------- | ----------------------------- |
| OQ-D1 | The original brief (input §2) asks the dashboard to show "recent notifications **and announcements**", but no requirement defines an announcement — REQ-028 lists event notifications only. Is there a separate broadcast concept, or was that wording loose? | The section is titled "Recent activity" and carries event notifications only (REQ-028); no announcements concept is designed | BA (`/ba`)     | open                          |
| OQ-D2 | REQ-006's "pending booking requests" does not say whose — requests the employee sent, requests awaiting their decision, or both                                                                              | Both, in one section split into "Waiting for you" and "Waiting for them"                            | BA (`/ba`)     | open                          |
| OQ-D3 | How close to a session's start should it be promoted as "Starting soon" (ST-09)?                                                                                                                             | 15 minutes before start                                                                             | Business owner | open                          |
| OQ-D4 | BRD OQ-1 (the auto-reject cutoff) is unresolved. If it lands on a value employees should see, a pending request ought to warn "auto-declines in 6 hours"                                                     | No countdown until BRD OQ-1 is answered; the row shows "Pending" only                                | Business owner | open (tracked as BRD OQ-1)    |
| OQ-D5 | BRD OQ-2 (video platform) is unresolved, and it decides the Join button's wording                                                                                                                            | Platform-neutral "Join meeting"                                                                     | IT             | open (tracked as BRD OQ-2)    |
| OQ-D6 | The greeting is time-based ("Good morning, Joy"). That is a tone choice, and it assumes a reliable local timezone                                                                                             | Time-based greeting, first name only                                                                | Designer       | open — your call at review    |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist. ST-01 and ST-03 are each worth two frames — desktop and mobile width — since NFR-004 is what the reflow has to prove.

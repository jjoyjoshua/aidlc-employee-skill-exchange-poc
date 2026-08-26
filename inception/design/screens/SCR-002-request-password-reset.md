# SCR-002 — Request a password reset

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-002/ST-## -->`.

|                  |                                                             |
| ---------------- | ----------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)           |
| **Traces to**    | REQ-003, NFR-004                                            |
| **Surface**      | `apps/ui` `features/auth` — `/forgot-password`               |
| **Primary user** | Employee (any portal user, before authentication)           |
| **Status**       | draft — awaiting designer review                            |

## Purpose

An employee who cannot get past SCR-001 asks for a reset link to be emailed to their corporate address (REQ-003). "Done" for them is a clear confirmation that a link is on its way, plus enough information to recover from the most common failure — having typed the wrong address. This is the first of the two screens that make up REQ-003's reset flow; SCR-003 is where the new password is actually chosen.

Reached from the "Forgot password?" link on SCR-001. There is no other entry point: an employee already signed in changes their password from their profile (REQ-007), which is a different screen and out of scope here.

## Layout

The same single centered card as SCR-001, deliberately unchanged — same width, same vertical rhythm, same plain background. Continuing the login screen's layout means the employee does not feel they have left the sign-in flow, and it inherits NFR-004 (one layout from mobile width to desktop) with no new responsive work.

```
┌───────────────────────────────────────┐
│              [ Logo/Title ]           │
│                                       │
│      ┌─────────────────────────┐      │
│      │ Forgot your password?   │      │
│      │ <one line of guidance>  │      │
│      ├─────────────────────────┤      │
│      │ Email                   │      │
│      ├─────────────────────────┤      │
│      │ [   Send reset link   ] │      │
│      ├─────────────────────────┤      │
│      │ Back to sign in         │      │
│      └─────────────────────────┘      │
└───────────────────────────────────────┘
```

## States

### ST-01 Default

- **When** the employee arrives from SCR-001's "Forgot password?" link, having submitted nothing yet
- **Shows** a heading ("Forgot your password?"), one line of guidance explaining that a link will be emailed to them, an empty email field with placeholder, an enabled "Send reset link" button, and a "Back to sign in" link returning them to SCR-001. If the employee had already typed an email on SCR-001, that value is carried across and pre-filled — they have already told us once.
- **Can do** type into the email field, submit (button or Enter), or go back to sign in

### ST-02 Email validation error

- **When** the employee submits with the email field blank, or containing something that is not a syntactically valid email address (client-side check, before any server call — consistent with SCR-001/ST-02)
- **Shows** the field outlined and marked with an inline message directly beneath it — "Email is required." when blank, "Enter a valid email address." when malformed. The submit button stays enabled so they can retry immediately.
- **Can do** correct the field and resubmit

### ST-03 Sending

- **When** a syntactically valid address has been submitted and the server call is in flight
- **Shows** the email field and the button disabled, the button showing a spinner in place of its label ("Send reset link" becomes "Sending…"). Matches SCR-001/ST-03 exactly.
- **Can do** nothing — no double-submission possible while this state holds

### ST-04 Request received

- **When** the server has accepted the request. **This state is shown whether or not the submitted address belongs to an account** — see the structural decision below.
- **Shows** the form is replaced by a confirmation panel: a success icon, "Check your email", and the message "If **jane.doe@company.com** belongs to a portal account, we've sent a link to reset the password. The link expires in *[60 minutes — unconfirmed, OQ-3]* and can be used once." The submitted address is echoed back **in bold**, because it is the only way an employee who mistyped it can discover that they did. Beneath: "Didn't arrive? Check the address above, then **send it again**" and a "Back to sign in" link.
- **Can do** trigger a resend (moves to ST-06), or return to sign in. The email field is no longer editable in this state; correcting a typo means going back and starting again, which is deliberate — an editable field here would make the confirmation read as provisional.

### ST-05 Couldn't send the request

- **When** the request did not complete for a reason unrelated to the address itself — timeout, server error, no connection
- **Shows** the form stays on screen with everything the employee typed intact, and an error banner above it: an alert icon + "Something went wrong. Please try again." Wording matches SCR-001/ST-05 so the same failure reads the same way across the sign-in flow.
- **Can do** resubmit; nothing has been lost

### ST-06 Link sent again

- **When** the employee triggers the resend offered in ST-04
- **Shows** the confirmation panel from ST-04 with an added inline note — a success icon + "Sent again — check your email." The resend control is disabled for a short cooldown and its label reflects that, so a confused employee cannot generate a queue of emails with repeated clicks. **The exact cooldown and any daily cap are not yet decided (OQ-5).**
- **Can do** wait out the cooldown and resend again, or return to sign in

## Components

| Component    | Preview                                               | Notes                                                          |
| ------------ | ----------------------------------------------------- | -------------------------------------------------------------- |
| `text-input` | `inception/design/components/text-input/preview.html` | email field — default (ST-01), error (ST-02), disabled (ST-03)  |
| `button`     | `inception/design/components/button/preview.html`     | "Send reset link" — default, sending (ST-03), resend cooldown   |
| `alert`      | `inception/design/components/alert/preview.html`      | confirmation panel (ST-04), failure banner (ST-05), resent note (ST-06) |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Keyboard:** tab order is email → "Send reset link" → "Back to sign in". Enter, pressed in the email field, submits. In ST-04 the order is "send it again" → "Back to sign in".
- **Focus:** every interactive element shows the `--focus-ring` outline on keyboard focus. On entering ST-04 focus moves to the confirmation heading, so a screen-reader user is not left on a button that no longer exists. On entering ST-02 focus moves to the email field.
- **Non-colour signalling:** ST-02, ST-04, ST-05 and ST-06 each pair an icon with text. Border and background colour are never the only cue.
- **Announcements:** the ST-04 confirmation and the ST-05 banner are inserted with `aria-live="assertive"`. ST-06's "Sent again" note uses `aria-live="polite"` — it confirms a repeat of something already announced and should not interrupt. The ST-02 message is linked to the input via `aria-describedby`.
- **Autocomplete:** the email field carries `autocomplete="username"` so a password manager fills it, matching SCR-001.

## Structural decisions

| Decision | Rationale | Alternative rejected |
| -------- | --------- | -------------------- |
| The same confirmation (ST-04) is shown whether or not the address has an account | Decided by the designer, 2026-08-26. Without it, anyone — unauthenticated — could type any address into this screen and learn whether that person has an account, which on an internal staff portal leaks who works here. It also keeps the promise SCR-001 already makes: that screen deliberately never reveals whether the email or the password was wrong (REQ-002) | Telling the employee honestly that no account exists for that address. Kinder to a typo, but turns the screen into an account-lookup tool for anyone who loads it |
| The submitted address is echoed back in the confirmation, in bold, with "check the address above" | This is the accepted cost of the decision above: an employee who mistyped gets a reassuring message and no email. Showing the address back is the cheapest way to make the mistake visible without revealing anything — we are repeating what they typed, not telling them anything about our records | A bare "Check your email" with no address shown, which leaves a mistyped address completely invisible and the employee waiting indefinitely |
| ST-04 replaces the form rather than appearing above it | The employee's next action is in their inbox, not on this screen. Leaving an editable field beside a "we've sent it" message invites a second submission and makes the confirmation read as provisional | Banner above a still-live form, as in ST-05 |
| Layout, wording and state numbering deliberately mirror SCR-001 | Continuity: this is the same journey, and two of the failure states (network error, field validation) are literally the same failures. Reusing the wording means one thing to translate and one thing to change | A visually distinct "utility" treatment for the recovery flow |
| A cooldown on resend, rather than an unlimited "send again" | An employee who is not receiving the email will click repeatedly. Unlimited resends mean a queue of near-identical emails, and — depending on OQ-4 — each new link may silently kill the previous one, so the link they eventually click is the one most likely to be dead | An always-available resend control |

## Conflicts and open questions

Blocks approval of this screen until each row has a resolution.

| #    | Conflict / question | Between | Owner | Status |
| ---- | ------------------- | ------- | ----- | ------ |
| OQ-3 | How long is a reset link valid? REQ-003 says "expires after a set time" without fixing the time, and ST-04's copy has to state it. Recommended default: **60 minutes** — long enough to survive a delayed corporate mail relay, short enough that a link left sitting in an inbox is not a standing key | REQ-003 (unquantified) vs. ST-04 copy | BA (with IT) | open |
| OQ-4 | If an employee requests a second link, is the first invalidated? REQ-003 fixes single *use*, not single *outstanding link*. This decides what an employee sees when they click the older of two emails: SCR-003's working form, or its dead-link message. Recommended default: **invalidate the older link** — fewer live keys, at the cost of one confusing case | REQ-003 (silent) vs. SCR-003/ST-03 | BA (with Architect) | open |
| OQ-5 | How often may "send again" be used — cooldown length, and any per-hour or per-day cap? ST-06 shows *that* the control is limited; the numbers are a rule, not a design choice. Recommended default: **60-second cooldown, 5 requests per address per hour** | ST-06 vs. no stated rule | BA (with IT) | open |
| OQ-6 | The "same message either way" behaviour in ST-04 is a security rule that BRD-001 does not state anywhere — the designer decided it here (see structural decisions). It needs to become a numbered requirement or business rule in its own right, so that it is testable and so a future change to this screen cannot quietly drop it | This spec vs. BRD-001 | BA | open |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist. ST-01/ST-02/ST-03/ST-05 share one composition (the form) and ST-04/ST-06 share another (the confirmation panel), so six states are two compositions plus variants.

All copy in this spec is a **draft for the designer and BA to overwrite**. It is written to be specific enough to review and argue with, not to be final wording.

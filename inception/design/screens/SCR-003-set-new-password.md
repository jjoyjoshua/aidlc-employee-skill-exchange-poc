# SCR-003 — Set a new password

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-003/ST-## -->`.

|                  |                                                                  |
| ---------------- | ---------------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)                |
| **Traces to**    | REQ-003, NFR-004                                                 |
| **Surface**      | `apps/ui` `features/auth` — `/reset-password` (link token in URL) |
| **Primary user** | Employee (any portal user, before authentication)                |
| **Status**       | draft — awaiting designer review                                 |

## Purpose

An employee who clicked the link emailed to them by SCR-002 chooses a new password and gets back into the portal (REQ-003). "Done" is being returned to SCR-001 with a confirmation, where they sign in with the password they just set.

This is the screen where REQ-003's two hard promises — the link **expires**, and the link **works only once** — become something the employee can actually see. Half of this spec is therefore about links that no longer work, which is the normal case rather than the exception: the employee arrives here minutes or hours after the email was sent, possibly on a different device, possibly having already used the link once.

**This screen is reached only by following the emailed link.** It is never linked to from inside the portal, and loading it without a valid token shows ST-03, never the form.

## Layout

The same single centered card as SCR-001 and SCR-002 (NFR-004, one layout at every width). The card has two quite different contents depending on whether the link is usable — a form, or an explanation — and the states below are grouped that way.

```
  Link usable (ST-02)                Link not usable (ST-03, ST-04)
┌───────────────────────────┐      ┌───────────────────────────┐
│ Set a new password        │      │  [ ! ]                    │
│ for jane.doe@company.com  │      │  This link has expired    │
├───────────────────────────┤      │                           │
│ New password       [👁]    │      │  <what happened, and      │
├───────────────────────────┤      │   what to do about it>    │
│ ✓ at least 12 characters  │      │                           │
│ ✗ not a common password   │      │  [ Request a new link ]   │
├───────────────────────────┤      │                           │
│ Confirm new password      │      │  Back to sign in          │
├───────────────────────────┤      └───────────────────────────┘
│ [   Save new password   ] │
└───────────────────────────┘
```

The requirement checklist sits **between** the two password fields rather than below both. It describes the first field only, and putting it directly under that field keeps its subject unambiguous — a checklist floating below "Confirm password" reads as if it applied to the confirmation.

## States

### ST-01 Checking the link

- **When** the screen first loads, while the token from the URL is being validated. This is unavoidable: whether to show a form or an explanation cannot be known until the server answers, so this screen has a genuine loading state before anything else.
- **Shows** the card containing only a centered spinner and "Checking your link…". No form, no error — showing the form first and snatching it away would be worse than a brief wait.
- **Can do** nothing. Expected to be brief (NFR-003, under 3 seconds).

### ST-02 Ready

- **When** the token has been confirmed valid, unused, and unexpired
- **Shows** a heading ("Set a new password"), the account's email address beneath it as confirmation of *whose* password is being changed, a "New password" field with a show/hide toggle, the live requirement checklist (every rule listed, each marked met or unmet with an icon **and** a text label), a "Confirm new password" field, and an enabled "Save new password" button. The checklist is visible from the outset with everything unmet, not revealed on first error — the employee should be able to read the rules before typing, not discover them by failing.
- **Can do** type in either field, toggle password visibility, submit

### ST-03 Link no longer valid

- **When** the token is expired, unrecognised, or malformed. These are **one state on purpose**: the employee's next action is identical in all three, and the system may genuinely be unable to tell them apart — an expired token that has been purged from storage is indistinguishable from one that never existed (see OQ-8).
- **Shows** no form at all. A warning icon, "This link is no longer valid", and an explanation: "Password reset links expire *[60 minutes — unconfirmed, OQ-3]* after they're sent, and can only be used once. Request a new one to continue." A primary "Request a new link" button returning to SCR-002, and a "Back to sign in" link.
- **Can do** request a new link, or go back to sign in. **Deliberately no retry** — nothing about this link will start working.

### ST-04 Link already used

- **When** the token is recognised and was valid, but has already been consumed by a successful password change (REQ-003's single-use promise)
- **Shows** no form. A warning icon, "This link has already been used", and: "The password for this account was already changed using this link. If that wasn't you, contact IT straight away." A "Back to sign in" button, and a secondary "Request a new link" option for the employee who genuinely used it and has since forgotten again.
- **Can do** go back to sign in, or request a new link
- **Why this is separate from ST-03:** the two states demand different things of the employee. "Expired" means *try again*; "already used" may mean *someone else has your reset link*, and that is worth saying out loud. Collapsing them would bury the only moment the portal could plausibly surface a compromised reset. This separation depends on OQ-8 — if the system cannot distinguish a consumed token from an unknown one, this state is unbuildable and folds into ST-03.

### ST-05 Password doesn't meet the rules

- **When** the employee submits, or leaves the new-password field, with one or more rules unmet
- **Shows** the field outlined as invalid and the checklist updated — unmet rules marked with a cross icon and the word "Missing", met rules with a tick and "Met". No count, no strength meter, no colour-only bar: the employee is told exactly which rule is unsatisfied and nothing else. The submit button stays enabled so they can retry.
- **Can do** amend the password and resubmit. The checklist updates as they type, so the state resolves itself without another submission.
- **The rules themselves are not yet decided (OQ-7).** BRD-001 quantifies nothing about password composition — NFR-001 governs only how the password is stored. The checklist is designed to render whatever list of rules is agreed; the two shown in previews are a recommendation, not a decision.

### ST-06 Passwords don't match

- **When** the employee submits with the confirmation field not matching the new-password field
- **Shows** the confirmation field outlined and marked with an inline message beneath it: "These passwords don't match." The new-password field and the checklist are left untouched — the password itself may be perfectly fine, and clearing it would force the employee to retype something they cannot see. Kept separate from ST-05 because the fix is in a different field and neither message should be shown for the other's problem.
- **Can do** correct the confirmation and resubmit

### ST-07 Saving

- **When** a valid, matching password has been submitted and the server call is in flight
- **Shows** both fields and the button disabled, the button showing a spinner in place of its label ("Save new password" becomes "Saving…"). Matches SCR-001/ST-03 and SCR-002/ST-03.
- **Can do** nothing — a double submission here would hit a token that the first request has already consumed, and land the employee on ST-04 for a change they made themselves

### ST-08 Couldn't save

- **When** the request failed for a reason unrelated to the password or the token — timeout, server error, no connection
- **Shows** the form stays on screen, both fields keep what was typed, and an error banner appears above: an alert icon + "Something went wrong. Your password has not been changed. Please try again." The second sentence is the important one — after a failed submission the employee has no way of knowing which password is now live, and guessing wrong sends them back to SCR-002.
- **Can do** resubmit. The link is still unconsumed, so retrying is safe.

**Success is not a state of this screen.** A saved password returns the employee to SCR-001, where the confirmation is shown as that screen's ST-06. This follows the designer's decision that an employee signs in with the new password rather than being dropped straight into the dashboard — so the last thing this screen does is navigate away, and there is nothing left here to design.

## Components

| Component               | Preview                                                          | Notes                                                        |
| ----------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------ |
| `text-input`            | `inception/design/components/text-input/preview.html`            | password fields — with toggle, error on confirmation (ST-06), disabled (ST-07) |
| `button`                | `inception/design/components/button/preview.html`                | "Save new password" — default, saving (ST-07)                 |
| `alert`                 | `inception/design/components/alert/preview.html`                 | dead-link explanation (ST-03), already-used warning (ST-04), save failure (ST-08) |
| `password-requirements` | `inception/design/components/password-requirements/preview.html` | the live rule checklist — all unmet (ST-02), partially met (ST-05) |
| `spinner`               | `inception/design/components/spinner/preview.html`               | link validation (ST-01)                                       |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Keyboard:** tab order in ST-02 is new password → show/hide toggle → confirm password → "Save new password". Enter in either field submits. In ST-03/ST-04 it is the primary button → the secondary link.
- **Focus:** every interactive element shows the `--focus-ring` outline on keyboard focus. On moving from ST-01 to ST-02 focus lands on the new-password field. On ST-03/ST-04 focus lands on the heading, because there is no field to fill and the explanation is the point. On ST-06 focus moves to the confirmation field.
- **Non-colour signalling:** every checklist row carries an icon **and** the word "Met" or "Missing" — a tick that is only green is invisible to a colour-blind employee and to anyone reading a greyscale screenshot. ST-03, ST-04, ST-06 and ST-08 all pair an icon with text.
- **Announcements:** the checklist is a `role="list"` inside an `aria-live="polite"` region, so a rule flipping to met is announced without interrupting typing; each row's state is in its text, not only its icon. ST-08's banner is `aria-live="assertive"`. ST-03/ST-04 replace the card's content and move focus to the new heading, so a screen reader reads the explanation from the top.
- **Password managers:** both fields carry `autocomplete="new-password"`, which is what tells a manager to offer to generate and then save the new password — the single most useful accessibility affordance on this screen.
- **Show/hide toggle:** present on the new-password field, matching SCR-001. Not on the confirmation field: being able to reveal both defeats the point of asking twice.

## Structural decisions

| Decision | Rationale | Alternative rejected |
| -------- | --------- | -------------------- |
| Success returns to SCR-001 with a confirmation there, rather than signing the employee straight in | Decided by the designer, 2026-08-26. Three reasons: the employee proves the new password works while it is still in front of them (or freshly in their manager), rather than discovering a typo days later; the link arrived by email, so auto-signing-in would make inbox access alone sufficient for a live session; and REQ-004's "dashboard immediately after a successful login" stays the *only* route into the dashboard, instead of this screen quietly adding a second one | Dropping the employee into the dashboard already authenticated. One click cheaper, on a screen an employee sees roughly twice a year |
| Expired, unrecognised and malformed links are one state (ST-03) | The employee's next action is identical for all three, and the distinction may not even be knowable — a purged expired token looks exactly like one that never existed (OQ-8). Three messages for one action is three things to translate and maintain, for no gain | A distinct message per failure kind, which reads as thorough and is mostly unbuildable |
| "Already used" is a separate state (ST-04) despite the above | This one *is* actionable, and differently: it may mean someone else holds the reset link. It is the only point in the flow where the portal can plausibly surface that, so it gets its own words and an instruction to contact IT | Folding it into ST-03's "request a new one", which would silently discard the one security signal this screen can offer |
| A rule checklist, not a strength meter | A meter tells an employee they are at "medium" without telling them what to change; a checklist names the unmet rule. It also degrades honestly — a checklist works in greyscale and reads aloud, a coloured bar does neither | A colour-graded strength bar, which would additionally violate the non-colour-signalling rule |
| The checklist is shown from the outset with everything unmet, not revealed on first failure | Rules an employee can read before typing are guidance; rules that appear after a rejection are a reprimand. Costs nothing | Showing the checklist only once a rule is broken |
| The checklist sits between the two fields, not below both | It describes the first field only. Below the confirmation field it reads as if it applied to the confirmation | Checklist below the whole form |
| ST-06 (mismatch) is separate from ST-05 (rules) and does not clear either field | Different field, different fix, and the passwords are masked — clearing on error would force a retype of something the employee cannot see, which is how people end up locked out of a reset flow entirely | One combined "check your entries" validation state, as SCR-001/ST-02 uses for its two blank fields |
| ST-08's message states explicitly that the password has **not** changed | After a failed save the employee cannot tell which password is live. Without that sentence, the reasonable guess is "it probably worked", and the next sign-in failure sends them round the whole flow again | A generic "Something went wrong. Please try again.", matching the other screens' wording exactly |
| The account's email address is shown in ST-02 | Reset links get forwarded and shared inboxes exist. Naming the account whose password is about to change is a one-line safeguard against changing the wrong one. It reveals nothing — the employee is holding the link already | An unattributed "Set a new password" heading |

## Conflicts and open questions

Blocks approval of this screen until each row has a resolution.

| #    | Conflict / question | Between | Owner | Status |
| ---- | ------------------- | ------- | ----- | ------ |
| OQ-3 | How long is a reset link valid? (shared with SCR-002) — ST-03's explanation has to state it. Recommended default: **60 minutes** | REQ-003 (unquantified) vs. ST-03 copy | BA (with IT) | open |
| OQ-7 | **What are the password rules?** BRD-001 states none — NFR-001 governs storage only, not composition. ST-02 and ST-05 cannot be built, tested, or drawn without the list. Recommended default, following current NIST 800-63B guidance: **minimum 12 characters, checked against a list of common and breached passwords, with no forced mixture of character types and no forced expiry** — length and screening beat composition rules, which mainly produce `Password1!`. This is a recommendation only; the decision is the BA's with IT | ST-02/ST-05 vs. BRD-001 (silent) | BA (with IT) | open |
| OQ-8 | Can the system distinguish a link that was **already used** from one that is **expired or unknown**? If expired tokens are deleted rather than marked consumed, it cannot — and **ST-04 becomes unbuildable and folds into ST-03**. This is a storage-design question with a directly visible consequence, which is why it is raised here rather than left to implementation | ST-04 vs. token storage design | Architect | open |
| OQ-9 | Does setting a new password sign the employee out of their other active sessions? Not stated anywhere; REQ-005 covers only a deliberate logout. It matters most in the case ST-04 warns about — if someone else used the reset link, an existing session of theirs surviving the change makes the reset largely cosmetic. Recommended default: **end all other sessions**, and say so in SCR-001's ST-06 confirmation | REQ-003/REQ-005 vs. no stated rule | BA | open |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist. The eight states are three compositions: the spinner card (ST-01), the form (ST-02, ST-05, ST-06, ST-07, ST-08) and the explanation card (ST-03, ST-04).

The checklist rows in any frame are **placeholders until OQ-7 is answered** — draw the row, not the rule.

All copy in this spec is a **draft for the designer and BA to overwrite**, with one exception worth defending: ST-08's "Your password has not been changed" earns its place.

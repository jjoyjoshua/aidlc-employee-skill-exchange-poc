# SCR-001 — Login

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-001/ST-## -->`.

|                  |                                                      |
| ---------------- | ---------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)    |
| **Traces to**    | REQ-001, REQ-002, REQ-003, REQ-004, NFR-004          |
| **Surface**      | `apps/ui` `features/auth` — `/login`                 |
| **Primary user** | Employee (any portal user, before authentication)    |
| **Status**       | draft — awaiting designer review                     |

## Purpose

An employee enters their corporate email and password to reach their dashboard. "Done" is either a successful redirect to the Dashboard (REQ-004), or a clear, generic message telling them why it failed and what to do next. The "Forgot password?" link on this screen is the entry point into REQ-003's reset flow — the request/reset screens themselves are a separate, upcoming screen spec (SCR-002, not yet drafted).

## Layout

Single centered card on a plain background — no side branding panel. Chosen so the same layout works unmodified from mobile width to desktop width (NFR-004), with no separate responsive variant to design or maintain.

```
┌───────────────────────────────────────┐
│                                       │
│              [ Logo/Title ]          │
│                                       │
│         ┌───────────────────┐        │
│         │  Email             │        │
│         ├───────────────────┤        │
│         │  Password    [👁]  │        │
│         ├───────────────────┤        │
│         │  Forgot password?  │        │
│         ├───────────────────┤        │
│         │   [   Log in   ]   │        │
│         └───────────────────┘        │
│                                       │
└───────────────────────────────────────┘
```

## States

### ST-01 Default

- **When** the employee lands on `/login` with no prior submission (fresh visit, or just logged out via REQ-005)
- **Shows** empty email and password fields, placeholder text in each, the "Forgot password?" link, an enabled "Log in" button. If arriving from logout, a small dismissible confirmation ("You've been signed out") above the card.
- **Can do** type into either field, click "Forgot password?", submit the form (click "Log in" or press Enter in either field)

### ST-02 Field validation error

- **When** the employee submits with one or both fields blank (client-side check, before any server call — Validations §7 of BRD-001)
- **Shows** each blank field outlined and marked with an inline message ("Email is required." / "Password is required.") directly under that field; the submit button stays enabled so they can retry
- **Can do** correct the field(s) and resubmit

### ST-03 Submitting

- **When** the employee has submitted a syntactically complete form and the server call is in flight
- **Shows** both fields and the submit button disabled, the button showing a spinner in place of its label ("Log in" → spinner)
- **Can do** nothing — no double-submission possible while this state holds

### ST-04 Invalid credentials

- **When** the server rejects the submitted email/password combination (REQ-002)
- **Shows** a single banner above the form: an alert icon + "Incorrect email or password." — never a hint at which field was wrong (REQ-002). The password field is cleared; the email field keeps what was typed, so the employee isn't forced to retype it.
- **Can do** retype and resubmit; the "Forgot password?" link stays available

### ST-05 Submission failed (network/server error)

- **When** the request could not complete for a reason unrelated to the credentials themselves (timeout, server error, no connection)
- **Shows** a banner above the form: an alert icon + "Something went wrong. Please try again." — distinct wording from ST-04 so the employee knows this isn't about their password
- **Can do** resubmit once ready; both fields keep what was typed (nothing to protect here, unlike ST-04)

## Components

| Component    | Preview                                                | Notes                                              |
| ------------ | ------------------------------------------------------- | --------------------------------------------------- |
| `text-input` | `inception/design/components/text-input/preview.html`  | default, focused, error, disabled — email + password fields |
| `button`     | `inception/design/components/button/preview.html`      | default, hover, loading, disabled — the "Log in" submit button |
| `alert`      | `inception/design/components/alert/preview.html`       | error, info variants — ST-04 / ST-05 banners        |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists.

## Interaction and accessibility

- **Keyboard:** tab order is email → password → show/hide password toggle → "Forgot password?" → "Log in". Enter, pressed in either field, submits the form.
- **Focus:** every interactive element (fields, toggle, link, button) shows the `--focus-ring` outline on keyboard focus.
- **Non-colour signalling:** ST-02/ST-04/ST-05 all pair a warning icon with text — the field/banner border colour is never the only cue that something is wrong.
- **Announcements:** the ST-04/ST-05 banner is inserted with `aria-live="assertive"` so a screen reader announces it the moment it appears. Each field-level message in ST-02 is linked to its input via `aria-describedby`.

## Structural decisions

| Decision | Rationale | Alternative rejected |
| -------- | --------- | --------------------- |
| Single centered card, no branding side-panel | Identical layout at any width, satisfies NFR-004 with no separate responsive design | Two-column layout with an illustration panel — extra asset + dev work, panel just disappears on mobile anyway |
| One combined error banner for bad email *or* bad password (ST-04), never a per-field highlight | REQ-002 requires the error not to reveal which field was wrong; highlighting only the "wrong" field would leak that | Marking only the password field as invalid (common pattern elsewhere, but violates REQ-002 here) |
| Separate wording + a distinct state (ST-05) for network/server failure vs. bad credentials (ST-04) | The employee's next action differs — retry later vs. re-check what they typed; collapsing them into one message would send someone into "check your password" for an outage that isn't about their password | One generic "something went wrong" message for every failure |
| Password field includes a show/hide toggle | Reduces mistyped-password failures at near-zero cost; a very common, low-risk pattern | Masked-only password field, no toggle |

## Conflicts and open questions

None identified — REQ-001 through REQ-004 and NFR-004 do not conflict on this screen.

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist.

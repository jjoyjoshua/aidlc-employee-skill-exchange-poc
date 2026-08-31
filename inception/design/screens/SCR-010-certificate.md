# SCR-010 — Certificate of completion

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-010/ST-## -->`.

|                  |                                                                                 |
| ---------------- | ------------------------------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)                               |
| **Traces to**    | REQ-024, REQ-026, NFR-004                                                       |
| **Surface**      | `apps/ui` `features/learning` — `/certificate/<session-id>`, opened in a new tab |
| **Primary user** | Employee, acting as Learner — the person the certificate is about               |
| **Status**       | draft — awaiting designer review                                                |

## Purpose

The page a learner opens when they want the thing they earned: proof that they
completed a session in a skill, taught by a named colleague, on a named date.
REQ-024 fixes what it carries — learner, skill, trainer, date completed — and
that it is produced automatically the moment the trainer marks the session
complete, with no manual step by anybody.

"Done" for the learner is one of two things, and the page has to serve both
equally well: they read it on screen and close the tab, or they print it — or
save it as a PDF through the browser's print dialog — and keep it.

This screen exists because closing **OQ-L5** on 2026-08-28 settled the medium as
_a printable page in the portal_ rather than a generated PDF or an externally
shareable link, and named the page SCR-010. SCR-008 owns the entry point (its
ST-27 and ST-28); this spec owns the page that opens.

### Why this is a separate screen and not a dialog on SCR-008

A dialog cannot be printed on its own, cannot be bookmarked, and cannot be
reopened months later out of a browser history — and all three are things people
actually do with a certificate. A separate tab at a stable URL gets all three for
free. It also costs something, worth naming rather than discovering later: a URL
that can be reopened is a URL that can be reopened by the wrong person, or after
signing out, which is why ST-06 and ST-08 below exist at all. A dialog would have
had neither problem and none of the three capabilities.

## Layout

One column, centred, on a page that is deliberately **not** the application.
No app header, no navigation, no bell. The tab holds a document.

```
new browser tab — /certificate/<session-id>

┌────────────────────────────────────────────────┐
│  ← My learning                     [ Print ]   │  toolbar — screen only,
├────────────────────────────────────────────────┤  never printed
│                                                │
│       ──────────────────────────────────       │
│                                                │
│            Certificate of completion           │
│                                                │
│                                                │
│                  Joy Joshua                    │  the largest thing
│                                                │  on the page
│            has completed a session in          │
│                                                │
│              Kubernetes Basics                 │
│                                                │
│                                                │
│              taught by Priya Menon             │
│                on 26 August 2026               │
│                                                │
│       ──────────────────────────────────       │
│             Employee Skill Exchange            │
│                                                │
└────────────────────────────────────────────────┘
```

The four facts REQ-024 names are joined by ordinary English — "has completed a
session in", "taught by", "on". That phrasing is doing accessibility work rather
than decoration: every fact is labelled in reading order without a single
visually-hidden label, so what a screen reader announces and what a sighted
reader sees are the same sentence.

## States

### ST-01 Opening

- **When** the tab has loaded and the certificate is being fetched
- **Shows** the toolbar with **Print** present but disabled, and a centred spinner in the space the certificate will occupy. Nothing of the certificate is drawn in skeleton form — a half-drawn certificate with placeholder names is a worse thing to glimpse for half a second than empty space.
- **Can do** nothing; wait, or close the tab

### ST-02 The certificate

- **When** the certificate loaded and belongs to the signed-in employee (REQ-024)
- **Shows** the four facts in the layout above: the learner's name at display size as the visual centre, the skill named beneath it, the trainer and the completion date at the foot, framed by a rule above and below. The toolbar carries **Print** and a link back to **My learning**. Rendered in the portal's own type and colour tokens, in whichever theme the reader's system asks for.
- **Can do** print (→ ST-03); go back to My learning, which navigates this tab to SCR-008; close the tab

### ST-03 Printed

- **When** the reader activates **Print**, or their browser's own print command
- **Shows** the certificate alone: the toolbar removed, the page background forced to white and the text to near-black **regardless of the reader's theme** — a certificate printed from dark mode must not arrive as a black rectangle — with the rules kept, on one page at A4 and at Letter without the reader adjusting scale.
- **Can do** whatever their print dialog offers, including **Save as PDF**, which is how a learner who wants a file gets one. The portal does not generate the file; the browser does.
- **Note** the browser may add its own header and footer carrying the page URL and today's date. That is a browser setting, outside anything this page controls, and the layout is designed to survive it — nothing sits so close to the page edge that a browser header collides with it.

### ST-04 A long name or a long skill

- **When** the learner's name, the skill, or the trainer's name is too long for one line at its display size
- **Shows** the value wrapping to a second line and stepping down one type size, still centred. It is **never** truncated and never given an ellipsis: a certificate that abbreviates the name of the person it is about has failed at its only job. Three lines is the point at which the size steps down again.
- **Can do** everything ST-02 allows
- **Why this is numbered** skill names come from a central catalog whose entries nobody on this project controls the length of (SCR-005), and employee names come from provisioned accounts. Neither is bounded by any rule anybody has written, so overflow is the ordinary case rather than the edge case.

### ST-05 Not ready yet

- **When** the session is complete but the certificate has not been produced
- **Shows** the certificate replaced by an alert-icon panel: "This certificate isn't ready yet." and beneath it "It's generated automatically — this usually takes a moment.", with **Try again**. **Print** is not shown at all rather than shown disabled; there is nothing to print.
- **Can do** retry; go back to My learning
- **Note** the copy blames the generation and never suggests the learner do something to earn it, because REQ-024 promises the certificate is automatic with no manual step — which makes its absence a system failure. This is the same reasoning, and deliberately close to the same words, as SCR-008's ST-28, which is the state the learner would have seen a moment earlier.

### ST-06 Not available to you

- **When** the certificate does not exist, **or** it exists and belongs to another employee
- **Shows** one message for both cases: an alert-icon panel reading "We can't show you this certificate." and "It may not exist, or it may not be yours.", with a link to **My learning**. No **Print**, no retry — retrying will not change either answer.
- **Can do** go to My learning
- **Why the two cases are one state** telling somebody "this certificate exists but is not yours" confirms that a particular colleague completed a particular session, to anyone able to type numbers into a URL. That is a disclosure. SCR-002 already established the same-message-either-way pattern in this portal for the same reason, so this screen follows it rather than inventing a second philosophy. The cost is real and worth naming: a learner who has genuinely mistyped their own URL gets a vaguer message than we could have given them.

### ST-07 Couldn't load it

- **When** the fetch fails — network, timeout, or server error
- **Shows** an alert-icon panel reading "Couldn't load this certificate." and "Check your connection and try again.", with **Try again**. Distinct from ST-05 and ST-06 because this one says nothing about whether the certificate exists; it says the portal could not find out.
- **Can do** retry (→ ST-01); go to My learning

### ST-08 Signed out

- **When** the tab is opened with no valid session — typically a bookmarked or re-opened URL, days later
- **Shows** the certificate replaced by a panel reading "Please sign in to see this certificate.", with a **Sign in** action leading to SCR-001 and carrying a return destination, so signing in lands the employee back on this certificate rather than on the dashboard.
- **Can do** sign in and return here
- **Why this state exists** a stable URL is both the reason the page is worth having and the reason this state is unavoidable. It is kept separate from ST-06 on purpose: "sign in" is an instruction the reader can act on, and folding it into "we can't show you this" would waste the one case where we know exactly what would fix it.

## Components

| Component     | Preview                                                | States covered               |
| ------------- | ------------------------------------------------------ | ---------------------------- |
| `certificate` | `inception/design/components/certificate/preview.html` | ST-01 to ST-08               |
| `button`      | `inception/design/components/button/preview.html`      | Print, Try again, Sign in    |
| `alert`       | `inception/design/components/alert/preview.html`       | the panels in ST-05 to ST-08 |
| `spinner`     | `inception/design/components/spinner/preview.html`     | ST-01                        |

## Interaction and accessibility

- **Keyboard:** tab order is **My learning** → **Print** → the certificate body (not focusable) — three stops at most on the default state, which is rather the point of a document. In the error states the single action is the first and only stop. Print is reachable by the browser's own shortcut whether or not the button has focus.
- **Focus:** visible ring (`--c-focus-ring`) on both toolbar controls and on every action in the error panels. After **Try again**, focus stays on the button through ST-01 and moves to the page heading when ST-02 renders, so a screen-reader user is told the certificate arrived rather than being left on a button that has silently vanished.
- **Heading structure:** one `h1`, "Certificate of completion". The learner's name is deliberately not a heading — it is the subject of a sentence, and marking it as one would produce a document outline that reads as a list of employee names.
- **Non-colour signalling (NFR-003 in spirit, and the framework's standing rule):** every one of ST-05 to ST-08 carries an icon **and** a full sentence. Nothing on this screen distinguishes anything by hue. Printed in one colour on a monochrome printer the certificate loses nothing at all — a fair test of the chosen design, and one the formal-award alternative would have failed.
- **Announcements:** ST-01 announces "Loading certificate" politely; ST-02 moves focus to the heading; ST-05 to ST-08 announce their message assertively, since each replaces the thing the reader came for.
- **Responsive (NFR-004):** the same single column at every width. On a phone the display type steps down and the rules run closer to the edges; nothing reflows, because there is only one column to reflow. The toolbar keeps both controls side by side — two short labels fit the narrowest supported width.
- **Theme:** on screen, the reader's theme. On paper, always light — see ST-03.

## Structural decisions

| Decision                                                                   | Rationale                                                                                                                                                                                                                    | Alternative rejected                                                                                                                                                                                                                                       |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Restrained centred layout — rules above and below, no decorative border     | Chosen by the designer on 2026-08-31. It reads as a genuine record, prints legibly in one colour, and is built entirely from tokens that already exist                                                                       | A formal landscape award with a decorative double border: it needs ornament the design system has no tokens for, and a raw value in a component is exactly what the token export exists to prevent. A plain label/value card was also rejected — cheapest to build, but reads as a receipt |
| No app header and no navigation on this page                               | A certificate is a document, not a screen of the application. Chrome around it would either print or need suppressing, and it invites the reader to treat the tab as somewhere they can work                                 | Rendering it inside the app shell                                                                                                                                                                                                                          |
| The four facts joined by ordinary English rather than set as labelled rows  | It puts every fact in reading order with its own label, so the screen-reader experience and the visual one are the same sentence, with no visually-hidden text to keep in step                                               | `Learner: / Skill: / Trainer: / Date:` rows                                                                                                                                                                                                                |
| A stable URL per completed session, opened in a new tab                    | Makes the certificate bookmarkable, reopenable and printable on its own — the three things people actually do with one. The cost is ST-06 and ST-08, accepted deliberately                                                   | A modal on SCR-008, which has none of the three                                                                                                                                                                                                            |
| One message for "doesn't exist" and for "isn't yours" (ST-06)               | Distinguishing them confirms to a stranger that a named colleague completed a named session. Follows the pattern SCR-002 already set in this portal                                                                          | Separate, more helpful messages                                                                                                                                                                                                                            |
| Print produces light-on-white whatever the reader's theme                   | A certificate printed from dark mode would otherwise arrive as a black rectangle, and the reader would not find out until after it was printed                                                                               | Honouring the reader's theme on paper                                                                                                                                                                                                                      |
| Names and skills wrap and step down rather than truncate (ST-04)             | The document's whole purpose is naming a person correctly                                                                                                                                                                    | Ellipsis, as the application's list rows use                                                                                                                                                                                                               |
| No **Save as PDF** button of our own                                        | The browser's print dialog already offers it on every platform this portal supports. A button of ours would either duplicate that or imply server-side generation, which closing OQ-L5 explicitly ruled out                  | A dedicated download action                                                                                                                                                                                                                                |

## Conflicts and open questions

| #     | Conflict / question                                                                                                                                                                                                                          | Working default in this spec                                                                                                                              | Owner                   | Status                    |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- | ------------------------- |
| OQ-C1 | **Who may open a certificate?** REQ-024 generates it "for the learner" and says nothing about the trainer — who is named on it, and might reasonably want to see what their name is on. ST-06 is built on the answer                          | Learner only. The trainer sees the completed session in their own history (SCR-007, REQ-027) but has no route to the certificate                            | BA (`/ba`)              | open                      |
| OQ-C2 | **Does the certificate carry a company name, a logo, or a signature?** REQ-024 names four facts and no issuer. A certificate with no issuer at all is a strange object, but adding one is a brand commitment nobody has made — and a signature would be a claim about who is attesting | The portal's own name, "Employee Skill Exchange", set quietly at the foot. No logo, no signature, no letterhead                                             | BA + whoever owns brand | open                      |
| OQ-C3 | **Is there a certificate reference number?** Nothing in REQ-024 asks for one                                                                                                                                                                 | No reference number. A number that cannot be checked anywhere implies a verification service that does not exist, which is worse than no number             | BA (`/ba`)              | open                      |
| OQ-C4 | **Which date is "date completed", and in whose timezone?** For colleagues in different offices the trainer's "26 August" can be the learner's "27 August", and the certificate is the learner's document                                      | The learner's local date, formatted in full — "26 August 2026" — never numeric, which is read differently on either side of the Atlantic                    | BA + Architect          | open                      |
| OQ-C5 | **Can a certificate be revoked?** This page assumes not, because SCR-007's OQ-T10 defaults to "mark complete is irreversible". If that default is overturned, a certificate can outlive the session it attests to and this screen needs a revoked state | No revoked state. The certificate is permanent, following OQ-T10                                                                                            | BA (`/ba`)              | open — depends on OQ-T10  |
| OQ-C6 | **What happens to a certificate when an employee leaves?** Their account goes; the certificate names them, and the trainer's history still references the session                                                                             | Not designed. Raised because REQ-024 creates a durable record about a person and no requirement covers the end of their account                             | BA + IT                 | open                      |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via
Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the
numbering is the checklist. Two of them want drawing twice: **ST-02** in both
themes, and **ST-03** at A4 as it prints. **ST-04** is worth drawing with a
deliberately punishing name — it is the state most likely to be skipped and one
of the most likely to be seen.

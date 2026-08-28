# SCR-005 — My profile and skills

> Approval = Gate 1 review of this file's PR. A state is "designed" when a component preview renders it and marks it `<!-- @state SCR-005/ST-## -->`.

|                  |                                                                                     |
| ---------------- | ----------------------------------------------------------------------------------- |
| **Serves**       | (no story yet — stories are drafted after design)                                   |
| **Traces to**    | REQ-007, REQ-008, REQ-009, REQ-010, REQ-011, REQ-013, NFR-004                        |
| **Surface**      | `apps/ui` `features/profile` — `/profile`, with `/profile#skills` for the skills half |
| **Primary user** | Employee — the same person acting as Learner and Trainer on one account              |
| **Status**       | draft — awaiting designer review                                                     |

## Purpose

The screen where an employee says who they are and what they can teach or want to learn. It carries two jobs the requirements keep separate but an employee does together: the profile fields REQ-007 names (description, department, designation, picture) and the skill list REQ-009–REQ-011 govern.

"Done" is one of two things. For a new employee it is a single setup pass — fill in the profile, list one skill, become findable. For everyone after that it is a small correction: a new designation, a proficiency raised from Intermediate to Expert, a skill they no longer want to teach.

This is also the screen where an employee becomes visible to the rest of the organization. Listing a skill as teachable puts them into trainer search results immediately, with no approval step (REQ-013, BR-001.1). That is a consequence worth stating on screen at the moment it happens, not a detail to discover later — see ST-13 and ST-17.

### Requirements this screen touches but does not fully serve

Recorded so a later audit doesn't read the manifest as "covered":

| Requirement                               | What this screen does                                                                | Where the rest lives                                                                     |
| ----------------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| REQ-025 (trainer's average rating)        | Shows the employee their **own** average rating and rating count, read-only           | The rating as a colleague sees it — trainer profile and search results — not yet designed |
| REQ-012 (search filterable by department) | Sets the department value that filter reads                                          | The search screen itself — not yet designed                                               |
| REQ-014 (trainer publishes time slots)    | Nothing. Listing a teachable skill is a precondition for publishing slots against it | An availability screen — not yet designed                                                 |

REQ-008 is served in full here, in the only way the constraints allow: the department is chosen from a fixed list, and this screen deliberately has no way to edit that list (BRD §8 — IT maintains it directly, no in-app editor).

## Layout

One shell header, then a single constrained centre column: the profile card, then the two skills cards. Profile first because it is what an employee sets once and what identifies them; skills second because it is what they come back to.

The skills half sits under an `#skills` anchor, because the dashboard (SCR-004) links to it twice — "My skills" in the header, and "Edit →" on each of its skills cards. Both land on this screen, scrolled to that section.

```
┌──────────────────────────────────────────────────────────┐
│ SkillEx   [ main navigation ]                  🔔3  (JJ)▾│  ← app-header
├──────────────────────────────────────────────────────────┤
│                                                          │
│   My profile                                             │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │  ┌────┐  Joy Joshua                            │     │
│   │  │ JJ │  joy_j@trigent.com                     │     │
│   │  └────┘  ★ 4.6 average · 12 ratings            │     │
│   │  Change photo   Remove                         │     │
│   │  ───────────────────────────────────────────── │     │
│   │  About you                                     │     │
│   │  ┌──────────────────────────────────────────┐  │     │
│   │  │ I run the finance reporting team and     │  │     │
│   │  │ enjoy teaching Excel to anyone who asks. │  │     │
│   │  └──────────────────────────────────────────┘  │     │
│   │                                     118 / 500  │     │
│   │                                                │     │
│   │  Department            Designation             │     │
│   │  ┌─────────────────┐  ┌────────────────────┐   │     │
│   │  │ Finance       ▾ │  │ Senior Analyst     │   │     │
│   │  └─────────────────┘  └────────────────────┘   │     │
│   │  ───────────────────────────────────────────── │     │
│   │              [ Save changes ]   Cancel         │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│   Skills                                    ← #skills    │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Skills I teach (3)            [ + Add a skill ]│     │
│   │ [● Excel · Expert            Edit ✕]           │     │
│   │ [● Power BI · Intermediate   Edit ✕]           │     │
│   │ [● Technical writing · Beginner  Edit ✕]       │     │
│   │ Listed here, colleagues can find and book you. │     │
│   └────────────────────────────────────────────────┘     │
│                                                          │
│   ┌────────────────────────────────────────────────┐     │
│   │ Skills I'm learning (1)       [ + Add a skill ]│     │
│   │ [◆ Python · Beginner         Edit ✕]           │     │
│   └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

**Two skills cards, not one list with a toggle.** Teach and learn are the same shape of data (skill + proficiency) with completely different consequences — one makes you findable, the other does not. Two labelled cards state that difference structurally, and they match how the dashboard already presents the pair.

**Read-only fields inside an editable card.** Name, email and average rating sit in the card's header above the divider and are not editable. REQ-007 enumerates exactly four editable fields and none of these is among them. The divider is what separates "about you, fixed" from "about you, yours to change" — see OQ-E6 and OQ-E8.

**Adding and editing a skill happens in a dialog, not inline.** Reasoning in Structural decisions.

## States

Twenty-one, which is a direct consequence of putting the profile form and the skill list on one screen: this screen owns two independent save paths, and each needs its own in-flight, success and failure. Grouped below for reading, numbered flat for building.

**Loading and whole-screen states**

### ST-01 Default

- **When** the employee opens the screen, the data has loaded, the profile has at least one field filled, and at least one skills list has an entry
- **Shows** the app-header inherited from SCR-004; the page title; the profile card with its read-only header (avatar or initials, name, email, and the average rating if the employee has been rated) and its four fields carrying their saved values; **Save changes** disabled because nothing has been touched; then the two skills cards, each with its count, its chips, and an **Add a skill** button. The teach card carries one line of standing copy — "Listed here, colleagues can find and book you." — because REQ-013 makes that list public the moment it has an entry.
- **Can do** edit any of the four profile fields (→ ST-06); change or remove the photo (→ ST-11); add a skill (→ ST-13); edit a chip (→ ST-19); remove a chip (→ ST-20); use the header

### ST-02 Loading

- **When** the screen has mounted and the profile-and-skills request is in flight
- **Shows** the app-header in full — it needs none of this data — the page title, then the profile card and both skills cards as placeholders: card headings and field labels are real, values are muted skeleton blocks at the height the real content will occupy. Counts are omitted from the skills headings, because the count is not yet known. **Save changes** and **Add a skill** are present but disabled.
- **Can do** use the header. Nothing in the page body is interactive.
- **Why skeletons and not a spinner:** the labels are known before the values are, so the form does not visibly re-flow when data lands — the same reasoning as SCR-004's ST-02.

### ST-03 Nothing set up yet (first visit)

- **When** the data loaded and the employee has no profile fields filled **and** no skills in either list — someone who has just set their password and signed in for the first time
- **Shows** the app-header and title, then a short setup panel above the profile card: one line saying what filling this in gets them ("Colleagues find you by your skills and your department"), and the profile card rendered with every field empty, each showing its placeholder. Both skills cards render in their empty form (ST-04) beneath. The panel is guidance, not a wizard — nothing is gated, and the employee may fill in one field and leave.
- **Can do** everything ST-01 allows
- **Why a panel and not a step-by-step wizard:** there are four optional fields and one list. A wizard implies a required order and a finish line, neither of which any requirement establishes.

### ST-04 A skills list with nothing in it

- **When** the data loaded and one or both skills lists are empty
- **Shows** that card keeping its heading and its **Add a skill** button, with the chips replaced by one line:

  | List                | Copy                                                                    |
  | ------------------- | ----------------------------------------------------------------------- |
  | Skills I teach      | "Nothing listed yet. Add a skill and colleagues can start booking you." |
  | Skills I'm learning | "Nothing listed yet. Add what you'd like to learn."                     |

- **Can do** add a skill (→ ST-13)
- **Why the card stays:** removing an empty list would change the screen's shape between visits, and the empty teach list is exactly where the "you become findable" message earns its place.

### ST-05 Could not load

- **When** the profile-and-skills request fails — timeout, server error, no connection
- **Shows** the app-header as normal and, in place of the whole page body, one error panel: alert icon + "We couldn't load your profile." + a **Try again** button. The header stays usable so the employee is never trapped.
- **Can do** retry (→ ST-02); use the header
- **Why one error, not one per card:** the screen loads as a single request, so there is exactly one thing that can fail — the same reasoning as SCR-004's ST-05.

**The profile form**

### ST-06 Unsaved changes

- **When** the employee has changed any of the four editable fields and the value differs from what was loaded
- **Shows** **Save changes** enabled and given primary emphasis; **Cancel** enabled; a quiet line beside them reading "Unsaved changes". The description's character counter updates live. If the employee tries to leave — following a header destination, or closing the tab — they are asked to confirm, naming what is at stake: "You have unsaved profile changes. Leave without saving?"
- **Can do** save (→ ST-07); cancel, which restores every field to its loaded value and returns to ST-01 with no network call; keep editing
- **Note** this state exists because of the single-Save decision below. It is the cost of that choice, stated rather than hidden. Cancel is deliberately not confirm-guarded while Save is one click away.

### ST-07 Saving

- **When** the employee has pressed **Save changes** and validation passed
- **Shows** **Save changes** replaced by a spinner labelled "Saving…", both it and **Cancel** disabled, and all four fields set read-only so a mid-flight edit cannot be silently discarded. The skills cards below stay fully interactive — the two save paths are independent, and a slow profile save must not block adding a skill.
- **Can do** nothing in the profile card; everything below it still works

### ST-08 Saved

- **When** the save succeeds (REQ-007)
- **Shows** the fields re-enabled with their new values as the new baseline, **Save changes** disabled again, and a confirmation beside it — tick icon + "Saved" — held for a few seconds and then faded out. If the department changed, one extra line: "Colleagues filtering by department will now see you under Finance." (REQ-012's filter reads this field, so the change has a consequence worth naming.)
- **Can do** everything ST-01 allows
- **Why inline and not a page banner:** consistent with SCR-004's ST-07 — the confirmation belongs where the action happened, and a banner at the top of the page is off-screen on a phone.

### ST-09 Something needs fixing before saving

- **When** the employee presses **Save changes** and one or more fields fail validation locally — today that means only a length cap exceeded (see OQ-E1 and OQ-E3; no requirement makes any field mandatory)
- **Shows** each offending field marked: an error-toned border **and** an icon **and** a line beneath it naming the problem and the limit — "Keep this under 500 characters. You have 540." The counter switches to the error tone and keeps its numerals. No network call is made. Because one Save can surface more than one problem at once, a summary line sits above the button row when there are two or more — "2 fields need attention" — and focus moves to the first offending field.
- **Can do** fix the fields and save again; cancel
- **Note** the summary line exists only because of the single-Save decision. With per-field saving there could never be more than one error on screen at a time.

### ST-10 Save failed

- **When** the save request fails — server error, timeout, no connection
- **Shows** an inline error above the button row: alert icon + "Couldn't save your profile. Please try again." Every field keeps the value the employee typed — nothing is reverted, and the screen does not drop back to ST-01. **Save changes** and **Cancel** are re-enabled, so the screen is back in ST-06. The wording says the _saving_ failed, so nobody reads it as their input being rejected.
- **Can do** retry; keep editing; cancel

### ST-11 Changing the photo

- **When** the employee chooses **Change photo** and picks a file
- **Shows** the chosen image immediately in the avatar position as a local preview, with a spinner over it and the caption "Uploading…", while **Change photo** and **Remove** are disabled. The form's other fields and its Save are unaffected — the photo saves on its own, not with the form, because a file upload has a different duration and a different failure mode from four short text fields. On success the spinner clears and "Photo updated" is announced beside the avatar. **Remove** returns to the initials avatar after a confirm, since a deletion cannot be undone by pressing Cancel.
- **Can do** nothing to the avatar while in flight; the rest of the screen still works
- **Note** the initials avatar is the permanent fallback, not a placeholder waiting to be replaced: REQ-007 makes the picture optional, so every screen showing an employee must render well without one.

### ST-12 That photo can't be used

- **When** the chosen file is rejected — wrong type, larger than the size cap (OQ-E2), or the upload itself fails
- **Shows** the avatar restored to whatever it was before, and an error line beneath the photo actions naming the actual reason and the rule: "That file is 4.2 MB. Please choose a JPG, PNG or WebP under 2 MB." A generic "invalid file" is useless — the employee cannot act on it.
- **Can do** choose a different file; leave the photo as it was

**The skills list**

### ST-13 Adding a skill

- **When** the employee presses **Add a skill** on either card
- **Shows** a dialog titled "Add a skill", with the list it was opened from pre-selected: a **Teach / Learn** choice (two radio options labelled with what each means — "I can teach this" / "I want to learn this"); a **Skill** field that filters the IT-maintained catalog as the employee types and accepts no free text (REQ-009); a **Proficiency** choice of Beginner, Intermediate or Expert, spelled out, with none pre-selected (REQ-010); and **Add skill** / **Cancel**. **Add skill** is disabled until both a catalog skill and a proficiency are chosen. When Teach is selected, one line under the choice states the consequence: "Colleagues will be able to find and book you for this straight away." (REQ-013, BR-001.1)
- **Can do** search and pick a skill; set teach/learn and proficiency; add (→ ST-16); cancel, which closes the dialog and changes nothing
- **Note** the catalog offers no "add a new skill" option, by constraint (BRD §8). What an employee sees when their skill isn't in the list is ST-14, and what that copy should say is OQ-E5.

### ST-14 That skill isn't in the catalog

- **When** the employee's search matches nothing in the catalog
- **Shows** the skill field's result list replaced by one line — 'No skill matches "kubernetes".' — plus a second line explaining that the catalog is maintained centrally and how to ask for an addition; **wording and route are TBD, OQ-E5**. **Add skill** stays disabled; free text is never accepted as a skill.
- **Can do** search for something else; cancel
- **Why this state matters more than it looks:** RISK-001 in the BRD is precisely this moment. It is the point where an employee with a skill the organization wants either gets a route or gives up, and no requirement currently gives them one.

### ST-15 You've already listed that

- **When** the chosen skill is already in the list the employee is adding it to
- **Shows** an inline note under the skill field — "You already teach Excel at Expert level. Edit it instead?" — with **Add skill** disabled and the offer to switch straight to editing the existing entry (→ ST-19). The same skill in the _other_ list is not a duplicate and is not blocked: REQ-009 says a skill may be listed "to teach and/or to learn", so an employee may teach Excel and be learning Excel at once. See OQ-E4.
- **Can do** switch to editing the existing entry; pick a different skill; cancel

### ST-16 Saving the skill

- **When** the employee presses **Add skill**, or **Save changes** from ST-19
- **Shows** the dialog's primary button replaced by a spinner and relabelled "Adding…" / "Saving…", every control in the dialog disabled, and the dialog held open. Escape and the close button are inert while in flight, so a half-committed change cannot be dismissed.
- **Can do** nothing until it resolves

### ST-17 Skill added

- **When** the add or edit succeeds (REQ-009, REQ-010, REQ-011)
- **Shows** the dialog closes; the new chip appears in the right card with the count incremented and the chip briefly highlighted so the eye finds it; focus returns to the **Add a skill** button that opened the dialog. A confirmation line sits in that card for a few seconds. For a **teachable** skill it names the consequence rather than the action: "Excel added — colleagues can now find and book you for this." For a learning skill it is simply "Python added." Adding the first teachable skill replaces the teach card's ST-04 copy with the standing line from ST-01.
- **Can do** everything ST-01 allows
- **Why the wording differs between the two lists:** they have different consequences. Listing something to teach makes the employee publicly findable with no approval step (BR-001.1); listing something to learn does not. Identical copy would flatten the one difference the employee most needs to understand.

### ST-18 Couldn't save the skill

- **When** the add or edit request fails
- **Shows** the dialog stays open with every choice the employee made intact, controls re-enabled, and an error at the top of the dialog: alert icon + "Couldn't add that skill. Please try again." Nothing is added to the list behind the dialog — the list never shows a chip that isn't saved.
- **Can do** retry; cancel

### ST-19 Editing a listed skill

- **When** the employee presses **Edit** on a chip, or takes the offer in ST-15
- **Shows** the same dialog as ST-13, titled "Edit skill", with the skill name shown as fixed text rather than a search field — changing which skill it is would be removing one and adding another, not an edit — and the proficiency and teach/learn choice carrying their current values. The primary button reads **Save changes** and stays disabled until something actually differs. Moving an entry from Teach to Learn is allowed here and moves the chip between cards on save; if that would collide with an existing entry in the target list, ST-15's duplicate note applies.
- **Can do** change proficiency or list; save (→ ST-16); cancel

### ST-20 Removing a skill

- **When** the employee presses **✕** on a chip
- **Shows** a confirm, because removal cannot be undone by a Cancel elsewhere on the screen. It names the specific entry — "Remove Excel from the skills you teach?" — and for a **teachable** skill adds the consequence: "Colleagues will no longer find you for this skill." **Remove** and **Cancel**; while removing, both are disabled and **Remove** shows a spinner. On success the chip fades out, the count decrements, focus moves to the card's heading rather than being lost, and the card falls to ST-04 if that was its last entry.
- **Can do** confirm; cancel
- **Blocked by conflict C-1.** What must happen when the skill being removed has pending booking requests or upcoming approved sessions against it is not defined by any requirement, and REQ-011's unconditional "may remove" pulls against the confirmed sessions REQ-021 creates. The working default drawn here is the conservative one: removal is refused while such requests or sessions exist, and the confirm is replaced by an explanation naming the count — "You have 1 upcoming session and 2 pending requests for Excel. Cancel or complete these before removing the skill." That is a placeholder for a decision the BA owns; it is not a rule this spec is entitled to set.

### ST-21 Couldn't remove the skill

- **When** the removal request fails
- **Shows** the chip still present and interactive, the confirm closed, and an inline error in that card: alert icon + "Couldn't remove Excel. Please try again." The chip is never optimistically removed — an employee must never believe they have stopped being bookable for a skill when they have not.
- **Can do** retry; carry on

## Components

| Component        | Preview                                                   | States covered                                                                                                                            |
| ---------------- | --------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `profile-form`   | `inception/design/components/profile-form/preview.html`   | default, loading skeleton, load error, unsaved, saving, saved, invalid, save failed — ST-01, ST-02, ST-05, ST-06, ST-07, ST-08, ST-09, ST-10 |
| `avatar-picker`  | `inception/design/components/avatar-picker/preview.html`  | initials fallback, photo set, uploading, rejected — ST-11, ST-12                                                                            |
| `skills-section` | `inception/design/components/skills-section/preview.html` | both lists populated, empty list, first visit, skill added, remove failed — ST-01, ST-03, ST-04, ST-17, ST-21                                |
| `skill-editor`   | `inception/design/components/skill-editor/preview.html`   | add, no catalog match, duplicate, saving, save failed, edit, confirm remove (and its refused variant) — ST-13, ST-14, ST-15, ST-16, ST-18, ST-19, ST-20 |
| `app-header`     | `inception/design/components/app-header/preview.html`     | the shell inherited from SCR-004, unchanged                                                                                                |
| `card`           | `inception/design/components/card/preview.html`           | the card shell all three sections sit in                                                                                                   |
| `text-input`     | `inception/design/components/text-input/preview.html`     | the field, its label, its error and its helper text                                                                                        |
| `skill-chip`     | `inception/design/components/skill-chip/preview.html`     | teach and learn variants at each level, and the removable form this screen uses                                                             |
| `button`         | `inception/design/components/button/preview.html`         | primary, secondary, loading, disabled                                                                                                      |
| `alert`          | `inception/design/components/alert/preview.html`          | error and success variants behind ST-05, ST-10, ST-18, ST-21                                                                                |
| `spinner`        | `inception/design/components/spinner/preview.html`        | the in-flight indicator for ST-07, ST-11, ST-16, ST-20                                                                                     |
| `empty-state`    | `inception/design/components/empty-state/preview.html`    | the setup panel shape reused by ST-03                                                                                                      |

Components are declared in `knowledge/traceability/manifest.json` under this screen; `aidlc-check` proves each preview file exists and that every state above is rendered.

## Interaction and accessibility

- **Keyboard:** the header keeps SCR-004's order. Then: page title (not focusable) → Change photo → Remove photo → description → department → designation → Save changes → Cancel → Add a skill (teach) → each teach chip's Edit then ✕ in visual order → Add a skill (learn) → each learn chip's Edit then ✕. Every chip action is a real button, reachable without a mouse and without a hover.
- **Dialog:** ST-13/ST-19 open a modal — `role="dialog"`, `aria-modal="true"`, labelled by its title. Focus moves to the first control on open and is trapped while open; Escape closes it (inert during ST-16); on close, focus returns to the exact button that opened it. The confirm in ST-20 follows the same rules with focus starting on **Cancel**, not **Remove**, so a stray Enter cannot delete a skill.
- **Focus after an action:** ST-17 returns focus to the **Add a skill** button; a successful ST-20 removal moves focus to that card's heading, because the element that had focus no longer exists. Focus is never dropped to `<body>`.
- **Focus ring:** every interactive element shows the `--focus-ring` outline on keyboard focus, chips included.
- **Non-colour signalling:** proficiency is always spelled out — Beginner, Intermediate, Expert — never a colour or a count of dots. Teach and learn are distinguished by icon _shape_ (filled circle vs diamond) as well as tone, per the existing `skill-chip`. Every field error pairs its border tone with an icon and a text line; a border colour alone is invisible to a colour-blind reader and absent to a screen reader. The average rating shows its numeral and count ("4.6 average · 12 ratings"), not stars alone.
- **Announcements:** the profile card and each skills card are landmark regions labelled by their headings. ST-02's skeletons are `aria-busy="true"`. ST-08's "Saved", ST-17's confirmation and ST-11's "Photo updated" are announced `aria-live="polite"` — the employee caused them and is looking at the screen. ST-09's field errors are tied to their inputs with `aria-invalid` and `aria-describedby`, and its summary line is `aria-live="assertive"`, since the press produced no visible movement. ST-05's panel is `aria-live="assertive"`. The description counter is silent while typing and announced only when it crosses the limit — a counter that speaks every keystroke is unusable.
- **Responsive (NFR-004):** the centre column goes full width below the desktop breakpoint. Department and designation sit side by side on desktop and stack on mobile. Chips wrap rather than truncate — the catalog is IT-maintained and names can be long. The dialog becomes a full-height sheet on mobile with its actions pinned in view, so **Add skill** is never below the fold behind a keyboard. The **Save changes** row stays in the card's flow rather than sticking to the viewport; the profile card is short enough that a sticky bar would cost more than it saves.
- **Motion:** ST-17's chip highlight and ST-20's removal fade are both suppressed under `prefers-reduced-motion`.

## Structural decisions

| Decision                                                                                         | Rationale                                                                                                                                                                                                                                                                 | Alternative rejected                                                                                                                                                               |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Profile and skills on **one** screen, profile above skills                                       | Designer's decision, 2026-08-28. A new employee sets themselves up in one pass with no navigation, and there is one screen to build rather than two                                                                                                                        | Two screens — a cleaner match to the dashboard's two separate links, and shorter on a phone; the designer accepted the length in exchange for the single setup pass                  |
| The dashboard's "My skills" destination and its "Edit →" links all resolve to `/profile#skills`   | Follows from the decision above: those links must land somewhere, and the skills half of this screen is that somewhere. SCR-004 named its destinations without fixing their routes, so nothing in it is contradicted                                                        | A separate skills route — would require the two-screen split                                                                                                                        |
| **One Save** for the whole profile, with **Cancel** as the revert                                | Designer's decision, 2026-08-28. The familiar account-settings pattern, and the better shape for the first-visit pass where all four fields are filled at once                                                                                                             | Per-field editing — no unsaved-changes trap and no multi-error case; the designer chose the setup pass over that safety. ST-06 and ST-09's summary line are the cost, made explicit |
| The photo saves **immediately**, outside the form's Save                                         | A file upload takes a different amount of time and fails in different ways from four short text fields. Bundling it would make one Save that can half-succeed, and would hold a spinner over the whole form for the length of an upload                                     | Photo as a fifth form field committed by Save — one failure mode covering two very different operations                                                                              |
| Name, email and average rating are read-only, above a divider                                    | REQ-007 enumerates exactly four editable fields; none of these is one. Putting them above a divider makes "fixed" a structural statement rather than a disabled-looking input                                                                                              | Rendering them as disabled inputs — reads as broken; or omitting them — the employee could not confirm which account they are looking at                                             |
| Skill add and edit happen in a **dialog**                                                        | Adding a skill is three coupled choices (list, catalog skill, proficiency) that are only valid together, and the catalog needs a search field. A dialog commits them as one unit, is reused unchanged for edit, and keeps an already-long screen from growing a second form | An inline expanding row on each card — no focus trap to specify and no modal on mobile, but it duplicates the form twice on screen and pushes the chips around while open             |
| The department is a fixed-list select with no way to add to it                                   | REQ-008 and BRD §8: IT maintains the list directly, no in-app editor. A select with no escape hatch is the honest rendering of that constraint                                                                                                                             | A combo box accepting free text — would silently create departments the REQ-012 search filter cannot rely on                                                                         |
| The catalog field accepts no free text                                                           | REQ-009 requires the skill to come from the predefined catalog. Free text would break trainer search, which matches on catalog skills                                                                                                                                      | Free text with a "we'll add it" promise — a promise no requirement backs                                                                                                             |
| Adding a teachable skill states the consequence, on the way in (ST-13) and on the way out (ST-17) | REQ-013 and BR-001.1 make the employee publicly findable with no approval step. That is the most surprising thing this screen does, and the employee should meet it before it happens, not discover it from a booking request                                               | A neutral "Skill added" for both lists — identical copy for two outcomes that differ in exactly the way that matters                                                                 |
| Removal is confirmed; changing proficiency is not                                                | Removal cannot be undone from this screen and, for a teachable skill, changes who can reach the employee. A proficiency change is one edit away from being changed back                                                                                                     | Confirming every change — trains people to dismiss dialogs unread, the same reason SCR-004 has no log-out confirm                                                                    |
| A removal is never optimistic (ST-21)                                                            | The failure that matters is an employee believing they are no longer bookable for a skill when they still are. Showing the chip until the server agrees it is gone is the only version that cannot lie                                                                      | Optimistic removal with a rollback — faster, and wrong in exactly the case that costs someone a session                                                                              |
| One whole-screen loading state (ST-02) and one whole-screen load error (ST-05)                    | The screen loads in a single request, so one thing can fail — consistent with SCR-004                                                                                                                                                                                      | Per-card loading and error states, which would design failure modes the data flow cannot produce                                                                                     |
| Two save paths that don't block each other (ST-07, ST-16)                                        | The profile form and the skills lists are independent. A slow profile save that froze the skills cards would be a bug the design invented                                                                                                                                  | A screen-wide busy overlay — simpler to build, and it makes every save feel like the slowest one                                                                                     |

## Conflicts

One open row. Per the gate it blocks sign-off of the state it touches (ST-20); the rest of the screen is reviewable as it stands.

| #   | Conflict                                                                                                                                                                                                                                                                                                                                                                                                     | Between                               | Owner      | Status |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | ---------- | ------ |
| C-1 | REQ-011 lets an employee remove a listed skill unconditionally. REQ-015, REQ-017 and REQ-021 allow booking requests and confirmed one-to-one sessions to exist against that same skill. Nothing says what happens to them when the skill is removed, and both readings — remove anyway, or refuse while they exist — are business rules this spec cannot set. ST-20 draws the conservative one as a placeholder. | REQ-011 vs REQ-015, REQ-017, REQ-021 | BA (`/ba`) | open   |

## Open questions

Each has a working default, so none of these stops the frames being drawn — but each is someone else's call, and the answer may change a limit or a line of copy.

| #     | Question                                                                                                                                                                                                                    | Working default                                                                                                            | Owner         | Status                     |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------- | -------------------------- |
| OQ-E1 | Which profile fields are required? No requirement makes any of the four mandatory, yet REQ-012 lets colleagues filter trainers by department — a trainer with no department matches no department filter                       | All four optional; a trainer without a department simply does not appear under a department filter                          | BA (`/ba`)    | open                       |
| OQ-E2 | ~~Profile picture rules: allowed file types, maximum size, and whether a non-square image is cropped automatically or the employee positions it~~ | **Resolved 2026-08-28 — automatic centre-crop, no cropping tool.** JPG, PNG or WebP, centre-cropped to the circle the avatar occupies; the employee never positions the crop. 2 MB is the proposed cap and remains IT's to confirm — the number does not change any state on this screen, only the message ST-08 shows when a file is refused | IT + designer | closed (cap with IT) |
| OQ-E3 | ~~Length caps for the description and the designation~~ | **Resolved 2026-08-28 — 500 and 80.** 500 characters for the description, 80 for the designation. The counter appears as the limit is approached rather than sitting there from the first keystroke, so an empty field is an invitation rather than a quota | Designer | closed |
| OQ-E4 | REQ-009 says a skill may be listed "to teach **and/or** to learn". Read literally that permits the same skill in both lists at once. Intended, or should one exclude the other?                                               | Permitted in both; a duplicate is blocked only within the same list (ST-15)                                                | BA (`/ba`)    | open                       |
| OQ-E5 | What an employee is told when their skill isn't in the catalog (ST-14). RISK-001 names a "lightweight request-a-new-skill path to IT" as its mitigation, but no requirement creates one and BRD §8 rules out an in-app editor | A line saying the catalog is maintained centrally, with **no** in-app mechanism; the exact wording and route are TBD        | BA + IT       | open — copy is TBD         |
| OQ-E6 | Nothing in the BRD says where an employee's **name** comes from. There is no sign-up (BRD §2 — accounts are provisioned), REQ-007 does not list the name as editable, and REQ-024 puts that name on a certificate             | Read-only, sourced from the provisioned account; this screen offers no way to change it                                    | BA (`/ba`)    | open                       |
| OQ-E7 | No requirement covers changing your password while signed in. REQ-003 covers only the forgotten-password email flow, so an employee who knows their password has no way to change it                                          | Not on this screen; nothing is designed for it until a requirement exists                                                  | BA (`/ba`)    | open                       |
| OQ-E8 | ~~Should an employee's own average rating (REQ-025) appear on this screen at all, given it is the trainer's public number?~~ | **Resolved 2026-08-28 — yes, read-only.** Shown in the card header as the average with its rating count beside it, so a 5.0 from a single session is not read as a track record. No control to edit or hide it: it is what colleagues already see in search results (REQ-025), and a profile that concealed it would make the employee search for themselves to find out | Designer | closed |

## Designer handoff

Tokens: `inception/design/tokens.json` (W3C DTCG — importable into Figma via Tokens Studio, Penpot, and others). Draw one frame per `ST-##` above; the numbering is the checklist. ST-01, ST-03 and ST-13 are each worth two frames — desktop and mobile width — since NFR-004 is what the reflow has to prove, and ST-13's dialog becomes a sheet on mobile.

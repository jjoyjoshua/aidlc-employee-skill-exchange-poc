# Figma handoff — Employee Skill Exchange Portal

The design step (Gate 1, step 2) is approved and merged: 10 screens, 156 numbered
states, 28 components, palette settled on warm teal. This file is the bridge from
that specification to the Figma file where the frames get drawn.

**Direction matters here, and the two directions are not symmetric.**

| Direction | How it moves | Automated? |
| --- | --- | --- |
| Repo → Figma (the palette) | `tokens.json` imported via the Tokens Studio plugin | No — a human runs the import once |
| Figma → repo (the frames) | Figma Dev Mode MCP, read by `/ux` | Yes — `/ux` can read frames back and check them against the specs |

The Figma MCP server is **read-only**. It cannot create variables or push tokens
into a Figma file, so nobody should wait for it to do the import. What it does
buy us is the return trip: once frames exist, `/ux` reads them and reports which
of the 156 states are drawn, which are missing, and which use a colour that is
not in the token set.

## 1. Import the palette (once, by hand)

1. In the Figma file, run **Plugins → Tokens Studio for Figma**.
2. **Import** → upload `inception/design/tokens.json`.
3. Two token sets appear: `light` and `dark`. Both are first-class — `dark` holds
   only the tokens the dark theme overrides, which is deliberate and matches the
   stylesheet.
4. **Set Tokens Studio's base font size to 16** (Settings → Base font size).
   `fontSize` and `spacing` are expressed in `rem`; without a base of 16 they
   import as literal strings and `0.75rem` never becomes `12px`. `radius` is
   already in `px` and is unaffected.
5. **Create Variables / Styles** to push the set into Figma variables.

74 of the 80 tokens import clean and typed. The six that do not are listed below.

## 2. Known gaps in the export

These are defects in the generator, not decisions of the design system. Each has
an owner. None of them block drawing frames.

| # | What is wrong | Effect on import | Workaround now | Owner |
| --- | --- | --- | --- | --- |
| H-1 | `color.link` exports as `"var(--c-primary)"` instead of the DTCG alias `{color.primary}`. The generator copies CSS values verbatim and does not translate `var()` references into aliases. | `color/link` imports as a broken literal, not a link to `color/primary`. | After import, edit `color/link` in Tokens Studio to alias `color/primary`. One click; the value is teal 700 either way. | DEV |
| H-2 | The three `shadow` tokens carry no `$type`. DTCG `shadow` requires a structured object (`color`/`offsetX`/`offsetY`/`blur`/`spread`); `shadow-md` and `shadow-lg` are two-layer shadows and need a DTCG array. The generator emits the raw CSS string. | Shadows do not import as Figma effect styles. | Create three effect styles by hand from the values in `tokens.css` (lines 138–140 light, 176–178 dark). Six values total. | DEV |
| H-3 | The generator splits the light set from the dark set on the first literal occurrence of the at-media keyword **anywhere in the file, comments included** — so mentioning it in a comment above the real block silently truncates the light palette to nothing. | Not currently triggered; `tokens.css` carries a comment warning authors off the word. A future edit could trip it and ship an empty light set. | None needed. Do not write that keyword in prose in `tokens.css`. | DEV |

Fixed as part of this handoff: `--focus-ring` was renamed `--c-focus-ring`. It is
a colour, but without the `c-` prefix the generator filed it under `other` with no
type, so it imported as a typeless value rather than a colour swatch. Pure rename
— same value in both themes, no visual change to any component.

## 3. Frames to draw

One frame per numbered state: **156 frames across 10 screens.**

Name each frame exactly `SCR-###/ST-##` — for example `SCR-001/ST-03`. That is
not cosmetic. It is the key `/ux` matches on when it reads the file back through
the MCP, and it is the same marker the component previews already use
(`<!-- @state SCR-###/ST-## -->`). A frame named "Login error v2" is invisible to
the check.

Put each screen's frames on their own Figma page, named `SCR-###`.

The per-state behaviour, layout, and accessibility notes live in the spec files
under `inception/design/screens/`. This table is the checklist, not the brief.

### SCR-001 — Login  (6 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-001/ST-01` | Default |
| `SCR-001/ST-02` | Field validation error |
| `SCR-001/ST-03` | Submitting |
| `SCR-001/ST-04` | Invalid credentials |
| `SCR-001/ST-05` | Submission failed (network/server error) |
| `SCR-001/ST-06` | Password changed |

### SCR-002 — Request a password reset  (6 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-002/ST-01` | Default |
| `SCR-002/ST-02` | Email validation error |
| `SCR-002/ST-03` | Sending |
| `SCR-002/ST-04` | Request received |
| `SCR-002/ST-05` | Couldn't send the request |
| `SCR-002/ST-06` | Link sent again |

### SCR-003 — Set a new password  (8 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-003/ST-01` | Checking the link |
| `SCR-003/ST-02` | Ready |
| `SCR-003/ST-03` | Link no longer valid |
| `SCR-003/ST-04` | Link already used |
| `SCR-003/ST-05` | Password doesn't meet the rules |
| `SCR-003/ST-06` | Passwords don't match |
| `SCR-003/ST-07` | Saving |
| `SCR-003/ST-08` | Couldn't save |

### SCR-004 — Dashboard  (10 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-004/ST-01` | Default |
| `SCR-004/ST-02` | Loading |
| `SCR-004/ST-03` | Nothing yet (first visit) |
| `SCR-004/ST-04` | A section with nothing in it |
| `SCR-004/ST-05` | Could not load |
| `SCR-004/ST-06` | Responding to a request |
| `SCR-004/ST-07` | Response recorded |
| `SCR-004/ST-08` | Response failed |
| `SCR-004/ST-09` | Session starting soon |
| `SCR-004/ST-10` | Signing out |

### SCR-005 — My profile and skills  (21 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-005/ST-01` | Default |
| `SCR-005/ST-02` | Loading |
| `SCR-005/ST-03` | Nothing set up yet (first visit) |
| `SCR-005/ST-04` | A skills list with nothing in it |
| `SCR-005/ST-05` | Could not load |
| `SCR-005/ST-06` | Unsaved changes |
| `SCR-005/ST-07` | Saving |
| `SCR-005/ST-08` | Saved |
| `SCR-005/ST-09` | Something needs fixing before saving |
| `SCR-005/ST-10` | Save failed |
| `SCR-005/ST-11` | Changing the photo |
| `SCR-005/ST-12` | That photo can't be used |
| `SCR-005/ST-13` | Adding a skill |
| `SCR-005/ST-14` | That skill isn't in the catalog |
| `SCR-005/ST-15` | You've already listed that |
| `SCR-005/ST-16` | Saving the skill |
| `SCR-005/ST-17` | Skill added |
| `SCR-005/ST-18` | Couldn't save the skill |
| `SCR-005/ST-19` | Editing a listed skill |
| `SCR-005/ST-20` | Removing a skill |
| `SCR-005/ST-21` | Couldn't remove the skill |

### SCR-006 — Find a trainer  (19 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-006/ST-01` | Default — everyone who teaches |
| `SCR-006/ST-02` | First load |
| `SCR-006/ST-03` | Refining |
| `SCR-006/ST-04` | Nobody teaches anything yet |
| `SCR-006/ST-05` | Nothing matches |
| `SCR-006/ST-06` | Couldn't load |
| `SCR-006/ST-07` | A filter open |
| `SCR-006/ST-08` | Filters in force |
| `SCR-006/ST-09` | A colleague with no ratings yet |
| `SCR-006/ST-10` | Opening a trainer |
| `SCR-006/ST-11` | Trainer panel — loaded |
| `SCR-006/ST-12` | Trainer panel — no times published |
| `SCR-006/ST-13` | Trainer panel — a slot already taken |
| `SCR-006/ST-14` | Requesting a slot |
| `SCR-006/ST-15` | Request sent |
| `SCR-006/ST-16` | Request failed |
| `SCR-006/ST-17` | Already requested |
| `SCR-006/ST-18` | Trainer panel — couldn't load |
| `SCR-006/ST-19` | Panel at phone width |

### SCR-007 — My teaching  (29 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-007/ST-01` | Default |
| `SCR-007/ST-02` | Loading |
| `SCR-007/ST-03` | Couldn't load |
| `SCR-007/ST-04` | You don't teach anything yet |
| `SCR-007/ST-05` | No requests waiting |
| `SCR-007/ST-06` | No times published yet |
| `SCR-007/ST-07` | Add a time — the form |
| `SCR-007/ST-08` | Add a time — the form won't accept it |
| `SCR-007/ST-09` | Publishing |
| `SCR-007/ST-10` | Time published |
| `SCR-007/ST-11` | Couldn't publish |
| `SCR-007/ST-12` | Add a time — at phone width |
| `SCR-007/ST-13` | A time with requests waiting on it |
| `SCR-007/ST-14` | A booked time |
| `SCR-007/ST-15` | A time that has passed |
| `SCR-007/ST-16` | Responding to a request |
| `SCR-007/ST-17` | Response recorded |
| `SCR-007/ST-18` | Response failed |
| `SCR-007/ST-19` | Two requests for the same time |
| `SCR-007/ST-20` | A request that expired |
| `SCR-007/ST-21` | Removing a published time |
| `SCR-007/ST-22` | A session waiting for you to confirm |
| `SCR-007/ST-23` | Marking complete — the confirm step |
| `SCR-007/ST-24` | Marking complete — in flight |
| `SCR-007/ST-25` | Marked complete |
| `SCR-007/ST-26` | Couldn't mark it complete |
| `SCR-007/ST-27` | A taught session with no rating yet |
| `SCR-007/ST-28` | A taught session the learner rated |
| `SCR-007/ST-29` | Nothing taught yet |

### SCR-008 — My learning  (28 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-008/ST-01` | Default |
| `SCR-008/ST-02` | Loading |
| `SCR-008/ST-03` | Couldn't load |
| `SCR-008/ST-04` | You haven't started learning yet |
| `SCR-008/ST-05` | Nothing coming up |
| `SCR-008/ST-06` | No open requests |
| `SCR-008/ST-07` | Nothing finished yet |
| `SCR-008/ST-08` | A confirmed session |
| `SCR-008/ST-09` | A confirmed session with no meeting link yet |
| `SCR-008/ST-10` | Cancelling — the confirm step |
| `SCR-008/ST-11` | Cancelling — in flight |
| `SCR-008/ST-12` | You cancelled |
| `SCR-008/ST-13` | Couldn't cancel |
| `SCR-008/ST-14` | The trainer cancelled |
| `SCR-008/ST-15` | A session whose time has passed |
| `SCR-008/ST-16` | Cancelling at phone width |
| `SCR-008/ST-17` | A request still pending |
| `SCR-008/ST-18` | A request that was approved |
| `SCR-008/ST-19` | A request that was rejected |
| `SCR-008/ST-20` | A request that ran out of time |
| `SCR-008/ST-21` | A finished session you haven't rated |
| `SCR-008/ST-22` | Rating — the form |
| `SCR-008/ST-23` | Rating — submitting |
| `SCR-008/ST-24` | Rating saved |
| `SCR-008/ST-25` | Couldn't save the rating |
| `SCR-008/ST-26` | A finished session you've rated |
| `SCR-008/ST-27` | The certificate |
| `SCR-008/ST-28` | The certificate isn't available |

### SCR-009 — Notifications  (21 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-009/ST-01` | Panel — default |
| `SCR-009/ST-02` | Panel — loading |
| `SCR-009/ST-03` | Panel — nothing yet |
| `SCR-009/ST-04` | Panel — could not load |
| `SCR-009/ST-05` | Panel — everything read |
| `SCR-009/ST-06` | Panel — narrow width |
| `SCR-009/ST-07` | Bell — unread count at the cap |
| `SCR-009/ST-08` | Page — default |
| `SCR-009/ST-09` | Page — loading |
| `SCR-009/ST-10` | Page — nothing yet |
| `SCR-009/ST-11` | Page — could not load |
| `SCR-009/ST-12` | Page — filtered, with results |
| `SCR-009/ST-13` | Page — filtered, nothing matches |
| `SCR-009/ST-14` | Page — loading older |
| `SCR-009/ST-15` | Page — end of the list |
| `SCR-009/ST-16` | Opening a notification |
| `SCR-009/ST-17` | Marking all read — in flight |
| `SCR-009/ST-18` | Marked all read |
| `SCR-009/ST-19` | Mark all read — failed |
| `SCR-009/ST-20` | Something arrives while the surface is open |
| `SCR-009/ST-21` | What it points at is gone |

### SCR-010 — Certificate of completion  (8 frames)

| Frame name to use in Figma | State |
| --- | --- |
| `SCR-010/ST-01` | Opening |
| `SCR-010/ST-02` | The certificate |
| `SCR-010/ST-03` | Printed |
| `SCR-010/ST-04` | A long name or a long skill |
| `SCR-010/ST-05` | Not ready yet |
| `SCR-010/ST-06` | Not available to you |
| `SCR-010/ST-07` | Couldn't load it |
| `SCR-010/ST-08` | Signed out |

## 4. The return trip

Once frames exist — even a handful — run `/ux` and say the file is ready. It will,
through the MCP:

- read the page structure and match frame names against the 156 expected states
- report what is drawn, what is missing, and what is named so it cannot be matched
- read the file's variable definitions and compare them against `tokens.json`,
  so a drifted or hand-typed colour shows up as a diff rather than a surprise
- flag any frame whose fills are raw hex values absent from the token set

That check is the reason the naming convention is strict. It is also the only
part of this handoff that stays true on its own; everything above is a one-time
setup a person performs.

## 5. What this does not do

It does not replace the design tool. The repo holds the specification — which
screens exist, which states each has, what the palette is. Figma holds the pixels.
The token file is what keeps the two agreeing on values without either owning the
other.

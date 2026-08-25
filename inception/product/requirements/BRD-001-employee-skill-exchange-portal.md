# BRD-001 — Employee Skill Exchange Portal

> Approval = the PO/BA human reviewing + merging this document's PR (Gate 1). No approval headers — GitHub records who approved what.

|                  |                                                                                  |
| ---------------- | -------------------------------------------------------------------------------- |
| **Author**       | BA persona (AI draft) with joy_j@trigent.com                                     |
| **Source input** | `inception/product/inputs/2026-08-25-employee-skill-exchange-portal.md` (verbatim raw material — required) |
| **Related**      | EPIC-### (filled as downstream artifacts appear)                                 |

## 1. Business goal

Give employees a single internal portal where they can offer the skills they're able to teach and find colleagues who can teach them something new, so the organization relies less on external training, spreads existing expertise across departments, and gives employees a visible way to keep learning. This is being built first as a **proof of concept for a team of under 10 employees**, to validate the workflow before any wider rollout.

## 2. Actors

| Actor    | Description                                                                                       | Needs                                                                                 |
| -------- | --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Employee | Any portal user. Acts as a **Learner** (finds and books training) and/or a **Trainer** (lists skills to teach, publishes availability, runs sessions) — one account, both roles, no separate sign-up. | See who can teach what, book time with them, track their own learning and teaching history. |
| IT       | Maintains the portal's reference data directly (no in-app admin screen for this POC).               | A simple way to add/retire entries in the skill catalog and department list.          |
| System   | Automated behavior: credential checks, auto-rejecting stale booking requests, generating certificates, sending notifications. | Consistent, timely execution of the rules in this document.                          |

## 3. Workflows

1. **Login & logout** — Employee logs in with corporate email + password; can reset a forgotten password via an emailed link; lands on their Dashboard on success; can log out, ending access until they log in again.
2. **Profile management** — Employee views/edits their profile: description, department (fixed list), designation, profile picture.
3. **Skill management** — Employee adds skills they can teach and/or want to learn, chosen from a predefined skill catalog, each with a proficiency level; can edit or remove entries later.
4. **Skill search** — Employee searches for trainers, filtering by skill name, department, skill level, and trainer rating. Any employee who has listed a skill as teachable appears immediately — no approval step.
5. **Availability & booking** — Trainer publishes bookable time slots; a learner requests one; the trainer approves or rejects it; an unanswered request auto-rejects as the slot's start time nears.
6. **Session delivery** — On approval, the system generates a video-meeting link and notifies both parties. Either party can cancel a confirmed session before it starts.
7. **Completion & feedback** — Trainer marks the session complete; learner is optionally prompted to rate (1–5 stars) and comment; the system auto-generates a certificate for the learner.
8. **History** — Employee reviews their Learning History (as a learner) and Teaching History (as a trainer).
9. **Notifications** — System raises in-app notifications for booking confirmation, approval/rejection, cancellation, session reminders, and feedback prompts.

## 4. Functional requirements

> Each REQ is testable (pass/fail decidable), prioritized MoSCoW, and sourced (input file or named person).

| ID      | Requirement                                                                                                                   | Priority | Source                              |
| ------- | ------------------------------------------------------------------------------------------------------------------------------ | -------- | ------------------------------------ |
| REQ-001 | The system must let an employee log in with a corporate email address and a password stored in the portal (standalone login, no SSO). | Must     | Input §1; grill Q1                  |
| REQ-002 | The system must validate submitted credentials and reject invalid ones with a generic error that does not reveal whether the email or password was wrong. | Must     | Input §1                             |
| REQ-003 | The system must let an employee request a password reset via a link emailed to their corporate address; the link expires after a set time and works only once. | Must     | Input §1                             |
| REQ-004 | The system must redirect an employee to their dashboard immediately after a successful login.                                 | Must     | Input §1                             |
| REQ-005 | The system must let an employee log out, ending their session so the portal requires login again.                             | Must     | Input §13                            |
| REQ-006 | The system must show, on the dashboard, the employee's upcoming approved sessions, pending booking requests, skills they teach, skills they are learning, and recent notifications. | Must     | Input §2                             |
| REQ-007 | The system must let an employee view and edit their profile: description, department, designation, and profile picture.       | Must     | Input §3                             |
| REQ-008 | The system must let an employee pick their department from a fixed list maintained directly by IT (no in-app editor).         | Must     | Grill Q8                             |
| REQ-009 | The system must let an employee add a skill (to teach and/or to learn) chosen from a predefined skill catalog maintained directly by IT (no in-app editor). | Must     | Input §4; grill Q6, Q6a              |
| REQ-010 | The system must let an employee set a proficiency level — Beginner, Intermediate, or Expert — for each skill they list.        | Must     | Input §4; grill Q7                   |
| REQ-011 | The system must let an employee edit or remove a previously listed skill.                                                      | Must     | Input §4                             |
| REQ-012 | The system must let an employee search for trainers, filterable by skill name, department, skill level, and trainer rating.    | Must     | Input §5                             |
| REQ-013 | The system must include an employee in trainer search results as soon as they list a skill as teachable, with no approval step. | Must     | Grill Q2                             |
| REQ-014 | The system must let a trainer publish available time slots for a skill they teach.                                            | Must     | Grill Q3                             |
| REQ-015 | The system must let a learner submit a booking request against one of a trainer's published, unbooked slots.                  | Must     | Input §6                             |
| REQ-016 | The system must let a learner view the status of each booking request they submitted (Pending / Approved / Rejected).         | Must     | Input §6                             |
| REQ-017 | The system must let a trainer approve or reject a pending booking request.                                                     | Must     | Input §7                             |
| REQ-018 | The system must automatically reject a booking request still pending as the slot's start time nears, and notify the learner.  | Must     | Grill Q4 (exact cutoff: OQ-1)        |
| REQ-019 | On approval, the system must generate a video-meeting link for the session and share it with both learner and trainer.        | Must     | Grill Q3 (platform: OQ-2)            |
| REQ-020 | The system must let either the learner or the trainer cancel a confirmed session before it starts, and notify the other party. | Must     | Grill Q5                             |
| REQ-021 | The system must restrict each booked session to exactly one learner and one trainer (no multi-learner sessions).               | Must     | Grill Q12                            |
| REQ-022 | The system must let a trainer mark a session as completed after it has occurred.                                               | Must     | Input §9                             |
| REQ-023 | On completion, the system must prompt the learner to rate the trainer (1–5 stars) and optionally add a comment; submitting is optional. | Must     | Input §10; grill Q9, Q13             |
| REQ-024 | On completion, the system must automatically generate a certificate (learner name, skill, trainer, date) for the learner, with no manual step by the trainer. | Must     | Grill Q10                            |
| REQ-025 | The system must display a trainer's average star rating on their profile and in search results.                               | Must     | Grill Q9                             |
| REQ-026 | The system must let an employee view their Learning History: skill name, trainer, date completed, certificate, and feedback they submitted. | Must     | Input §11                            |
| REQ-027 | The system must let an employee view their Teaching History: learner name, skill taught, session date, and rating received.    | Must     | Input §12                            |
| REQ-028 | The system must raise an in-app notification for: booking confirmation, booking approval/rejection, session cancellation, session reminders, and feedback prompts. | Must     | Input §8; grill Q5, Q11              |

## 5. Non-functional requirements

| ID      | Category        | Requirement (quantified or `TBD (owner)`)                                                                 | Priority |
| ------- | --------------- | ------------------------------------------------------------------------------------------------------------ | -------- |
| NFR-001 | Security        | Passwords must be stored using an industry-standard one-way hash (e.g., bcrypt); never stored in plain text.  | Must     |
| NFR-002 | Availability    | No formal uptime SLA for this POC (under 10 users); best-effort availability during business hours is sufficient. | Should   |
| NFR-003 | Performance     | Pages should load within 3 seconds under normal use at POC scale (fewer than 10 concurrent users).            | Should   |
| NFR-004 | Usability       | The portal must be usable on both desktop-width and mobile-width browsers (responsive layout).                | Must     |
| NFR-005 | Integration     | Video-meeting link generation depends on a specific platform (Teams/Zoom/Meet) — `TBD (owner: IT)`, see OQ-2. | Must     |

## 6. Business rules

### BR-001.1 Self-declared trainers

- **Statement:** When an employee lists a skill as teachable, the system must make them appear in trainer search results for that skill immediately, without any approval step.
- **Rationale:** The organization chose an open, self-declared model over a gatekept one — trust is built through ratings and feedback after the fact, not a vetting step up front (grill Q2).
- **Examples:** Pass — an employee adds "Excel – Expert" as teachable and appears in search results right away. Fail — the system withholds them from search pending a manager's sign-off.
- **Affects:** REQ-013

### BR-002.1 Booking auto-rejection

- **Statement:** When a booking request has not been approved or rejected and the slot's start time is approaching, the system must automatically reject it and notify the learner. The exact cutoff (e.g., 24 hours vs. 2 hours before start) is not yet fixed.
- **Rationale:** Prevents a learner being left in indefinite limbo when a trainer doesn't respond (grill Q4).
- **Examples:** Pass — a request pending past the cutoff is auto-rejected and the learner is notified to pick another slot. Fail — the request sits pending past the session's start time with no resolution.
- **Affects:** REQ-018

### BR-003.1 One learner per session

- **Statement:** When a learner books a trainer's slot, the system must not allow any other learner to book that same slot.
- **Rationale:** Sessions are one-on-one for this scope; group/multi-learner sessions are explicitly out of scope (grill Q12).
- **Examples:** Pass — a second learner attempting to book an already-booked slot is blocked. Fail — two learners are both confirmed into the same slot.
- **Affects:** REQ-021

### BR-004.1 Feedback is optional

- **Statement:** When a session is marked complete, the system must prompt the learner for feedback but must not block any other action (e.g., new bookings, viewing history) if the learner skips it.
- **Rationale:** Feedback is encouraged but not enforced (grill Q13).
- **Affects:** REQ-023

### BR-005.1 Certificates are automatic

- **Statement:** When a trainer marks a session complete, the system must generate the learner's certificate automatically — no manual upload or trigger from the trainer.
- **Rationale:** Removes an extra step from the trainer and guarantees every completed session has a consistent record (grill Q10).
- **Affects:** REQ-024

## 7. Validations

- Login: email and password are both required; invalid credentials show one generic error (REQ-002).
- Password reset: the emailed link expires after a set time and is single-use (REQ-003).
- Booking: a learner cannot request a slot that is already booked or in the past (REQ-015, BR-003.1).
- Booking: a trainer cannot approve a request after it has already auto-rejected (REQ-017, REQ-018).
- Feedback: a submitted rating must be a whole number from 1 to 5 (REQ-023).
- Cancellation: a confirmed session cannot be cancelled once its scheduled start time has passed (REQ-020).

## 8. Constraints

- Proof-of-concept scope: fewer than 10 employees; no corporate identity-provider (SSO) integration — login is standalone (grill Q1).
- No in-app admin screens for the skill catalog or department list; both are maintained directly by IT (grill Q6a, Q8).
- Video-meeting integration is blocked on IT confirming which platform to use (OQ-2).

## 9. Risks

| ID       | Risk                                                                                                   | Likelihood | Impact | Mitigation                                                                              |
| -------- | -------------------------------------------------------------------------------------------------------- | ---------- | ------ | ------------------------------------------------------------------------------------------ |
| RISK-001 | A skill catalog that only IT can edit may lag behind skills employees actually want to list, discouraging use. | Medium     | Medium | Keep a lightweight "request a new skill" path to IT; revisit an in-app admin screen if edits become frequent. |
| RISK-002 | Standalone login means employees manage a portal-specific password separate from their corporate one, which may reduce adoption. | Medium     | Low    | Acceptable at POC scale; revisit SSO if the portal moves beyond POC.                       |
| RISK-003 | The booking auto-reject cutoff (OQ-1) is not yet fixed; too aggressive a value could reject requests before a trainer reasonably could respond. | Low        | Medium | Confirm the exact cutoff with the business owner before delivery.                          |

## 10. Out of scope

- Manager/HR approval workflow for becoming a trainer (open/self-declared model chosen instead).
- Group or multi-learner sessions.
- In-app admin screens for managing the skill catalog or department list.
- Single sign-on / corporate identity-provider integration.
- Formal uptime SLA or high-availability architecture (POC scale only).
- Mandatory (blocking) feedback.

## 11. Open questions

| #    | Question                                                                                          | Owner         | Status                                     |
| ---- | ---------------------------------------------------------------------------------------------------- | ------------- | ------------------------------------------- |
| OQ-1 | Exact cutoff for auto-rejecting an unanswered booking request before the session's start time.       | Business owner| Open (default recommendation: 24 hours)     |
| OQ-2 | Which video-conferencing platform to integrate for auto-generated meeting links (Teams/Zoom/Meet).   | IT            | Open                                        |

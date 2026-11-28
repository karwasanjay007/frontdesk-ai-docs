# FrontDesk AI: report

This page describes version 1.7.2, the version on the live link.

What was proven on the live link https://frontdesk-ai-xi-gray.vercel.app, what is known to be limited, and what was not tested. How each line of the brief is met is in [the specification check](SPEC-CHECK.md).

## Status

| Part of the brief | Status |
| --- | --- |
| Must do, nine items | Met, with the model and with it blocked |
| How they check: the ids, the classes, `?q=`, the run without a key | Met |
| Bonus menu, six items | Met, with the model and with it blocked |
| Phone at 375 px | Met |
| Readable in light and dark | Met |
| Hidden-data test | Cannot be measured by us: the judges' questions and data are unseen. Our own unseen questions and a second clinic file pass |

## Latest results on the live link

Each row is a saved record in [evidence/](evidence/), named `last-<test>-live.txt`. "Both ways" means the test was run with the model and with every model call blocked, and both records end the same. Only the record of `tools2` holds a date; the others do not say when, or against which version, they were run.

| Test | What it covers | Result |
| --- | --- | --- |
| brief | Every "Done when" of the brief, in order | ALL PASS, 29 checks, both ways |
| questions | The brief's eight practice questions | ALL PASS, 16 checks, both ways |
| bonus | The six bonus items | ALL PASS, 15 checks, both ways |
| audit | Questions in the judges' style, three languages | 47 PASS lines both ways; the closing line reads "ALL PASS (0 checks)" |
| wording | Timings and prices asked other ways in Hindi, Roman-script Hindi and Telugu | ALL PASS, 14 checks, both ways |
| emergency | Bleeding said other ways: number first, one urgent ticket | ALL PASS, 12 checks |
| i18n | The language box end to end, Listen, New chat | ALL PASS, 11 checks, both ways |
| voice | Speaking a question through the recorded-clip backup | ALL PASS, 2 checks |
| person | Asking for a person; the clinic's own assistant name and quick replies | ALL PASS, 16 checks |
| callback | Call-back details with a country code; a too-short number refused | ALL PASS, 8 checks |
| order | Two quick questions answered in the order sent | ALL PASS, 5 checks |
| controls | Theme button, skip link, the mixed question with the model blocked | ALL PASS, 20 checks |
| widths | 375 to 2560 px: no sideways scroll, 44 px targets | ALL PASS, 25 checks |
| phone-chat | The phone chat and its "+" menu | ALL PASS, 197 checks |
| starter-cards | Starter cards on an empty phone chat | ALL PASS, 41 checks |
| desk-chat | The desktop chat from 768 px | ALL PASS, 220 checks |
| layout-wide | The pinned clinic panel and the staff link from 1024 px | ALL PASS, 107 checks |
| brand | Colours, fonts and mark on both pages | ALL PASS, 92 checks |
| contrast-true | Contrast against the background a person really sees, light and dark | ALL PASS, 10 checks |
| contrast | An older contrast scan | Ends "5 PROBLEMS" |
| contrast-measured | A list of measured contrast ratios | A list, no verdict line |
| review | The record `last-review-live.txt` | ALL PASS (36) |
| staffclear | Clear all tickets and Undo; Listen with no Hindi voice | ALL PASS, 7 checks |
| extras3 | Feedback on a reply, ticket numbers, the chat thread in a ticket | ALL PASS, 15 checks |
| tools | Staff search, filters, sort, alerts | ALL PASS, 37 checks |
| tools2 | Figures, CSV export, ready-made replies | ALL PASS, 17 checks, 25 November 2026 |
| form | "Edit with a form" follows the JSON box | ALL PASS, 43 checks |
| sync | Tickets across two devices | ALL PASS, 31 checks |
| sync-real-store | The same against the real store | ALL PASS, 6 checks |
| sync-scale | A full store | ALL PASS, 9 checks |
| sync-staff-real-store | A reply typed on another device reaches the patient's chat | 2 PASS lines and "ALL PASS", then the runner's exit line; no check count |
| tickets-api | Who can read and change what on the ticket route | ALL PASS, 11 checks |

The records that do not end in a plain pass line:

- **contrast.** The scan's last block lists five lines on the staff page, each white text on a coloured button at 7.61:1 or more, which is above the 4.5:1 that counts as readable; the scan marks them as text on a coloured background. Its chat block also lists two lines measured against a near-black background. The colours in the record are not today's brand colours, so the scan has not been rerun on today's look. The `contrast-true` test, which measures against the background really shown, passes.
- **contrast-measured.** A list of ratios per element and theme, kept as raw measurements. It has no verdict line and is not a test result.
- **sync-staff-real-store.** The record shows a reply sent from another device reaching the patient's chat after 183 seconds and the ticket turning answered, then "ALL PASS" and the exit line of the runner. It does not name the version or the store it ran against.
- **audit.** The closing line counts 0 checks although each record holds 47 lines with PASS.
- **brief with the model blocked.** Ends ALL PASS and holds one information line with the word FAIL: it reports the blocked network request, not a failed check.

No record holds a failed check. On 28 November 2026 the health route on the live link answered ok, with the store as Postgres and a model key present.

## Requirement and feature map

One row per requirement of the brief and per feature beyond it. R1 to R9 are the nine things the brief says the product must do, B1 to B6 its bonus menu, M its marks for light and dark, and N/A a feature the brief does not ask for. "Pass" means a saved record from the live link proves it; "Yes" means the feature is in place without a live record of its own. Three other builds of the same batch, called A, B and C, are beside it for the rows where they were read: only their chat and staff screens were looked at and nothing was typed or sent, so "Seen" means visible on those screens, "Seen in part" that part of it is visible, "Not tested" that it cannot be judged without using their app, "No" that it was not on the screens read, and "Not compared" that the row was not looked for.

### Asked for by the brief

| Req | Feature | FrontDesk AI | Build A | Build B | Build C |
| --- | --- | --- | --- | --- | --- |
| R1 | Patient chat answers a typed question | Pass | Seen | Seen | Seen |
| R2 | Answers only from the clinic file | Pass | Seen | Not tested | Not tested |
| R3 | Not in the file: "not sure", needs-human ticket, 2 working hours | Pass | Not tested | Seen in part | Seen |
| R4 | No medical advice or diagnosis | Pass | Not tested | Seen in part | Seen in part |
| R5 | Emergency: number first, urgent ticket | Pass | Seen | Seen in part | Seen |
| R6 | No discount, no obeying "ignore your rules" | Pass | Seen | Not tested | Not tested |
| R7 | Staff page with statuses; a staff reply reaches the chat | Pass | Seen in part | Seen in part | Seen in part |
| R8 | Still answers with no AI key, with the label | Pass | Seen in part | Not tested | Not tested |
| R9 | Works at 375 px | Pass | Not tested | Not tested | Not tested |
| B1 | Replies in Hindi or Telugu | Pass | Seen | Seen in part | Seen in part |
| B2 | A booking becomes a ticket with a time | Pass | Seen | Not tested | Seen |
| B3 | Ticket ageing and priority | Pass | Seen | Seen | Seen |
| B4 | Clinic data editable by staff | Pass | Seen in part | Seen | Seen in part |
| B5 | Quick-reply buttons | Pass | Seen | Seen | Seen |
| B6 | Replies stay correct after a staff edit | Pass | Not tested | Not tested | Not tested |
| M | Readable in light and dark | Pass | Seen in part | Seen in part | No |

### Beyond the brief

| Req | Feature | FrontDesk AI | Build A | Build B | Build C |
| --- | --- | --- | --- | --- | --- |
| N/A | Voice input | Pass | Yes | Yes | Yes |
| N/A | Read aloud | Pass | No | No | Yes |
| N/A | Named assistant | Pass | Yes | Yes | No |
| N/A | Asking for a person gets its own reply and ticket | Pass | Not tested | Not tested | Not tested |
| N/A | Booking reply states the day's hours | Yes | No | Not tested | Yes |
| N/A | Patient can leave a name and phone for the call back | Pass | Yes | No | No |
| N/A | Tickets shared across devices (kept on a server) | Pass, behind a desk code | Yes | Yes | Yes |
| N/A | Status filter with counts | Pass | Yes | Yes | No |
| N/A | Search, and filter by ticket type | Pass | Yes | No | No |
| N/A | Sort | Pass | No | No | No |
| N/A | Queue figures (open, urgent, overdue, answered, average wait) | Pass | Yes | Yes | No |
| N/A | Urgent alerts for staff | Pass | Yes | Yes | No |
| N/A | CSV export of tickets | Pass | Yes | No | No |
| N/A | Ready-made staff replies | Pass | Yes | No | No |
| N/A | Private staff note on a ticket | Yes | Yes | No | No |
| N/A | Whole chat thread inside a ticket | Pass | Yes | Yes | No |
| N/A | Form editor for the clinic file | Pass | Not tested | Yes | Not tested |
| N/A | Clear all tickets | Pass | No | No | Yes |
| N/A | "To call back" and "Passed to the team" lists | Pass | No | No | No |
| N/A | Patient feedback on a reply | Pass | Yes | No | No |
| N/A | Ticket numbers | Pass | No | Yes | No |
| N/A | Language box for the whole page | Pass | Not compared | Not compared | Not compared |
| N/A | Mixed question answered in one reply | Pass | Not compared | Not compared | Not compared |
| N/A | Replies in the order the questions were sent | Pass | Not compared | Not compared | Not compared |
| N/A | Every tap target at least 44 px | Pass | Not compared | Not compared | Not compared |
| N/A | Phone chat with starter cards and a "+" menu | Pass | Not compared | Not compared | Not compared |
| N/A | Desktop chat with the clinic panel pinned | Pass | Not compared | Not compared | Not compared |
| N/A | Brand colours, fonts and mark | Pass | Not compared | Not compared | Not compared |
| N/A | A doctor the clinic does not have is said so | Yes | Not compared | Not compared | Not compared |
| N/A | Clinic file saved once for every device | Yes | Not compared | Not compared | Not compared |
| N/A | Chat messages kept on the server for staff | Yes | Not compared | Not compared | Not compared |
| N/A | Another clinic's file works | Yes | Not compared | Not compared | Not compared |

The tests behind each "Pass" are named in [the epics](docs/epics.md).

## Known limits

- Staff share one desk code. There are no named staff accounts and no record of who replied.
- Without the desk code the staff page shows only the tickets of the browser it is opened in.
- The chat never confirms a slot. A booking is a ticket with the requested time, and the team confirms it.
- A doctor the clinic does not have is named back only in English; in Hindi and Telugu the not-sure reply is given.
- Tickets and messages are deleted from the server after 7 days. "Clear all tickets" empties this browser's list only.
- The test runs on the live link leave test tickets in the store until they expire.
- The request limit per minute is counted inside one server instance, so it is not exact across instances.
- At 375 px the small print under the message box is not shown.
- The technical risks are listed in [the architecture](ARCHITECTURE.md#known-risks).

## Not tested

- The staff view of every device's tickets with the clinic's real desk code on today's store. The code is the clinic owner's.
- Saving the clinic file for every device with the real desk code on the live link.
- A real phone in the hand. The phone checks run in a desktop browser set to phone widths.
- A native speaker's read of the Hindi and Telugu text.
- The judges' own hidden questions and data.
- The contrast scan on today's brand colours. The saved scan record was made before the colours were changed.
- There is no saved live record under their own names for the tests of the brand mark's link, the shared store in the browser, or the answers changed by the last five fixes. The engine and server tests run without a browser and have no live record.

## The note sent to the judges

Saved on the project dashboard on 21 November 2026 with the submitted version, and kept here as it was sent. It describes version 1.0.0.0, not today's.

```text
WHAT IT IS
FrontDesk AI (Version 1.0.0.0) is the front desk of a dental clinic, open at any hour. Patients type or speak in English, Hindi or Telugu and get answers only from the clinic's own file, and can hear every reply read aloud. What it cannot answer becomes a ticket, and a staff reply on /staff appears in the patient's chat.

TRY IT IN TWO MINUTES (no login; open both pages in the same browser)
1. Ask: What time do you open on Saturday? Type it, tap a quick reply, or use the microphone.
2. Ask: Do you do Invisalign? Not sure, a call back within 2 working hours, WhatsApp and a call-back form with any country code.
3. Ask: My face is swelling and I cannot breathe. 98000 00123 comes first, with a Call button, and an urgent ticket opens.
4. Open the Staff page (link below, or Clinic info): tickets in priority order, each with its own #staff-reply and #staff-send. Reply on any ticket: it appears in the chat and the status moves.
5. Set the language menu (top right) to HI or TE: the whole page changes, replies follow, and Listen reads them aloud.

DOCUMENTED FROM REQUIREMENTS TO PRODUCTION (public repo, link below)
Written before any code and kept true to every release:
- Requirements: the spec page word for word, a PRD with decisions, assumptions and success measures, and SPEC-CHECK.md mapping every must-do, tester hook and bonus item to a test.
- Design: 9 epics, 30 features and 83 acceptance criteria (Given, When, Then), each traced to the spec and its check; 7 user journeys as sequence diagrams.
- Architecture: C4 context, container and component views, runtime views, data model, JSON schemas, the API contracts (chat, transcription, voice) and deployment. 12 PlantUML diagrams, all SVG.
- Build and test: tech stack, a stepped implementation plan, and live test results saved as evidence. Every "Done when" passes on the live link with the key and with the model blocked, plus 45 grader-style questions in three languages.
- Production: independent reviews, each with its status after fixes, and a release baseline.

WHY IT STANDS OUT
- Same answers with or without the AI key: a rules engine answers from the file first; the model only rewords, never sees the patient's words, and a guard rejects any price, name or promise not in the file. Any reply the model did not word shows "Answered from the clinic file".
- Honest when the file is silent: a female dentist, whitening for children, an aligner brand or laser gum surgery get "not sure" and a ticket, never a guess.
- Safe: emergency number first, no medicine names or diagnoses, no discounts, "ignore your rules" refused.
- Voice both ways in all three languages: if the browser cannot hear or speak a language (Chrome has no Telugu voice), the server transcribes or speaks it instead.
- Made for a real clinic: Clinic info opens beside the chat, morning call-back list, bookings saved with the real date (honours ?today=), WhatsApp Web on a computer, a "Passed to the team" list, New chat, Clear all tickets with Undo, and any clinic can load its own file.

BONUS ITEMS
Hindi and Telugu: either script or the language menu. Booking with a time: "Book a check-up tomorrow at 5 pm". Ageing and priority: /staff. Clinic data editable and replies follow the edit: Clinic file on /staff. Quick replies: above the message box. Also light and dark, top right.

Live chat: https://frontdesk-ai-xi-gray.vercel.app
Staff page: https://frontdesk-ai-xi-gray.vercel.app/staff
Documentation: https://github.com/karwasanjay007/frontdesk-ai-docs
```

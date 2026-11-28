# FrontDesk AI: report

This page describes version 1.7.3, the version on the live link.

What was proven on the live link https://frontdesk-ai-xi-gray.vercel.app, what is known to be limited, and what was not tested. How each requirement of the specification is met is in [the specification check](SPEC-CHECK.md).

## Status

| Part of the specification | Status |
| --- | --- |
| Required, nine items | Met, with the model and with it blocked |
| The ids kept stable for automated tests, `?q=`, the run with the model blocked | Met |
| Optional features, six items | Met, with the model and with it blocked |
| Phone at 375 px | Met |
| Readable in light and dark | Met |
| Another clinic's file | Met on a second clinic file and on questions the rules were not written for |

## Latest results on the live link

Each row is a saved record in [evidence/](evidence/), named `last-<test>-live.txt`. "Both ways" means the test was run with the model and with every model call blocked, and both records end the same. Only the record of `tools2` holds a date; the others do not say when, or against which version, they were run.

| Test | What it covers | Result |
| --- | --- | --- |
| brief | Every "Done when" of the specification, in order | ALL PASS, 29 checks, both ways |
| questions | The specification's eight practice questions | ALL PASS, 16 checks, both ways |
| `bonus.mjs` | The six optional features | ALL PASS, 15 checks, both ways |
| audit | The same questions asked many ways, in three languages | 47 PASS lines both ways; the closing line reads "ALL PASS (0 checks)" |
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
| starter-cards | The quick-reply row on an empty phone chat | ALL PASS, 37 checks |
| desk-chat | The desktop chat from 768 px | ALL PASS, 220 checks |
| layout-wide | The pinned clinic panel and the staff link from 1024 px | ALL PASS, 107 checks |
| brand | Colours, fonts and mark on both pages | ALL PASS, 92 checks |
| contrast-true | Contrast against the background a person really sees, light and dark | ALL PASS, 10 checks |
| contrast | An older contrast scan | Ends "5 PROBLEMS" |
| contrast-measured | A list of measured contrast ratios | A list, no verdict line |
| review | The record `last-review-live.txt` (the look of both pages to a first-time visitor) | ALL PASS (36) |
| mark-link | The brand mark in the top bar opens sai4b.com, on both pages | ALL PASS, 14 checks |
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

One row per requirement of the specification and per feature beyond it. R1 to R9 are the required items, B1 to B6 the optional ones, M the quality line for light and dark, and N/A a feature the specification does not ask for. "Pass" means a saved record from the live link proves it; "Yes" means the feature is in place without a live record of its own. The last column names the record in the table above.

### In the specification

| Req | Feature | Result | Test record |
| --- | --- | --- | --- |
| R1 | Patient chat answers a typed question | Pass | brief |
| R2 | Answers only from the clinic file | Pass | brief |
| R3 | Not in the file: "not sure", needs-human ticket, 2 working hours | Pass | brief |
| R4 | No medical advice or diagnosis | Pass | brief |
| R5 | Emergency: number first, urgent ticket | Pass | brief |
| R6 | No discount, no obeying "ignore your rules" | Pass | brief |
| R7 | Staff page with statuses; a staff reply reaches the chat | Pass | brief |
| R8 | Still answers with no AI key, with the label | Pass | brief |
| R9 | Works at 375 px | Pass | brief |
| B1 | Replies in Hindi or Telugu | Pass | `bonus.mjs` |
| B2 | A booking becomes a ticket with a time | Pass | `bonus.mjs` |
| B3 | Ticket ageing and priority | Pass | `bonus.mjs` |
| B4 | Clinic data editable by staff | Pass | `bonus.mjs` |
| B5 | Quick-reply buttons | Pass | `bonus.mjs` |
| B6 | Replies stay correct after a staff edit | Pass | `bonus.mjs` |
| M | Readable in light and dark | Pass | contrast-true |

### Beyond the specification

| Req | Feature | Result | Test record |
| --- | --- | --- | --- |
| N/A | Voice input | Pass | voice |
| N/A | Read aloud | Pass | i18n |
| N/A | Named assistant | Pass | person |
| N/A | Asking for a person gets its own reply and ticket | Pass | person |
| N/A | Booking reply states the day's hours | Yes | None of its own |
| N/A | Patient can leave a name and phone for the call back | Pass | callback |
| N/A | Tickets shared across devices (kept on a server) | Pass, behind a desk code | sync |
| N/A | Status filter with counts | Pass | tools |
| N/A | Search, and filter by ticket type | Pass | tools |
| N/A | Sort | Pass | tools |
| N/A | Queue figures (open, urgent, overdue, answered, average wait) | Pass | tools2 |
| N/A | Urgent alerts for staff | Pass | tools |
| N/A | CSV export of tickets | Pass | tools2 |
| N/A | Ready-made staff replies | Pass | tools2 |
| N/A | Private staff note on a ticket | Yes | None of its own |
| N/A | Whole chat thread inside a ticket | Pass | extras3 |
| N/A | Form editor for the clinic file | Pass | form |
| N/A | Clear all tickets | Pass | staffclear |
| N/A | "To call back" and "Passed to the team" lists | Pass | `bonus.mjs` |
| N/A | Patient feedback on a reply | Pass | extras3 |
| N/A | Ticket numbers | Pass | extras3 |
| N/A | Language box for the whole page | Pass | i18n |
| N/A | Mixed question answered in one reply | Pass | controls |
| N/A | Replies in the order the questions were sent | Pass | order |
| N/A | Every tap target at least 44 px | Pass | widths |
| N/A | Phone chat with the quick-reply row and a "+" menu | Pass | phone-chat |
| N/A | Desktop chat with the clinic panel pinned | Pass | desk-chat |
| N/A | Brand colours, fonts and mark | Pass | brand |
| N/A | A doctor the clinic does not have is said so | Yes | None of its own |
| N/A | Clinic file saved once for every device | Yes | None of its own |
| N/A | Chat messages kept on the server for staff | Yes | None of its own |
| N/A | Another clinic's file works | Yes | None of its own |

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
- Clinic files other than the two tried, and wordings beyond those in the tests.
- The contrast scan on today's brand colours. The saved scan record was made before the colours were changed.
- There is no saved live record under their own names for the tests of the shared store in the browser, or the answers changed by the last five fixes. The engine and server tests run without a browser and have no live record.

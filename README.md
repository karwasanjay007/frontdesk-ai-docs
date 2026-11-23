# FrontDesk AI

FrontDesk AI is the front desk chat for a dental clinic: patients get answers from the clinic's own file at any hour, and anything it cannot answer goes to a human as a ticket.

| | |
| --- | --- |
| Live | Patient chat https://frontdesk-ai-xi-gray.vercel.app and staff page https://frontdesk-ai-xi-gray.vercel.app/staff |
| Documentation | This repository |
| Code | `frontdesk-ai` (private while a competition is open) |
| Built by | Sanjay Karwa, Smart AI for Business |
| Stack | Plain HTML, CSS and JavaScript with a rules engine in the browser; one Vercel function calling OpenRouter with DeepSeek V4.1 Flash (`deepseek/deepseek-v4.1-flash`), the key on the server only; localStorage |
| Version | 1.3.0.1 (23 Nov 2026; baseline 1.0.0.0, Sanjay Karwa, 21 Nov 2026) |
| Status | Built and live. Every functional test passes on the live link, with the key and with the model blocked (rerun 22 Nov 2026 after two wording fixes, see REPORT.md). The contrast scan finds nothing under 4.5:1; its flagged lines are notes about text on the teal brand colour, not failures. Version 1.1.0.0 (22 Nov 2026) adds four things beside the baseline: a reply and a ticket when a patient asks for a person, search and filters on the staff page, a form for the clinic file, and an assistant name and quick replies set in that file. Every earlier test passes on the live link after it, unedited |

## For the judges: FrontDesk AI in two minutes

**Promise.** Patients get clinic-true answers at any hour; what the file cannot cover becomes a human
ticket, not a made-up reply.

**Why not a typical LLM demo.** Rules answer from the clinic file first. Emergency, medical, refusal and
"not sure" use fixed templates and never wait on the model. The model may only reword a real file answer;
a guard rejects invented prices or promises. With the key blocked, the same facts still show.

**Same browser for the staff step.** The staff page opens with the tickets of this browser. Open the
patient chat and `/staff` as two tabs in the **same browser**. A different browser, device or private
window opens with an empty queue. Since version 1.3.0.0 staff can also turn on "Show tickets from all
devices" there, which needs the clinic's desk code; without the code another device still shows nothing.

**Try it in two minutes (five steps).**

1. Open https://frontdesk-ai-xi-gray.vercel.app and ask "Are you open on Saturday?" (type it, tap a
   quick reply, or use the microphone).
2. Ask "Do you offer Invisalign?": not sure, call-back within 2 working hours, optional name and phone.
   One needs-human ticket opens.
3. Ask "My face is swelling and I cannot breathe.": "Call 98000 00123 now." comes first, with a Call button,
   and an urgent ticket opens.
4. Open https://frontdesk-ai-xi-gray.vercel.app/staff in a **second tab of the same browser** (also linked from Clinic info, which opens beside the chat). Tickets appear
   in priority order (Urgent, then Needs human, then Answered) with their age. Every ticket has its own reply box
   (`#staff-reply`) and Send button (`#staff-send`). Reply on the Invisalign ticket and send: the reply appears in the patient's chat as "From the clinic team" and the status moves to
   answered. Optional: edit Saturday hours in the clinic-file editor, save, and ask again in chat; Reset
   restores the original file.
5. Set the language menu (top right) to HI or TE: the whole page changes, ask the Saturday question again, and
   Listen reads the reply aloud; where the browser has no voice for the language (Chrome on Windows has no Telugu voice), a server voice plays it instead.

**Against the specification.** Our own tests on the live link; line by line in [SPEC-CHECK.md](SPEC-CHECK.md),
results in [evidence/](evidence/), build report in [REPORT.md](REPORT.md).

| Part of the scoring | Marks | Status | Evidence |
| --- | --- | --- | --- |
| Must do, 9 items | (in the hidden-data test) | All met, with the key and with the model blocked | [last-brief-live.txt](evidence/last-brief-live.txt) ALL PASS (29 checks); [last-brief-nokey-live.txt](evidence/last-brief-nokey-live.txt) ALL PASS (31 checks); [last-audit-ai-live.txt](evidence/last-audit-ai-live.txt) and [last-audit-blocked-live.txt](evidence/last-audit-blocked-live.txt): 45 grader-style questions in three languages ALL PASS (rerun 22 Nov 2026) |
| How they check: ids, classes, `?q=`, no-key run | (in the hidden-data test) | All met | the same two files; [last-questions-live.txt](evidence/last-questions-live.txt) ALL PASS (16 checks) |
| Hidden-data test | 60 | Met on our tests; the graders' questions and data are unseen | the files above and [last-engine.txt](evidence/last-engine.txt) ALL PASS (127 checks); [last-emergency-live.txt](evidence/last-emergency-live.txt) ALL PASS (12 checks): bleeding that will not stop, said five other ways in English, Hindi and Telugu (22 Nov 2026); [last-wording-live.txt](evidence/last-wording-live.txt) and [last-wording-blocked-live.txt](evidence/last-wording-blocked-live.txt) ALL PASS (14 checks each): timings and prices asked other ways in Hindi, Roman-script Hindi and Telugu (22 Nov 2026); [last-sweep.txt](evidence/last-sweep.txt) ALL PASS (386 questions): unseen questions written by two reviewers who read only the clinic file, in English, Hindi, Telugu and both typed in Roman letters (22 Nov 2026); [last-other-clinic.txt](evidence/last-other-clinic.txt) ALL PASS: a second clinic with other names, numbers, hours, services and policies, and the same file with a section left out (22 Nov 2026) |
| Bonus menu, at most 6 count | 30 | All 6 met, both ways | [last-bonus-live.txt](evidence/last-bonus-live.txt) ALL PASS (15 checks) |
| Phone and light/dark | 10 | Met | [last-widths-live.txt](evidence/last-widths-live.txt) ALL PASS (25 checks, 375 to 2560 px); [last-contrast-live.txt](evidence/last-contrast-live.txt): nothing under 4.5:1 (its flagged lines are text on the teal brand colour at 5.89:1 or more) |

**What we added, and why.**

- A morning call-back list on `/staff`, so the "someone will call you back" promise has a list to work from.
- An optional name and phone for the call-back beside the reply, shown on the staff card with call and
  WhatsApp links, so staff have a number to call.
- A WhatsApp hand-off beside replies that need a person, and an after-hours note saying when the clinic
  opens next.
- A clinic-file editor, so the clinic (or another booking-based business) keeps its own hours, prices and
  details current without a developer.
- A "Could not answer" list on `/staff`, a Directions link built from the file's address, and a privacy line.
- The one difference: voice in and out in English, Hindi and Telugu, using the browser's own speech.

**How it answers.** A rules engine answers every question from the clinic file first: emergency, rule
change or discount, medical, booking, file answer, not sure, in that order. Emergency, refusal, medical and
not-sure replies are fixed templates and never go to the model (and they carry the file label because the model did not word them).
For a real file answer, the model receives only the drafted answer and the clinic file, never the patient's
words, and may only reword it; a guard rejects any number, name or promise that is not in the draft or the
file. With no key, an error, no reply within 6 s or a rejection, that file answer shows with "Answered from
the clinic file", so the no-key run gives the same facts.

**How it was built.** The grading page was captured word for word before kickoff, and one literal test per
"Done when" was written first and failed until the build existed. The build followed the PRD, tech stack
and implementation plan in this repository. Before release four independent reviews (functional, brief
audit, leading products, visual) were run and their findings fixed, then every test was rerun on the live
link. Every diagram is PlantUML source plus SVG.

**Deliberately left out.**

| Left out | Why | Next step |
| --- | --- | --- |
| Staff sign-in with accounts | Since version 1.3.0.0 tickets are mirrored to a server store and a staff device can see every device's tickets with the clinic's desk code. One shared code, no named accounts | Named staff accounts and an audit of who replied |
| Real slot booking | The bot never invents or confirms a slot; a booking request becomes a needs-human ticket with the requested time, and the team confirms | Booking against the clinic's real diary |
| Sign-in for staff | A sign-in screen would stand between a fresh-browser tester and `/staff` | Staff sign-in before a real clinic uses it |

## What it does

- Answers patients' questions about timings, doctors, prices, insurance, policies and location, only from
  the clinic's own file, in English, Hindi or Telugu, typed or spoken.
- Puts the emergency number first when a message sounds like an emergency, and opens an urgent ticket.
- Never gives medical advice, never names a medicine, never offers a discount and ignores "ignore your
  rules"; for symptoms it says a dentist must see the patient and offers to book.
- When the file does not hold the answer, says so, opens a needs human ticket and promises a call back
  within 2 working hours.
- Turns a booking request into a ticket with the requested time; the team confirms.
- Gives staff a ticket list in priority order with ages, a call-back list, a reply box whose reply reaches
  the patient's chat, and an editor for the clinic file.
- Still answers when no AI key works; every reply the model did not word carries the label "Answered from the clinic file"
  (the fixed emergency, refusal, medical, booking and not-sure replies included).
- Works on a 375 px phone and on wide screens, in light and dark.

## What FrontDesk AI supports

Every requirement of the brief and every feature beyond it, in one table. The first column names the
requirement the row belongs to: R1 to R9 are the nine "must do" items, B1 to B6 the bonus menu, M the phone and
light/dark marks. N/A means the brief does not ask for it. "Pass" means tested on the live link, with the AI and
with it blocked; "Yes" means the feature is in place. The last column is the version it arrived in.

| Req | Feature | Supported | Since |
| --- | --- | --- | --- |
| R1 | Patient chat: type a question, get a reply | Pass | 1.0 |
| R2 | Answers only from the clinic file: timings, doctors, prices, policies, insurance, location | Pass | 1.0 |
| R3 | Not in the file: says "not sure", opens a needs-human ticket, promises a call within 2 working hours | Pass | 1.0 |
| R4 | No medical advice or diagnosis; a dentist must see the patient, with an offer to book | Pass | 1.0 |
| R5 | Emergency: the emergency number first, and an urgent ticket | Pass | 1.0 |
| R6 | No discount, and no obeying "ignore your rules" | Pass | 1.0 |
| R7 | Staff page at `/staff` with every ticket and its status; a staff reply shows in the patient's chat | Pass | 1.0 |
| R8 | Still answers with no AI key, with the label "Answered from the clinic file" | Pass | 1.0 |
| R9 | Works on a phone at 375 px | Pass | 1.0 |
| B1 | Replies in Hindi or Telugu when the patient writes in them | Pass | 1.0 |
| B2 | A booking request becomes a ticket with a time | Pass | 1.0 |
| B3 | Ticket ageing and priority | Pass | 1.0 |
| B4 | Clinic data editable by staff | Pass | 1.0 |
| B5 | Quick-reply buttons | Pass | 1.0 |
| B6 | Replies stay correct after a staff edit | Pass | 1.0 |
| M | Readable in light and dark | Pass | 1.0 |
| N/A | Voice input, with a recording fallback when the browser has no speech service | Yes | 1.0 |
| N/A | Read aloud ("Listen" beside each reply) | Yes | 1.0 |
| N/A | Patient can leave a name and phone for the call back | Yes | 1.0 |
| N/A | "To call back" and "Could not answer" lists for staff | Yes | 1.0 |
| N/A | Clear all tickets, with undo | Yes | 1.0 |
| N/A | Language menu: every label, the replies and Listen follow the chosen language; a New chat button | Yes | 1.0 |
| N/A | Clinic info panel: today's hours, Call, WhatsApp, Directions, the week's hours, the doctors and the address, with an open or closed sign | Yes | 1.0 |
| N/A | Call-back phone with a country code list, checked for that country's length | Yes | 1.0 |
| N/A | Asking for a person gets its own reply and a ticket; a seventh quick reply | Yes | 1.1 |
| N/A | Named assistant and the clinic's own quick replies, set in the clinic file | Yes | 1.1 |
| N/A | Ticket search, status filter and sort | Yes | 1.1 |
| N/A | Urgent alerts for staff (turned on by staff) | Yes | 1.1 |
| N/A | Form editor for the clinic file, beside the JSON box | Yes | 1.1 |
| N/A | Queue figures: open, urgent, over 2 hours, answered, average wait, feedback | Yes | 1.2 |
| N/A | CSV export of tickets | Yes | 1.2 |
| N/A | Ready-made staff replies | Yes | 1.2 |
| N/A | Filter by ticket type | Yes | 1.2 |
| N/A | Private staff note on a ticket | Yes | 1.2 |
| N/A | Booking reply states the named day's hours | Yes | 1.2 |
| N/A | Ticket numbers | Yes | 1.3 |
| N/A | The patient's whole chat thread inside a ticket | Yes | 1.3 |
| N/A | Patient feedback on a reply ("Helpful" or "Not helpful") | Yes | 1.3 |
| N/A | Tickets shared across devices: a staff device can see and answer tickets from any phone | Yes, behind the clinic's desk code | 1.3 |

All 16 rows of the brief pass, and all 23 rows beyond it are in place. The same table with three other builds of
the batch beside it is in [REPORT.md](REPORT.md), section "Requirement and feature map"; what each version added
and how it was tested is in the sections "Version 1.1.0.0" to "Version 1.3.0.0" there.

## Documentation

| Document | What it covers |
| --- | --- |
| [REPORT.md](REPORT.md) | Build report, claims ledger, review fixes and the note for judges |
| [SPEC-CHECK.md](SPEC-CHECK.md) | The specification, rated line by line with evidence |
| [PRD.md](PRD.md) | What is built and why: decisions, rules, scenarios |
| [TECH-STACK.md](TECH-STACK.md) | Each technology choice with its reason and cost |
| [IMPLEMENTATION-PLAN.md](IMPLEMENTATION-PLAN.md) | The build slices and release checks |
| [ARCHITECTURE.md](ARCHITECTURE.md) | C4 context, container, component and deployment views; decisions; risks |
| [docs/epics.md](docs/epics.md) | Epics, features and acceptance criteria, traced to the PRD and the tests |
| [docs/journeys.md](docs/journeys.md) | One sequence diagram per user journey |
| [docs/data-model.md](docs/data-model.md) | Entities, the formula for every derived figure, storage |
| [docs/diagrams/](docs/diagrams/) | PlantUML source and SVG for every diagram |
| [docs/schema/](docs/schema/) | JSON Schemas for the clinic file and the tickets; the chat function's request and reply |
| [review/](review/) | The pre-release reviews (leading products, brief audit, functional, visual), each with its status after fixes |
| [evidence/](evidence/) | The latest test results against the live link |

Every diagram is PlantUML source plus SVG. No PNG or JPG anywhere in this repository.

## Privacy

- Messages and tickets are kept in the browser's localStorage on the device, and the staff page opens with
  the tickets of its own browser. Since version 1.3.0.0 a patient's tickets (the question, the reply, and a name
  and phone if the patient left them) are also mirrored to the clinic's server store, so staff on another
  device can answer. They are deleted there after seven days.
- On the server, a ticket can be read or changed only with the private key of the browser that created it, or
  by staff with the clinic's desk code. A ticket id or a patient id alone opens nothing. Until the clinic sets a
  desk code, the list of every device's tickets is closed to everyone.
- When the model is used to word an answer, the server function sends it the drafted answer and the clinic file
  only, never the patient's words.
- Voice is the one exception, and only as a backup. Speaking a question uses the browser's own speech service.
  If that service refuses, the page records a short clip (about 30 seconds at most) and sends it to the server,
  which passes it to a model to be turned into text; the text then goes through the same rules as typed text.
  Listen sends only the assistant's own reply text to the server voice, and only when the browser has no voice
  for the language.
- A name and phone are asked for only if the patient chooses to leave them for a call-back; they are saved
  on that ticket.
- The fine print says who sees the messages (the clinic's front desk, once a chat is stored on the server; this
  browser and the clinic team before that), and asks patients not to type medical records.
- The model key is kept in the Vercel environment and never reaches the page.

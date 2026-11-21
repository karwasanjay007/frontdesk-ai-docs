# FrontDesk AI

FrontDesk AI is the front desk chat for a dental clinic: patients get answers from the clinic's own file at any hour, and anything it cannot answer goes to a human as a ticket.

| | |
| --- | --- |
| Live | Patient chat https://frontdesk-ai-xi-gray.vercel.app and staff page https://frontdesk-ai-xi-gray.vercel.app/staff |
| Documentation | This repository |
| Code | `frontdesk-ai` (private while a competition is open) |
| Built by | Sanjay Karwa, Smart AI for Business |
| Stack | Plain HTML, CSS and JavaScript with a rules engine in the browser; one Vercel function calling OpenRouter with DeepSeek V4.1 Flash (`deepseek/deepseek-v4.1-flash`), the key on the server only; localStorage |
| Version | 1.0.0.0 (baseline, Sanjay Karwa, 21 Nov 2026) |
| Status | Built and live; every test passes on the live link, with the key and with the model blocked (21 Nov 2026) |

## For the judges: FrontDesk AI in two minutes

**What it is.** A patient with a question for a dental clinic today has to call during opening hours and
wait for the front desk. FrontDesk AI answers timings, doctors, prices, insurance and location at any hour,
only from the clinic's own file, and passes anything else to the team as a ticket that staff answer on
`/staff`; the reply appears in the patient's chat.

**Same browser (required for the staff step).** Tickets live in this browser's localStorage only. Open the
patient chat and `/staff` as two tabs in the **same browser**. A different browser, device or private
window shows an empty queue and cannot complete the staff reply check.

**Try it in two minutes (five steps).**

1. Open https://frontdesk-ai-xi-gray.vercel.app and ask "What time do you open on Saturday?" (type it, tap a
   quick reply, or use the microphone). The reply gives Saturday hours from the clinic file. Press Listen to
   hear it read aloud.
2. Ask "Do you do Invisalign?": it says it is not sure, will not guess, promises a call back within 2 working
   hours, and offers WhatsApp plus an optional name and phone for the call-back. One needs-human ticket opens.
3. Ask "My face is swelling and I cannot breathe.": "Call 98000 00123 now." comes first, with a Call button,
   and an urgent ticket opens.
4. Open https://frontdesk-ai-xi-gray.vercel.app/staff in a **second tab of the same browser**. Tickets appear
   in priority order (Urgent, then Needs human, then Answered) with their age. Reply on the Invisalign ticket
   and send: the reply appears in the patient's chat as "From the clinic team" and the status moves to
   answered. Optional: edit Saturday hours in the clinic-file editor, save, and ask again in chat; Reset
   restores the original file.
5. Switch the language menu to हिंदी or తెలుగు: page chrome changes, ask the Saturday question again, and
   Listen reads the reply aloud.

**Against the specification.** Our own tests on the live link; line by line in [SPEC-CHECK.md](SPEC-CHECK.md),
results in [evidence/](evidence/), build report in [REPORT.md](REPORT.md).

| Part of the scoring | Marks | Status | Evidence |
| --- | --- | --- | --- |
| Must do, 9 items | (in the hidden-data test) | All met, with the key and with the model blocked | [last-brief-live.txt](evidence/last-brief-live.txt) ALL PASS (28 checks); [last-brief-live-nokey.txt](evidence/last-brief-live-nokey.txt) ALL PASS (30 checks) |
| How they check: ids, classes, `?q=`, no-key run | (in the hidden-data test) | All met | the same two files; [last-questions-live.txt](evidence/last-questions-live.txt) ALL PASS (16 checks) |
| Hidden-data test | 60 | Met on our tests; the graders' questions and data are unseen | the files above and [last-engine.txt](evidence/last-engine.txt) ALL PASS (48 checks) |
| Bonus menu, at most 6 count | 30 | All 6 met, both ways | [last-bonus-live.txt](evidence/last-bonus-live.txt) ALL PASS (15 checks) |
| Phone and light/dark | 10 | Met | [last-widths-live.txt](evidence/last-widths-live.txt) ALL PASS (20 checks, 375 to 2560 px); [last-contrast-live.txt](evidence/last-contrast-live.txt): nothing under 4.5:1 (its flagged lines are text on the teal brand colour at 5.89:1 or more) |

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
not-sure replies are fixed templates and never go to the model (and they are not labelled as file answers).
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
| Server ticket store | Tickets stay in this browser's localStorage only: no database or account. Chat and `/staff` must share one browser | A shared store so staff on another device see every ticket (phase 2) |
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
- Still answers when no AI key works; real file answers then carry the label "Answered from the clinic file"
  (fixed emergency, refusal, medical and not-sure replies stay unlabelled).
- Works on a 375 px phone and on wide screens, in light and dark.

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

- Messages and tickets are kept in the browser's localStorage on the device; the staff page reads them from
  the same browser. Nothing is stored on a server.
- When the model is used, the server function sends it the drafted answer and the clinic file only, never
  the patient's words.
- A name and phone are asked for only if the patient chooses to leave them for a call-back; they are saved
  on that ticket.
- The fine print says messages stay in this browser and reach only the clinic team, and asks patients not
  to type medical records.
- The model key is kept in the Vercel environment and never reaches the page.

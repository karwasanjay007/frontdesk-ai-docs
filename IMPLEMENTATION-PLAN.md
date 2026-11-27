# FrontDesk AI: implementation plan

DRAFT, written after `TECH-STACK.md` is approved. The build agent follows these steps in order
(AGENTS.md section 3b). Each step is one small slice with the check that proves it done. If a step turns
out wrong, stop, say so, and update this plan before carrying on. Code and tests are in `../frontdesk-ai`;
every document, this one included, stays in this repository.

| | |
| --- | --- |
| Last revised | 21 Nov 2026 |

How to read the last column. "SC 1.n" is row n of SPEC-CHECK.md section 1 (Must do); "SC 2" names a row
of section 2 (How they check); "SC 3.n" is bonus item n; "SC 4" is the phone and light/dark part of the
scoring. S-1 to S-9 are the nine must-do scenarios in PRD.md 5.4 (spec items 1 to 9, in order). S-10 to
S-17 are the eight sample questions, in the order of `sample-questions.json`: S-10 Saturday time, S-11
root canal, S-12 children's doctor, S-13 Niva Bupa, S-14 Invisalign, S-15 painkiller, S-16 face
swelling, S-17 ignore your rules. G-n are the decisions in PRD.md 2.5.

## Steps

| # | Slice | Done when (the check, run and pasted into WORKLOG.md) | Requirement |
| --- | --- | --- | --- |
| 0 | Literal specification test | `tests/brief.mjs` has one check per "Done when" in `brief/spec-page.md`, runs in a fresh browser in the order given, and fails because nothing is built yet. `tests/questions.mjs` asks the eight sample questions and checks `must_include`, `must_not_include` and the ticket status, also failing | SC 1 to 3; S-1 to S-17 |
| 1 | Shell and first visit | `/` loads with the brand header, an empty conversation, one greeting with class `bot-greeting` (no `.bot-msg` on the page), `#chat-input` and `#chat-send`; `/staff` loads from its own folder with an empty `#ticket-queue`; no dialog, login or splash. Script: `document.querySelectorAll('.bot-msg').length === 0` on first load | SC 2 (Chat, Bot replies), SC 1.7; G1, G13, G17 |
| 2 | Clinic file loader and rules engine, no model | `clinic-data.json` loads into one clinic store; the engine answers S-10 to S-13 from the data with the exact words ("9:00 am to 8:00 pm", "Rs 6,500 to Rs 9,000", "Dr. Sneha Rao" and "Saturday", lower-case "cashless"). A unit run in Node over the engine prints all four PASS; a second run with a changed price in a copy of the file prints the changed price | SC 1.1, 1.2; S-1, S-2, S-10 to S-13; G6, G18 |
| 3 | Safety rules in the fixed order | S-14 gives the not-sure reply with "2 working hours"; S-15 names no medicine and says "a dentist must see you" and offers to book; S-16 starts with "Call 98000 00123 now."; S-17 is a polite no with no "%" anywhere. No reply contains the patient's own words. `tests/questions.mjs` S-14 to S-17 PASS with no model | SC 1.3 to 1.6; S-3 to S-6, S-14 to S-17; G4, G5, G8, G9 |
| 4 | Chat page sends and shows replies | Typing S-10 in `#chat-input` and pressing `#chat-send` (and Enter) adds one complete `.bot-msg`; the typing indicator has no `.bot-msg` class and is removed before the reply is added; an empty message is ignored, any other message is accepted | SC 1.1, SC 2 (Chat, Bot replies); S-1; G2, G16 |
| 5 | Tickets | Every patient message writes one ticket (question, reply, time, status) to localStorage. Statuses are exactly `answered`, `needs human`, `urgent`, `closed`, set by the reply kind (G11a): emergency `urgent`; medical advice or symptoms `needs human`; not sure `needs human`; booking `needs human` with the time; discount or rule-change refusal `answered`; file answer `answered`. Script over S-10 to S-17 checks one new ticket each with exactly these statuses (S-14 and S-15 `needs human`, S-16 `urgent`, S-17 and S-10 to S-13 `answered`) | SC 1.3, 1.5, SC 2 (Ticket list); S-3 to S-6, S-10 to S-17; G10, G11a |
| 6 | `/staff` with the single reply box | `#ticket-queue` holds one `[data-ticket]` per ticket with `data-status` set; only the open ticket holds `#staff-reply` and `#staff-send`; on load the open ticket is the first in priority order (urgent, then needs human, newest first); tapping another ticket moves the box; `document.querySelectorAll('#staff-reply').length === 1` | SC 1.7, SC 2 (Ticket list, Staff reply); S-7; G11 |
| 7 | Staff reply reaches the chat and always changes the status | With the chat open in one tab and `/staff` in another, a staff reply typed and sent appears in that patient's chat within a second as a `.bot-msg` marked "From the clinic team", carried by the `storage` event. The status always changes: `needs human` or `urgent` becomes `answered`; `answered` becomes `closed`. Script replies once to an urgent, a needs-human and an answered ticket and checks each `data-status` before and after. The same check in one tab after a reload of the chat page | SC 1.7; S-7; G11, G12 |
| 8 | `?q=` | Opening `/?q=What%20time%20do%20you%20open%20on%20Saturday%3F` sends the question once, exactly as if typed: one patient message, one ticket, the answer as the last `.bot-msg`. A redraw of the chat (after a staff reply or a storage update) does not send it again | SC 2 (Ask on load); S-1; G14 |
| 9 | No-key label | With no function reachable (local static server, or the key removed), every reply from the file carries the small label "Answered from the clinic file". `tests/questions.mjs --no-key` prints the same eight PASS lines as step 3 | SC 1.8, SC 2 (No key run); S-8; G15 |
| 10 | The model function with guard and timeout | `api/chat` calls OpenRouter with DeepSeek V4.1 Flash (`deepseek/deepseek-v4.1-flash`) with the key from the Vercel environment, sending only the drafted answer and the clinic file (never the patient's words), allows the model 5 s and returns `{ text }` only; the browser gives up after 6 s; the guard (G7) drops a reply with a price, time, doctor or phone not in the file, a medicine, a "%" or a missing fact. Unit run: ten crafted bad replies are all rejected; on the preview link the eight questions PASS with the key; the function's response body never contains the key. Record the model's limits and cost in TECH-STACK.md | SC 1.2, 1.8; S-2, S-8, S-10 to S-13; G3, G7 |
| 11 | Hindi and Telugu | A question in Devanagari script gets its reply in Hindi, one in Telugu script in Telugu; amounts, times, names and phone numbers stay exactly as in the file. Script: the Hindi and Telugu forms of S-10 and S-16 each contain "9:00 am" or "98000 00123" | SC 3.1; G19 |
| 12 | Booking tickets | "Can I book for Saturday at 11 am?" gives a reply that says the team will confirm (never confirms a slot) and a `needs human` ticket that stores and shows the requested day and time on `/staff` | SC 3.2; G20 |
| 13 | Ageing and priority | `/staff` sorts urgent, then needs human, then answered, then closed; each ticket shows its age ("12 min"); a needs-human ticket older than the set age is highlighted. Script with tickets of fixed times checks the order and the age text | SC 3.3; G21 |
| 14 | Clinic data editor and answers after an edit | Staff change the root canal price on `/staff`; the edit is saved; the chat page (open or reloaded) answers S-11 with the new price and never the old one; a reset brings back the shipped file. Pasting a file with a field removed still gives the not-sure reply, never an error | SC 3.4, 3.6; S-2, S-11; G18 |
| 15 | Quick replies | Buttons for timings, prices, doctors, insurance, location and book a visit each send their question as if typed and produce one `.bot-msg` and one ticket; the buttons themselves are not `.bot-msg` | SC 3.5; G22 |
| 16 | After-hours awareness | With the device clock set outside the file's hours (Playwright `clock`), a separate small note under the reply (never inside the `.bot-msg`) says when the clinic opens next, worked out from the file, and the ticket is marked for the morning list. The same eight questions at 11 pm and at 11 am give identical `.bot-msg` text and identical `data-status` values; inside hours no note shows | G26, G11a |
| 17 | Morning call list | `/staff` shows "To call back" first: needs-human and booking tickets with the question, age and requested time. It does not change `#ticket-queue` or any `data-status` | G27 |
| 18 | Continue on WhatsApp | Each needs-human and booking reply offers a `wa.me` link to the file's `whatsapp` number with the question written in; the link sits outside the `.bot-msg` | G28 |
| 19 | Any booking-based business | A gym file pasted in the editor gives answers from the gym file with the same code; recorded for the judges' note | G29 |
| 20 | Voice in and out | Microphone and speaker buttons in English, Hindi and Telugu using the browser's speech; hidden where the browser lacks it; typing works the same with them hidden. The buttons sit outside the `.bot-msg` | G30 |
| 21 | Phone layout | At 375 px nothing scrolls sideways (`scrollWidth <= clientWidth`), `#chat-input` and `#chat-send` stay in view with the keyboard space and after 30 messages; targets 44 px; content reaches the right edge at 375, 768, 1366, 1920 and 2560 px | SC 1.9, SC 4; S-9; G23 |
| 22 | Themes and contrast | Light and dark both pass the contrast scan at 4.5:1 in every state: rest, hover, focus, the typing indicator, the label, staff actions, toasts and the editor | SC 4; G24 |

**Extras never weaken a graded item.** Steps 16 to 20 add nothing inside a `.bot-msg` that could break a
`must_include` check or add a number that is not in the file. The speaker button, the WhatsApp link and
the after-hours note are not `.bot-msg` elements and sit beside the reply, never inside its text. None
of them adds a ticket, changes a `data-status` or repeats an id; after hours never changes an answer or a
status (G11a). After each of steps 16 to 20,
`tests/brief.mjs` and `tests/questions.mjs` are rerun and must still print ALL PASS.

## Where the agent will likely get stuck

Known traps from earlier builds; keep them and add this project's own.
- A `confirm()` or `alert()` in front of a graded action: an automated tester dismisses it and the action
  silently fails. Use Undo.
- Demo data loaded on the first visit, when the tester starts by adding its own record: a duplicate, then
  wrong totals.
- Validation stricter than the specification: a valid input the tester types is refused.
- A colour that passes at rest but not on hover, after a click or in a toast, in one of the two themes.
- A formatted number the grader cannot parse: keep the sign next to the digits, keep one figure per graded
  element.
- Testing a deploy by changing only the `#` part of the address: the page does not reload and shows the
  old build. Load the page afresh.
- A Bash heredoc on this machine collapses `\\` to `\`: write scripts with backslashes or regular
  expressions with the Write tool, not a heredoc.

This project's own traps:
- **The greeting class.** A welcome message with class `bot-msg` is read as the answer if the tester
  reads before the reply lands. The greeting is `bot-greeting` only (G1).
- **The typing indicator.** A "typing..." bubble with class `bot-msg` is the last `.bot-msg` for a moment.
  It never carries the class, and the reply element is added once, complete; never streamed (G2).
- **Model latency and timeout.** A slow model reply leaves the tester waiting or reading nothing. The file
  answer is computed first; the model gets about 5 s in the function and 6 s in the browser; on timeout
  the file answer shows with the label (G3).
- **Echo.** "I cannot give you 50% off" puts "50%" in the reply; "do not take ibuprofen" names a
  medicine. Fixed templates never repeat the patient's words, and the guard checks model text (G5, G7).
- **Invisalign against aligners.** The file says "braces and aligners", so a keyword match answers
  "Invisalign" with Dr. Imran Qureshi. A named product or brand not in the file is not sure plus a
  needs-human ticket (G8).
- **Case of "cashless".** The file writes "Cashless"; the check expects "cashless". The insurance reply
  is written so the word appears in lower case (G6).
- **Duplicate ids on `/staff`.** A reply box on every ticket repeats `#staff-reply` and `#staff-send`;
  `fill('#staff-reply')` then fails. Only the open ticket holds them (G11).
- **`/staff` as a real path.** A `#/staff` route, or a page that only works when reached from `/`, fails.
  `staff/index.html` is checked on Vercel and on the local static server, with and without the trailing
  slash (G13).
- **`?q=` sent once.** Sending on every render, or again after a staff reply redraws the chat, makes two
  tickets and moves the last `.bot-msg`. The question is sent once per page load (G14).
- **Cross-page updates.** The `storage` event fires only in the other tab, not the one that wrote. The
  chat page listens for it and also reads storage on load and on focus, so a staff reply appears whether
  the chat was open or reopened (G12).
- **The no-key run.** The tester removes the key; the score is the lower of the two runs. Every answer
  must come from the rules engine first, and step 9's no-key test runs after every deploy, not once (G4,
  G15).
- **Extras inside the reply.** A speaker button label, a WhatsApp link text or an after-hours time placed
  inside the `.bot-msg` changes the text the tester reads. Keep them beside it (steps 16 to 20).
- **The device clock.** Grading may run at night. The after-hours note is separate from the `.bot-msg`,
  and the answer and the status are the same by day and by night; every time in the note comes from the
  file (G26, G11a).
- **A staff reply that leaves the status unchanged.** Replying to an `answered` ticket and setting it to
  `answered` again fails "the ticket status changes". A reply always moves the status: `needs human` or
  `urgent` to `answered`, `answered` to `closed` (G11).
- **The wrong ticket open by default.** If the newest ticket is open but the tester replies to an urgent
  one, the box is not where it looks. `/staff` opens the first ticket in priority order: urgent, then
  needs human, newest first (G11).
- **Statuses by reply kind.** An advice reply marked `answered`, or a refusal marked `needs human`, puts
  the wrong `data-status` in the queue. Each fixed reply has its status in one table in the code (G11a),
  tested in step 5.

## Release

| # | Step | Done when |
| --- | --- | --- |
| R1 | Security pass: no key reaches the browser; access checked on the server; row-level security on | Audit output pasted, by a different model from the builder. Here: the key is only in the Vercel environment; `api/chat` returns only `{ text }` or `{ error }`; size limits answer 413; no database, so row-level security does not apply |
| R2 | Deploy to a project address (never sai4b.com) | Live URL fetched, status 200 pasted, for `/`, `/staff` and `/api/chat` (POST) |
| R3 | Literal specification test on the live link | `tests/brief.mjs` ALL PASS with the key and with the key removed; result saved in `tests/` and copied to `evidence/` here |
| R4 | Contrast in every state, both themes | The contrast scan (pattern: sai-resources `kickoff/tools/contrast-scan.mjs`) acts, hovers every control, finds nothing under 4.5:1 |
| R5 | Widths and targets | No sideways scroll at 375 px; content reaches the right edge at 375, 768, 1366, 1920 and 2560 px; 44 px targets |
| R6 | Four reviews in parallel | Category leaders, brief and coaching audit, functional test, visual critique, each in `review/`; every finding fixed or recorded; all tests rerun on the live link |
| R7 | Documentation set complete | README (judges or client section first), SPEC-CHECK filled, ARCHITECTURE (C4), docs/epics.md traced to tests, docs/journeys.md, docs/data-model.md, docs/schema/; every diagram PlantUML plus SVG; no PNG or JPG; links checked |
| R8 | Recapture the specification | The page captured again and compared with `spec-page.md`; changes reflected in SPEC-CHECK.md; R3 rerun |
| R9 | Publish the documents if they are to be shared | This repository made public with one baseline commit by the GitHub no-reply author, after a scan for keys, tokens and personal details; the code repository stays private while the competition is open |
| R10 | Submission draft | The form filled only on request, the draft shown in full, stopped before Submit |

## Revisions

| Date | Change |
| --- | --- |
| 20 Nov 2026 | First draft: steps 0 to 22 from decisions G1 to G30, this project's traps |
| 20 Nov 2026 | G11 and G11a settled: default open ticket by priority, a staff reply always changes the status, the status of each fixed reply; after-hours note outside the `.bot-msg` |
| 21 Nov 2026 | Step 10 brought in line with the build: the model is OpenRouter with DeepSeek V4.1 Flash (Gemini was the draft choice), it receives only the drafted answer and the clinic file, the server allows it 5 s; the guard checks every number, name and promise |

## Versions 1.1.0.0 to 1.3.0.0, as built (22 Nov 2026)

The steps above built the baseline, version 1.0.0.0. Three versions followed on 22 Nov 2026. This section
records them as built.

**Method**

- Each addition lives in its own file beside the baseline, or as a wrapper over a baseline function. The ids,
  statuses and replies the tester reads stay as they were.
- No earlier test was edited. A new feature gets a new test file.
- Each change was tested as a delta first: its own test and the earlier suites on the paths it touched.
- One review round for a change, by a reviewer that did not write it, before the deploy.
- One full regression before the deploy and one on the live link after it.
- One deploy per version, with the earlier deployment noted so it can be rolled back to.

**Version 1.1.0.0**

| What was built | File | Test that covers it | What the review changed |
| --- | --- | --- | --- |
| Ask for a person: its own reply, one needs-human ticket, a seventh quick reply | `core.js` (`answer()` as a wrapper over `answerFile()`) | `tests/person.cjs` (76 checks), `tests/person.mjs` (16) | The first cut scored 5 out of 10: the rule took 29 ordinary questions that the clinic file answers. It was moved out of the rule chain into the wrapper before anything went live |
| Assistant name and the clinic's own quick replies | `core.js`, `chat.js` | `tests/person.cjs`, `tests/form.mjs` (the Assistant section of the form) | Nothing |
| Staff ticket tools: search, status filter, sort, urgent alerts | `staff/tools.js`, `staff/tools.css` | `tests/tools.mjs` (37) | Nothing. The reviewer found no change to the queue in the default state |
| Form for the clinic file | `staff/form.js`, `staff/form.css` | `tests/form.mjs` (43) | Nothing. The reviewer found no markup injection and no change to the JSON box in the default state |

**Version 1.2.0.0**

| What was built | File | Test that covers it | What the review changed |
| --- | --- | --- | --- |
| Queue figures, CSV export, ready-made replies, filter by ticket type, private staff note | the staff add-on, `staff/tools.js` | `evidence/last-tools2-live.txt` (17 checks in the saved file), `evidence/last-tools-live.txt` (37) | No finding is recorded in the report for this version |
| A booking that names one weekday also gives that day's hours | the rules engine | `evidence/last-booking-day.txt` (10) | No finding is recorded in the report for this version |

**Version 1.3.0.0**

| What was built | File | Test that covers it | What the review changed |
| --- | --- | --- | --- |
| Ticket store function | `api/tickets.js` | `tests/tickets-api-live.mjs` (11), `evidence/last-tickets-redis.txt` (9) | The first cut was safe to deploy with the store absent but not safe to turn on: a stranger who knew a ticket id could read the patient's phone number and overwrite the ticket, and the full list was open. It was rewritten before any deploy: a private key per browser with only its hash on the server, a desk code for the full list, no change to a stored question, reply or patient, a rate limit, seven days |
| Sync add-on on `/` and `/staff` | `sync.js` | `tests/sync.mjs` (31), `tests/sync-live.mjs` (6), `evidence/last-sync-staff-real-store-live.txt` | The same rewrite. The test on the real store also found that a reply was twice typed into the private staff note; the note is now folded under a line that says it is not a reply |
| Ticket numbers, the chat thread in a ticket, feedback on a reply | `staff/staff.js`, `chat.js`, `staff/tools.js` | `tests/extras3.mjs` (15) | No finding is recorded in the report |

**Left for later**

- Named staff accounts. Today there is one shared desk code.
- Test tickets from the release checks stay in the store for seven days.
- The alert tone was not heard and the browser notification was not shown in a test, because a test browser
  grants no permission.
- The contrast of the clinic file form was not measured on its own; it uses the page's own colour tokens.
- The assistant name and the clinic's own quick replies apply to every language as written in the file.

## Version 1.3.0.2, as built (25 Nov 2026)

Three corrections after the project feedback, each reproduced first, changed in one place and guarded by a new
test; no earlier test was edited.

- `core.js`: the list of general words beside a price word now holds the words of the other topics the file
  answers, and "how much time" never asks for the price list (`tests/mixed.cjs`, 17 checks).
- `core.js`: `toggleTheme()` calls `themeNote()`, which sets the button's pressed state and name and writes the
  status line `#theme-status`; both pages use it (`tests/controls.mjs`, 20 checks).
- `chat.js`: a click handler on the skip link puts the cursor in `#chat-input`.
- Order of work: engine suites, the new tests and the suites of the graded path on a local server, the deploy,
  then controls, brief (with the AI and with it blocked) and questions on the live link.

## Version 1.3.0.3, as built (25 Nov 2026)

- `sync.js`: `tick()` no longer stops when `document.hidden`; in the background it sends what changed and skips
  the request for news. A send also runs on `visibilitychange` to hidden and on `pagehide`, and a small send is
  marked `keepalive` so it outlives the page.
- Tests: `tests/sync-scale.mjs` (9 checks, three failing before the change), `tests/sync-watch-live.mjs` for the
  staff side on the real store with a person replying.

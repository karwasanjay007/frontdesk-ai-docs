# FrontDesk AI: report

Build report, 21 Nov 2026. Live chat https://frontdesk-ai-xi-gray.vercel.app, staff page
https://frontdesk-ai-xi-gray.vercel.app/staff. Code in the private repository `frontdesk-ai` (four commits:
kickoff, build, tests and warm-up, review fixes). Every result below is from our own tests against the live
link, saved in `evidence/`; none of it is the graders' tester.

## Status per part

| Part | Status | Evidence |
| --- | --- | --- |
| Must do (9 items) | All 9 met, with the key and with the model blocked | `evidence/last-brief-live.txt` ALL PASS (29 checks), `evidence/last-brief-live-nokey.txt` ALL PASS (30 checks) |
| How they check (ids, classes, `?q=`, no-key run) | All met | the same two files; `evidence/last-questions-live.txt` ALL PASS (16 checks) |
| Bonus menu (6 count) | All 6 met, both ways | `evidence/last-bonus-live.txt` ALL PASS (15 checks) |
| Phone and light/dark | No sideways scroll from 375 to 2560 px on both pages; nothing under 4.5:1 in light or dark, at rest or on hover | `evidence/last-widths-live.txt` ALL PASS (20 checks); `evidence/last-contrast-live.txt` (see the note in the evidence table) |
| Go further (after hours, call-back list, WhatsApp hand-off, any business, voice) | Four of five built; any business only partly, because the word lists that recognise intents are dental | `docs/epics.md` E9; functional review finding 17 |
| Rules engine and guard, without a browser | ALL PASS (103 checks) | `evidence/last-engine.txt` |
| Documentation | SPEC-CHECK, PRD, TECH-STACK, ARCHITECTURE, IMPLEMENTATION-PLAN, epics, journeys, data model, schemas, four reviews | this repository |
| Submission | Paste-ready judges' note below; live dashboard recapture still UNVERIFIED (sign-in required; see SPEC-CHECK.md) | SPEC-CHECK.md header; judges' note in this file |
| Version 1.0.0.0 baseline | Label rule; +18 paraphrased fixtures; aligners brand → not sure; patient chat full-bleed UI | `frontdesk-ai` #1 on main; local `node tests/engine.cjs` ALL PASS; `node tests/brief.mjs … --nokey` ALL PASS (31 checks). See `evidence/release-1-0-0-0.md` and `VERSION` |
| Emergency wording, 22 Nov 2026 (after the baseline, version unchanged) | Bleeding that will not stop, said other ways ("I can't stop the bleeding", "bleeding non stop for an hour", for a number of hours, keeps bleeding, and the Hindi and Telugu forms), had reached the ordinary medical, not-sure or file reply: 14 phrasings found. The emergency rule now covers them; ordinary questions about bleeding gums stay ordinary | `frontdesk-ai` 9453a0b, deployed the same day; `evidence/last-engine.txt` ALL PASS (87 checks, 19 new cases); `evidence/last-emergency-live.txt` ALL PASS (12 checks, live); `evidence/last-brief-live.txt` rerun ALL PASS (29 checks) |
| Timing and price wording, 22 Nov 2026 (after the baseline, version unchanged) | "What time" in Hindi (कितने बजे) was read as a price question and got "not sure" with a needs human ticket, in six phrasings; a Roman-script question with "Shanivar" got the insurance answer because the word contains "niva"; a Telugu closing-time question, the Hindi words for consultation, cleaning, EMI and cancelling were not recognised. The word lists now cover them; a price asked for something the file does not price still gets "not sure" | `frontdesk-ai` main, deployed the same day; `evidence/last-engine.txt` ALL PASS (103 checks, 16 new cases); `evidence/last-wording-live.txt` and `evidence/last-wording-blocked-live.txt` ALL PASS (14 checks each, live, with the key and with the model blocked); every other live suite rerun the same day (evidence table) |
| Full sweep of unseen questions, 22 Nov 2026 (after the baseline, version unchanged) | Instead of fixing one family of wordings at a time, two reviewers who read only the clinic file (not the code) wrote 300 unseen questions: 150 in English and 150 in Hindi, Telugu and both typed in Roman letters; 86 traps were added. Of the first 367, 85 differed from the expected reply, among them emergencies typed in Roman letters that reached "not sure" and "bad breath" read as trouble breathing. All were fixed in one pass; things the file does not hold (home visit, video call, warranty, e-mail, ceramic braces, a full-mouth X-ray) now go to "not sure" instead of a nearby answer. Ten questions keep the engine's reply where a reviewer's label was a judgement call; each carries a note in the fixture | `frontdesk-ai` main, deployed the same day; `evidence/last-sweep.txt` ALL PASS (386 questions); `evidence/last-engine.txt` ALL PASS (103 checks); every live suite rerun after the deploy (evidence table) |

## What was built

- **Patient chat** (`/`): `#chat-input` and `#chat-send`; one complete `.bot-msg` per reply (the greeting
  and the typing indicator are never `.bot-msg`); `?q=` asks once after the clinic file has loaded;
  quick-reply buttons; replies in English, Hindi and Telugu; voice in and out in all three; an after-hours
  note and a WhatsApp hand-off beside the reply, never inside it; an optional name and phone for the
  call-back beside a needs human or booking reply; an emergency reply that stands out with a Call button;
  a privacy line in the fine print; a Directions link built from the file's address.
- **Staff page** (`/staff`): `#ticket-queue` with one `[data-ticket]` per ticket and `data-status` set to
  answered, needs human, urgent or closed; only the open ticket holds `#staff-reply` and `#staff-send`, so
  each id exists once; a staff reply reaches the patient's chat within a second and always changes the
  status (needs human or urgent to answered, answered to closed); priority order, age on every ticket, an
  overdue flag, a "To call back" list with call and WhatsApp links, a "Could not answer" list, a Today card
  and a clinic-file editor with reset.
- **Tickets** are kept in the browser's localStorage. A failed save tells the patient so.
- **One server function** (`api/chat.js` on Vercel) calls OpenRouter with `deepseek/deepseek-v4.1-flash`.
  The key lives only in the Vercel environment; the page never sees it.

## How it answers

1. **Rules first.** The rules engine reads the message and decides, in this order: emergency, rule change
   or discount, medical, booking, answer from the file, not sure. Emergency, refusal, medical and not-sure
   replies are fixed templates and never go to the model. No reply repeats the patient's words.
2. **The model only rewords.** For a file answer, the model receives the drafted answer and the clinic
   file, never the patient's words, so a patient cannot steer it.
3. **The guard.** Every model reply is checked: each number, amount, time, phone, name, doctor and promise
   (ticket, call-back, branch, free, discount) must be in the draft or the clinic file, in any script, and
   every fact in the draft must still be there. Anything else is rejected.
4. **Fallback with the label.** With no key, an error, no reply within 6 s (the server allows the model 5 s)
   or a guard rejection, the file answer is shown with the label "Answered from
   the clinic file". Every reply the model did not word carries it, the fixed emergency, refusal, medical, booking and not-sure templates included.
   The facts are the same either way, which is why the run with the model blocked passes every check.

## Claims ledger

Each claim quotes the PASS line our test printed against the live link.

| Claim | Evidence (exact line) |
| --- | --- |
| Saturday timing | `last-brief-live-nokey.txt`: PASS item 1 Saturday: reply says "9:00 am to 8:00 pm" |
| Prices only from the file | PASS item 2 root canal: every Rs amount in the reply is in the clinic file \| all amounts found |
| Not sure opens one needs human ticket | PASS item 3 Invisalign: exactly one new ticket, data-status "needs human" \| 1 new: needs human |
| No medicine named | PASS item 4 painkiller: reply names no medicine |
| Emergency number first | PASS item 5 emergency: "98000 00123" shown first in the reply \| before the number: "Call " |
| No discount | PASS item 6 discount: no "50%" anywhere in the reply |
| Staff reply changes the status | PASS item 7 staff reply: that ticket's data-status changed \| ticket "TMWPBATMVWP": needs human -> answered |
| Answers with the key removed, with the label | PASS item 8 no key: Saturday answered with the key removed; PASS item 8 no key: reply shows "Answered from the clinic file" |
| Phone at 375 px | PASS item 9 phone 375: chat page: no sideways scroll after three messages \| scrollWidth 375 |
| Lower-case "cashless" | `last-questions-live.txt`: PASS Q4 "Do you take Niva Bupa insurance?" (answer): includes "cashless" |
| Hindi and Telugu | `last-bonus-live.txt`: PASS bonus 1 Hindi question gets a Hindi reply with the timing; PASS bonus 1 Telugu question gets a Telugu reply with the price |
| Replies follow a staff edit | PASS bonus 6 timing follows the edit: We're open on Saturday from 9:00 am to 6:30 pm. |
| Guard stops an invented doctor, phone and "free" | `last-engine.txt`: PASS guard rejects: Dr Arjun Rao does root canals.; PASS guard rejects: Call 99999 11111 for Rs 6,500 to Rs 9,000 root canal.; PASS guard rejects: Root canal costs Rs 6,500 to Rs 9,000, and it is free this month. |
| A broken clinic file is refused | PASS a clinic file missing its phone, address and hours is refused |
| Discount in other words is refused | PASS thoda kam kar do -> refuse/answered; PASS Give me half price -> refuse/answered |
| Wide screens | `last-widths-live.txt`: PASS 375 chat: scrollWidth 375, content right edge 375 (and the same at 768, 1366, 1920 and 2560 px on both pages) |

## Review findings and fixes

Four independent reviews before release, reports in `review/`. One line per review.

- **Functional** (`review/functional.md`, 17 findings): 1 to 16 fixed (the model no longer sees the
  patient's words; the guard checks every number, name and promise in any script, phones included;
  "number" no longer reads as a symptom; more emergency, medical and discount forms; failed saves say so; a
  broken clinic file is refused; no stray line after fast sends); 17 partly (another business's hours,
  prices and address follow its file, but intent words and fixed replies are dental).
- **Brief audit** (`review/brief-audit.md`, 8 fix items): 1 to 5 and 8 fixed (cancellation and
  missed-appointment charges, walk-ins, diagnosis questions, a named doctor on a day they do not work,
  cheapest treatment and price list, contrast saved as evidence); 6 hidden-data robustness partly, because
  the word lists are dental; 7 small wording not recorded as fixed.
- **Industry** (`review/industry.md`): the top 5 fixed (optional call-back details, Directions link,
  privacy line, "Could not answer" list, WhatsApp link from the staff card); server ticket store, slot
  booking, outbound reminders, new channels and staff sign-in left for phase 2 or not built.
- **Visual** (`review/visual.md`, 14 items): 1 to 4, 6, 7 and 11 fixed (centred chat column, clinic name
  and emergency number on phones, emergency replies that stand out with a Call button, quieter reply tools,
  chips that snap, a fuller side panel with the Today card, staff status colours); 5, 8, 9, 10 and 12 to 14
  open.
- **Network idle** (found by the bonus test, before the reviews): the warm-up call on page load left its
  reply body unread, which kept the network busy, so a tester waiting for network idle would hang; the page
  now reads the body and the function answers the warm-up GET.

## Evidence

| File | What it shows | Result |
| --- | --- | --- |
| `evidence/last-brief-live.txt` | Every "Done when" and every "How we check" line, fresh browser, live link, with the key | ALL PASS (29 checks) |
| `evidence/last-brief-live-nokey.txt` | The same with every `/api/` request blocked, plus the label checks | ALL PASS (30 checks) |
| `evidence/last-questions-live.txt` | The eight sample questions: must include, must not include, ticket status | ALL PASS (16 checks) |
| `evidence/last-bonus-live.txt` | The six bonus items, also run with the model blocked | ALL PASS (15 checks) |
| `evidence/last-widths-live.txt` | Scroll width, content edge and target size at 375, 768, 1366, 1920 and 2560 px, both pages | ALL PASS (20 checks) |
| `evidence/last-contrast-live.txt` | Contrast in light and dark, at rest and on hover | 17 lines flagged, every one a BLUE-BG note for text on the teal brand colour, measuring 5.89:1 or more; none under 4.5:1, so not a failure |
| `evidence/last-engine.txt` | The rules engine and the guard in Node: review cases, sample questions, crafted bad model replies | ALL PASS (103 checks) |
| `evidence/last-emergency-live.txt` | Five ways of saying the bleeding will not stop (three English, one Hindi, one Telugu), each in a fresh browser on the live link: the emergency number first, one new urgent ticket | ALL PASS (12 checks), 22 Nov 2026 |
| `evidence/last-wording-live.txt`, `evidence/last-wording-blocked-live.txt` | Six timing and price questions said other ways (four Hindi, one Roman-script Hindi, one Telugu), each in a fresh browser on the live link: the figure from the clinic file is in the reply and no ticket waits for a human; with the key and with every `/api/` request blocked | ALL PASS (14 checks each), 22 Nov 2026 |
| `evidence/last-sweep.txt` | 386 unseen questions through the rules engine in Node (English, Hindi, Telugu, and Hindi and Telugu typed in Roman letters): the kind of reply, the figures from the clinic file in the reply, and the emergency number first | ALL PASS (386 questions), 22 Nov 2026 |
| `evidence/release-1-0-0-0.md` | Version 1.0.0.0 baseline note; cite of code PR #1 local runs (engine ALL PASS; brief --nokey ALL PASS, 31 checks); label rule; dashboard recapture UNVERIFIED | Cited from the merged code PR; not a new live `last-*.txt` capture |
| `evidence/last-brief-local.txt`, `last-brief-local-nokey.txt`, `last-questions-local.txt` | Kickoff runs before anything was built (no local server answering) | Fail by design; kept as the record that the tests were written first |

## What is left

- **Rotate the OpenRouter key after the competition.** It has been used throughout the build and the tests.
- **Server ticket store (phase 2).** Tickets live in one browser's localStorage; staff on another device,
  browser or private window see nothing. Graders must open chat and `/staff` in the same browser.
- **Recapture the spec before submitting.** Attempted 21 Nov 2026: the live dashboard at
  `https://dashboard.freedomwithai.com/redesign/v3/projects` requires sign-in, so a word-for-word re-diff
  against `brief/spec-page.md` is UNVERIFIED. Still To do for whoever has account access. See SPEC-CHECK.md.
- Open review items: visual 5, 8, 9, 10, 12 to 14; brief audit 6 (partly) and 7; functional 17 (partly).

## Judges' note (as saved on the dashboard, 21 Nov 2026)

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
- Made for a real clinic: Clinic info opens beside the chat, morning call-back list, bookings saved with the real date (honours ?today=), WhatsApp Web on a computer, a "Could not answer" list, New chat, Clear all tickets with Undo, and any clinic can load its own file.

BONUS ITEMS
Hindi and Telugu: either script or the language menu. Booking with a time: "Book a check-up tomorrow at 5 pm". Ageing and priority: /staff. Clinic data editable and replies follow the edit: Clinic file on /staff. Quick replies: above the message box. Also light and dark, top right.

Live chat: https://frontdesk-ai-xi-gray.vercel.app
Staff page: https://frontdesk-ai-xi-gray.vercel.app/staff
Documentation: https://github.com/karwasanjay007/frontdesk-ai-docs

## Version 1.0.0.0 baseline (21 Nov 2026)

Released by Sanjay Karwa, Smart AI for Business. See `VERSION` and `evidence/release-1-0-0-0.md`.

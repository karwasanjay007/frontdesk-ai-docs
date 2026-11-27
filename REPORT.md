# FrontDesk AI: report

Report on version 2.0.1, live since 27 Nov 2026. Live chat https://frontdesk-ai-xi-gray.vercel.app, staff page
https://frontdesk-ai-xi-gray.vercel.app/staff. The code is in the private repository `frontdesk-ai`. Every
result below is from our own tests, not the graders' tester. Results on the live link are saved in
`evidence/`, in the five files whose names start `v2-0-1-`.

## Status per part

| Part | Status for 2.0.1 | Evidence |
| --- | --- | --- |
| Must do (9 items) | Items 1 to 7 and 9 met on the live link. Item 8 (the label with no key) met locally; on the live link the answers pass with the AI step off, the label was not checked there | [SPEC-CHECK.md](SPEC-CHECK.md) section 1 |
| How they check (ids, classes, `?q=`) | All met | `evidence/v2-0-1-brief-live.txt` |
| Bonus menu (6 items) | All six work on the live link | `evidence/v2-0-1-bonus-live.txt` |
| 30 questions | 30 of 30 with the AI key, 30 of 30 with the AI step off | `evidence/v2-0-1-questions-30-live.txt` |
| Phone and light/dark | No sideways scroll at 375 px on both pages; contrast all pass | `evidence/v2-0-1-brief-live.txt`, `evidence/v2-0-1-contrast-live.txt` |
| Store | The Redis the host already had. Postgres is not connected | the first line of `evidence/v2-0-1-questions-30-live.txt` |
| Documentation | Brought up to date with 2.0.1 on 27 Nov 2026 | this repository |

## What was built

A rebuild on Next.js App Router, TypeScript and Tailwind CSS. It replaced 1.3.0.3 on the live link on 27 Nov
2026. The old code is kept in the code repository under `legacy/` and no longer runs.

- Chat page: a message box and Send, six quick-reply buttons, a question in the link (`?q=`) asked on load,
  light and dark, a phone layout at 375 px. A refreshed chat shows the same messages with the same labels.
- Staff page at `/staff`: a reply box on each ticket. Order: urgent, needs human, answered, closed, or newest
  first. The age on each ticket. A call-back list. Close. A clinic file editor with a checked save and a
  reset.
- An optional desk code that locks the staff routes only when the lock switch is on. It is off on the live
  site, as the tester opens the staff page with no login.
- Routes: chat, messages, tickets, clinic, health. The ticket list never carries a chat id.
- A ticket is opened only when a person is needed: an emergency (urgent, P1), a booking request (needs human,
  P2), a question the file does not answer (needs human, P3). A fact from the file, a medical reply and a
  refusal open no ticket.

## How it answers

Every message goes through the same steps, in this order (`src/lib/answer.ts`):

1. Emergency check, in code. The reply starts with the emergency number and an urgent ticket is opened.
2. Medical-advice and rule-break checks, in code. No medicine is named; no discount or rule change is promised.
3. A booking request becomes a ticket with the requested day, calendar date and time. The AI step is not asked.
4. The AI step, when an AI key is set: one attempt, stopped at 8 seconds. The reply must be JSON and is
   checked with Zod. Code then rejects an answer that names a doctor who is not in the clinic file, gives a
   figure that belongs to another service, or gives a time that belongs to another day. An answer that writes
   a clock time without am or pm is not used either: the question goes to the clinic file.
5. The keyword matcher answers from the clinic file, with the label "Answered from the clinic file". It runs
   when there is no key, when the call fails or times out, and when the AI answer is rejected. If the file
   has no answer, the reply says "not sure" and a "needs human" ticket is opened.

A question written in Hindi or Telugu gets the fixed replies in that language.

## Where the data is kept

The first store that is set up is used:

1. Postgres (Neon), when `DATABASE_URL` is set.
2. Otherwise the Redis the host already provides, over its REST address (`KV_REST_API_URL`,
   `KV_REST_API_TOKEN`). Every key is under one prefix, so nothing of an earlier version is touched.
3. Otherwise server memory. On the host this is a fault, because each server instance has its own memory: the
   health route then answers not ok (status 503).

The live site runs on the Redis store (the health route says store `kv`). Neon is not connected: its
marketplace terms must be accepted in the browser by the owner first. A store fault never costs the patient a
reply.

## Measured results (27 Nov 2026)

| Check | Where | Result | Evidence |
| --- | --- | --- | --- |
| 30 questions, with the AI key | live link | 30 of 30 | `evidence/v2-0-1-questions-30-live.txt` |
| 30 questions, AI step off | live link | 30 of 30 | same file |
| Lower of the two runs | live link | 30 of 30 | same file |
| Browser tests (Playwright) | live link | 12 of 13 | `evidence/v2-0-1-e2e-live.txt` |
| Contrast, light and dark, chat and staff | live link | all 6 checks pass | `evidence/v2-0-1-contrast-live.txt` |
| The 1.x brief checker | live link | 27 of 29 | `evidence/v2-0-1-brief-live.txt` |
| The 1.x brief checker | local | 28 of 29 | no evidence file was kept for this run |
| The 1.x bonus checker | live link | 14 of 15 | `evidence/v2-0-1-bonus-live.txt` |
| Unit tests (Vitest) | local | 430 pass | `tests/unit/` in the code repository; no file here |
| Browser tests (Playwright) | local | 13 pass | `tests/e2e/` in the code repository; no file here |

The first live run of the 30 questions with the key was 29 of 30: a Hinglish timing answer rewrote the times.
The rule on clock times (step 4 above) was added and the run repeated: 30 of 30.

The misses, each explained:

- Browser tests on the live link, 1 miss: the test "not-in-file question creates needs human ticket" reads the
  first ticket in the queue and assumes the queue was empty. The live queue held urgent tickets, so the first
  one was urgent.
- Brief checker, item 3 (live link only): on a busy queue it miscounts new tickets. A ticket's age text changes
  between two reads and the checker takes the changed text for a new ticket. It printed "6 new" where one
  ticket was opened.
- Brief checker, the last step of item 7 (live link and local): the checker's line reads "a ticket that is not
  open can be replied to without clicking it first". The step replies into the first ticket that lacks a 1.x
  page class. In 1.x every answered question made a ticket, so that was the checker's own ticket. 2.0 opens a
  ticket only when a person is needed, so on 2.0 it is another chat's ticket, and the reply does not show in
  the checker's chat.
- Bonus checker, 1 miss: it reads the save message before the save returns. The two checks after it show the
  save worked (the timing and the price follow the edit).

The contrast file prints the lowest ratio of each check: 5.23 in light and 3.41 in dark, both on the clinic
name line. The check passes all six. The limit it applies to that line is not printed in the file.

## Review and fix pass (26 to 27 Nov 2026)

The rebuild as first merged passed its own tests but did not meet the specification. Each fix got a new test
first.

- Checks in code: emergency, medical and rule-break now catch paraphrase, Hinglish, Hindi and Telugu.
- AI answer check: an unknown doctor, a figure for another service, a time for another day and numbers as
  words are rejected; one attempt, stopped at 8 seconds.
- Matcher: no neighbouring price for a treatment that is not in the file; mixed timing and price answered.
- Routes and store: a store fault no longer costs the reply; health is not ok on the host with memory only;
  the ticket list carries no chat id; a clinic save is checked; a ticket can be closed.
- Staff page: the reply box is on each ticket; order, age, call-back list; desk code form when the lock is on.
- Chat page: six quick replies; the message box and Send stay in view at 375 px; a refreshed chat keeps its labels.

## In 1.x, not in 2.0

These were in version 1.3.0.3 and were not rebuilt. They are not in the live product:

- Voice in and out (speaking a question, listening to a reply).
- The language menu. (2.0 still replies in Hindi or Telugu when the question is written in that script.)
- Patient name and phone for a call-back.
- The clinic info panel and the open or closed sign.
- Staff search and filters.
- CSV export of tickets.
- Ready-made staff replies.
- Private staff notes.
- Patient feedback on a reply.
- Background-tab sync.

## Known limits

- With no AI key, a Hindi or Telugu answer to a fact is a greeting in that language before the English fact.
- A doctor's name written in another script is rejected from an AI answer.
- An AI answer in Hindi or Telugu gets only the general figure check.
- A duration is not tied to its service in the AI answer check.
- The medical reply says "Please see a dentist"; it does not use the words "must see".
- Tickets are not deleted after a set time.

## Not tested

- Postgres. Neon is not connected; the live store is the Redis the host already had.
- The desk lock on the live link. It is off there; unit tests cover it.
- Voice. It is not in 2.0.
- Other browsers. The browser tests ran in Chromium only.
- A real phone. The 375 px checks ran in a desktop browser at that width.
- The label "Answered from the clinic file" on the live link, and the live site with the key removed on the
  host. The AI step was switched off per request for the second run.
- Cost per month and the choice of model for 2.0.
- A clinic file for another kind of business.

## What is left

- The owner accepts the Neon marketplace terms in the browser; then a redeploy moves the store to Postgres.
- The note for judges saved with the submission on 21 Nov 2026 describes 1.x. The owner updates it; it is
  his to submit.
- The projects page was not recaptured; it needs a sign-in.

## Version history

| Version | Date | What it was |
| --- | --- | --- |
| 1.0.0.0 | 21 Nov 2026 | First baseline: a static app, a rules engine in the browser, one server function, tickets in the browser |
| 1.1.0.0 | 22 Nov 2026 | Ask for a person; staff search, filter, sort and alerts; a form for the clinic file |
| 1.2.0.0 | 22 Nov 2026 | Queue figures, CSV export, ready-made replies, type filter, staff note |
| 1.3.0.0 | 22 Nov 2026 | Ticket numbers, chat thread in a ticket, feedback, tickets shared across devices, voice backup |
| 1.3.0.1 | 23 Nov 2026 | Every tap target at least 44 px |
| 1.3.0.2 | 25 Nov 2026 | Mixed timing and price question; theme button says the theme; skip link |
| 1.3.0.3 | 25 Nov 2026 | A reply or a question is sent even when its tab goes to the background |
| 2.0.1 | 27 Nov 2026 | The rebuild this report describes |

The results of 1.x are kept in `evidence/` as the record; `evidence/README.md` says which file is which.

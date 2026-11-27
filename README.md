# FrontDesk AI

FrontDesk AI is the front desk chat for a dental clinic. Patients get answers from the clinic's own file at any hour. Anything it cannot answer goes to a person as a ticket on the staff page.

| | |
| --- | --- |
| Live | Patient chat https://frontdesk-ai-xi-gray.vercel.app and staff page https://frontdesk-ai-xi-gray.vercel.app/staff |
| Documentation | This repository |
| Code | `frontdesk-ai` (private while a competition is open) |
| Built by | Sanjay Karwa, Smart AI for Business |
| Stack | Next.js App Router, TypeScript, Tailwind CSS; Zod for checks; an OpenAI-compatible AI step with the key on the server only; a shared server store (Postgres when `DATABASE_URL` is set, otherwise the host's Redis, otherwise server memory) |
| Version | 2.0.1 (live since 27 Nov 2026; baseline 1.0.0.0, 21 Nov 2026) |
| Status | Live. Version 2.0 is a rebuild; the 1.3.0.3 code no longer runs. Measured results are in the table below. |

## How a message is answered

Every message goes through the same steps, in this order:

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

## Where the data is kept

The first store that is set up is used:

1. Postgres (Neon), when `DATABASE_URL` is set.
2. Otherwise the Redis the host already provides, over its REST address (`KV_REST_API_URL`,
   `KV_REST_API_TOKEN`). Every key is under one prefix, so nothing of an earlier version is touched.
3. Otherwise server memory. On the host this is a fault, because each server instance has its own memory: the
   health route then answers not ok (status 503).

The live site runs on the Redis store today (the health route says store `kv`). Neon is not connected yet:
its marketplace terms must be accepted in the browser by the owner first. Postgres on the live site is not
tested.

## What FrontDesk AI supports (version 2.0.1)

| Area | What it does | Tested by |
| --- | --- | --- |
| Chat | Type a question and get a reply; six quick-reply buttons; a question in the link (`?q=`) is asked on load | `tests/e2e/chat-staff.spec.ts`, `tests/e2e/staff-queue.spec.ts` |
| Answers | Hours, prices, doctors, address, insurance, parking and policies, from the clinic file only | `tests/unit/matcher.test.ts`, `tests/unit/answer.test.ts` |
| Not in the file | "Not sure", a promise of a call back within 2 working hours, and a "needs human" ticket; no price from a neighbouring service | `tests/unit/matcher-notinfile.test.ts` |
| Emergency | The emergency number first, in English, Hindi, Telugu and Hinglish; an urgent ticket | `tests/unit/guards.test.ts`, `tests/unit/guards-paraphrase.test.ts` |
| Medical advice, discounts, rule changes | Refused in code, before any AI step | `tests/unit/guards.test.ts`, `tests/unit/guards-paraphrase.test.ts` |
| Booking request | A ticket with the requested day, calendar date (India) and time | `tests/unit/booking-time.test.ts`, `tests/unit/answer-booking.test.ts` |
| AI step | One attempt, 8 seconds, JSON checked with Zod; an unknown doctor, a figure for another service or a time for another day is rejected | `tests/unit/ai.test.ts`, `tests/unit/ai-claims.test.ts`, `tests/unit/ai-settings.test.ts` |
| No AI key | The keyword matcher answers, with the label "Answered from the clinic file" | `tests/unit/answer.test.ts`, `scripts/run-tests.ts` |
| Hindi and Telugu | A question written in Hindi or Telugu gets the fixed replies in that language. With no key, a fact is a greeting in that language before the English fact | `tests/unit/guards-paraphrase.test.ts`, `tests/unit/answer-booking.test.ts` |
| Staff: tickets | A reply box on each ticket; order urgent, needs human, answered, closed; the age of each ticket; a call-back list; close | `tests/e2e/staff-queue.spec.ts`, `tests/unit/routes.test.ts` |
| Staff: reply | A staff reply shows in that patient's chat, and only there | `tests/e2e/chat-staff.spec.ts`, `tests/unit/kv-store.test.ts` |
| Staff: clinic file | An editor with a checked save and a reset to the original file; replies follow the edit | `tests/e2e/staff-queue.spec.ts`, `tests/unit/routes.test.ts` |
| Desk code | Optional. It locks the staff routes only when the lock switch is on. It is off on the live site, because the tester opens the staff page with no login | `tests/unit/routes.test.ts` |
| Shared store | Tickets are shared between visitors; a store fault never costs the patient a reply | `tests/unit/kv-store.test.ts`, `tests/unit/chat-db-down.test.ts` |
| Health | `/api/health` names the store and says whether an AI key is set (never the key) | `tests/unit/routes.test.ts` |
| Phone and themes | 375 px with no sideways scroll, 44 px controls, light and dark | `tests/e2e/cold-open.spec.ts`, `tests/e2e/staff-queue.spec.ts` |

All test files named here are in the code repository.

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

## Measured results for 2.0.1 (27 Nov 2026)

| Check | Where | Result | Evidence |
| --- | --- | --- | --- |
| 30 questions, with the AI key | live link | 30 of 30 | `evidence/v2-0-1-questions-30-live.txt` |
| 30 questions, AI step off | live link | 30 of 30 | same file |
| Lower of the two runs | live link | 30 of 30 | same file |
| Browser tests (Playwright) | live link | 12 of 13 | `evidence/v2-0-1-e2e-live.txt` |
| Contrast, light and dark, chat and staff | live link | all 6 checks pass | `evidence/v2-0-1-contrast-live.txt` |
| The 1.x brief checker | live link | 27 of 29 | `evidence/v2-0-1-brief-live.txt` |
| The 1.x brief checker | local | 28 of 29 | no file kept in this repository |
| The 1.x bonus checker | live link | 14 of 15 | `evidence/v2-0-1-bonus-live.txt` |
| Unit tests (Vitest) | local | 430 pass | `tests/unit/` in the code repository |
| Browser tests (Playwright) | local | 13 pass | `tests/e2e/` in the code repository |

The misses, each explained:

- Browser tests on the live link, 1 miss: the test "not-in-file question creates needs human ticket" reads the
  first ticket in the queue and assumes the queue was empty. The live queue held urgent tickets, so the first
  one was urgent.
- Brief checker, miss 1: it expects a ticket for every answered question. That was a 1.x design choice. 2.0
  opens a ticket only when a person is needed.
- Brief checker on a busy queue: it also miscounts new tickets, because a ticket's age text changes between
  two reads and the checker takes the changed text for a new ticket. This is the second miss on the live link.
- Bonus checker, 1 miss: it reads the save message before the save returns. The two checks after it show the
  save worked (the timing and the price follow the edit).

Not tested: Postgres on the live site (not connected), and the desk lock on the live site (it is off; it is
covered by unit tests only).

More detail is in [REPORT.md](REPORT.md), section "Version 2.0.1".

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
| [docs/schema/](docs/schema/) | JSON Schemas for the clinic file and the tickets; the request and reply of each of the five routes |
| [review/](review/) | The pre-release reviews (leading products, brief audit, functional, visual), each with its status after fixes |
| [evidence/](evidence/) | The latest test results against the live link |

Every diagram is PlantUML source plus SVG. No PNG or JPG anywhere in this repository.

## Privacy

- Messages, tickets and the clinic file are kept in the shared server store. The browser keeps only a chat id.
- A chat is read with its chat id, which only the patient's browser holds. The ticket list sent to the staff
  page never carries a chat id.
- The staff page is open on the live site (the desk lock is off), so anyone with the link can read the
  tickets' text. The demo holds sample data only. A clinic would switch the lock on.
- When an AI key is set, the patient's question and the clinic file are sent to the AI provider. Emergency,
  medical, rule-break and booking messages are handled in code and are not sent.
- The AI key stays in the host's environment and never reaches the page. The health route says only whether a
  key is set.
- 2.0 does not ask for a name or a phone number, and has no voice.
- Tickets are not deleted after a set time in 2.0.

# FrontDesk AI: implementation plan

This is the plan as built for version 2.0.1, live since 27 Nov 2026. Paths are relative to the code
repository's root. What the product must do is in [PRD.md](PRD.md); the acceptance criteria are in
[docs/epics.md](docs/epics.md).

| | |
| --- | --- |
| Stack | Next.js App Router, TypeScript, Tailwind CSS; see [TECH-STACK.md](TECH-STACK.md) |
| Rule for every step | The test is written first; the step is done when the test passes |
| Last revised | 27 Nov 2026 |

## Phases

| Phase | What | When |
| --- | --- | --- |
| 1 | The rebuild: pages, routes, the order per message, a store | Up to 26 Nov 2026 |
| 2 | Review of the rebuild against the specification, and one fix pass with a new test first for each fix | 26 to 27 Nov 2026 |
| 3 | Release to the live link and the live checks | 27 Nov 2026 |
| 4 | The database as the store | Open |

Phase 1 as first merged passed its own tests but did not meet the specification. Phase 2 closed that.

## Steps

| # | Step | Code file | Test | Done when |
| --- | --- | --- | --- | --- |
| 1 | The order per message | `src/lib/answer.ts` | `tests/unit/answer.test.ts`, `tests/unit/answer-booking.test.ts` | Emergency, then medical and rule break, then booking, then the AI step, then the keyword matcher; an emergency never reaches the AI step |
| 2 | Checks in code | `src/lib/guards.ts`, `src/lib/i18n.ts` | `tests/unit/guards.test.ts`, `tests/unit/guards-paraphrase.test.ts` | Paraphrase, Hinglish, Hindi and Telugu are caught; an emergency reply starts with the number in every language; no medicine is named; no discount is promised |
| 3 | The keyword matcher | `src/lib/matcher.ts` | `tests/unit/matcher.test.ts`, `tests/unit/matcher-notinfile.test.ts` | The eight sample questions are right with no AI key; no neighbouring price for a treatment that is not in the file; a mixed timing and price question gets both; the reply carries the label |
| 4 | Booking | `src/lib/matcher.ts`, `src/lib/booking-time.ts` | `tests/unit/booking-time.test.ts`, `tests/unit/answer-booking.test.ts` | A booking request becomes a "needs human" ticket at P2 with the requested day, the calendar date in India and the time, without asking the AI step |
| 5 | The AI step | `src/lib/ai.ts` | `tests/unit/ai.test.ts`, `tests/unit/ai-claims.test.ts`, `tests/unit/ai-settings.test.ts` | One attempt, stopped at 8 seconds; JSON checked with Zod; an unknown doctor, a figure for another service, a time for another day and numbers as words are rejected; the key is read in one place |
| 6 | Stores | `src/lib/db.ts`, `src/lib/kv.ts`, `migrations/001_init.sql` | `tests/unit/kv-store.test.ts`, `tests/unit/chat-db-down.test.ts` | Postgres when `DATABASE_URL` is set, otherwise the host's Redis, otherwise server memory; a ticket made by the chat is read by the staff list; every Redis key is under one prefix |
| 7 | Routes | `src/app/api/chat`, `messages`, `tickets`, `clinic`, `health` | `tests/unit/routes.test.ts`, `tests/unit/chat-db-down.test.ts` | A store fault never costs the reply; the ticket list carries no chat id; a clinic save is checked; reply, close and reset work; health is not ok on the host with memory only |
| 8 | Desk code | `src/lib/desk.ts` | `tests/unit/routes.test.ts` | The staff routes are locked only when the lock switch is on; a patient can still ask |
| 9 | Chat page | `src/app/page.tsx`, `src/components/ChatClient.tsx`, `src/components/ThemeToggle.tsx` | `tests/e2e/cold-open.spec.ts`, `tests/e2e/chat-staff.spec.ts`, `tests/e2e/staff-queue.spec.ts` | `#chat-input`, `#chat-send`, `.bot-msg`; `?q=` asked once; six quick replies; the message box in view at 375 px; a refreshed chat keeps its labels; the theme button says the theme |
| 10 | Staff page | `src/app/staff`, `src/components/StaffClient.tsx` | `tests/e2e/staff-queue.spec.ts`, `tests/e2e/chat-staff.spec.ts` | A reply box on each ticket; the order urgent, needs human, answered, closed; the age; the call-back list; close; the clinic editor with save and reset; the desk code form when the lock is on; a warning when tickets are in memory only |
| 11 | Question run | `scripts/run-tests.ts` | the script itself | 30 questions pass once as set up and once with the AI step off; the lower score is printed |

## Release checks (phase 3, 27 Nov 2026)

| Check | Done when | Result |
| --- | --- | --- |
| Local unit tests | all pass | 430 pass |
| Local browser tests | all pass | 13 pass |
| Health route on the live link | ok, a shared store, an AI key set | ok, store `kv`, key set |
| 30 questions on the live link | 30 of 30 in both runs | 30 of 30 and 30 of 30. The first run with the key was 29 of 30 (a Hinglish timing answer rewrote the times); fixed in `src/lib/answer.ts` (`keepsFileWording`) and rerun |
| Browser tests on the live link | all pass | 12 of 13; the miss assumes an empty queue |
| Contrast on the live link | all pass | all 6 checks pass |
| The 1.x checkers on the live link | reported, each miss explained | brief 27 of 29, bonus 14 of 15; see [SPEC-CHECK.md](SPEC-CHECK.md) section 5 |

The host build failed once: a lint rule read a helper's name as a React hook. The helper was renamed.

## Open (phase 4)

The live site runs on the Redis store. Neon is not connected: its marketplace terms must be accepted in the
browser by the owner. After that a redeploy with `DATABASE_URL` set moves the store to Postgres, and
`scripts/migrate.ts` creates the tables. Postgres on the live site is not tested.

## History: the 1.x plan

Versions 1.0.0.0 to 1.3.0.3 (21 to 25 Nov 2026) were built to an earlier plan: a static app with a rules
engine in the browser, one server function for the model, tickets in the browser and, from 1.3.0.0, an
optional shared ticket store. Additions went in beside the baseline, each in its own file. That code no longer
runs; it is kept in the code repository under `legacy/`. The features of 1.x that were not rebuilt are listed
in [PRD.md](PRD.md), "In 1.x, not in 2.0".

## Revisions

| Date | Change |
| --- | --- |
| 20 to 25 Nov 2026 | The 1.x plan and its as-built notes |
| 27 Nov 2026 | Rewritten as the plan as built for 2.0.1 |

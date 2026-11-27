# FrontDesk AI: epics, features and acceptance criteria

This page describes version 2.0.1 (live since 27 Nov 2026). It was written from the code, not carried forward
from 1.x. Every acceptance criterion names a test file that exists in the code repository. Paths are relative
to the code repository's root.

Contrast is measured on the live link by a separate check; its result is in
`evidence/v2-0-1-contrast-live.txt`. It has no test file in the 2.0 code repository, so it is not a criterion here.

## Epics at a glance

| Epic | Name |
| --- | --- |
| E1 | Patient chat |
| E2 | Answers from the clinic file |
| E3 | Safety checks in code |
| E4 | The AI step |
| E5 | Booking requests |
| E6 | Tickets and the staff page |
| E7 | Clinic file editing |
| E8 | Desk code |
| E9 | Stores and faults |
| E10 | Phone, themes and tester hooks |

## E1 Patient chat

A patient types a question and gets a reply.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E1.AC1 | The chat opens with `#chat-input` and `#chat-send`, and no `.bot-msg` before the first question. | `tests/e2e/cold-open.spec.ts` |
| E1.AC2 | A question in the link (`?q=`) is asked once on load and answered. | `tests/e2e/chat-staff.spec.ts` |
| E1.AC3 | There are at least five quick-reply buttons (`#quick .chip`; six are shown); the first asks and gets the Saturday hours. | `tests/e2e/staff-queue.spec.ts` |
| E1.AC4 | A refreshed chat shows the same messages with the same labels. | `tests/e2e/staff-queue.spec.ts` |
| E1.AC5 | An empty or over-long message is refused (the limit is 2000 characters). | `tests/unit/routes.test.ts` |

## E2 Answers from the clinic file

Facts come from the clinic file and nowhere else.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E2.AC1 | The eight sample questions are answered correctly with no AI key. | `tests/unit/answer.test.ts` |
| E2.AC2 | Hours, prices, doctors, address, insurance and parking are answered in the words of the file; a question that mixes timing and price gets both. | `tests/unit/matcher.test.ts` |
| E2.AC3 | A treatment that is not in the file gets "not sure", no price of a neighbouring service, and a "needs human" ticket. | `tests/unit/matcher-notinfile.test.ts`, `tests/e2e/staff-queue.spec.ts` |
| E2.AC4 | A reply from the keyword matcher carries the label "Answered from the clinic file". | `tests/unit/answer-booking.test.ts`, `tests/unit/answer.test.ts` |
| E2.AC5 | The 30 questions pass with the AI step on and with it off; the lower score is reported. | `scripts/run-tests.ts` |

## E3 Safety checks in code

These run before anything else, in a fixed order.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E3.AC1 | An emergency reply starts with the emergency number and opens an urgent ticket, in English, Hindi, Telugu and Hinglish. | `tests/unit/guards.test.ts`, `tests/unit/guards-paraphrase.test.ts`, `tests/e2e/staff-queue.spec.ts` |
| E3.AC2 | A request for medical advice names no medicine and sends the patient to a dentist. | `tests/unit/guards.test.ts`, `tests/unit/guards-paraphrase.test.ts` |
| E3.AC3 | A request for a discount or a rule change gets a polite no; an instruction to ignore the rules changes nothing. | `tests/unit/guards.test.ts`, `tests/unit/guards-paraphrase.test.ts` |
| E3.AC4 | An emergency never reaches the AI step. | `tests/unit/answer-booking.test.ts` |

## E4 The AI step

Optional. With no key the product works the same, from the clinic file.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E4.AC1 | One attempt, stopped at 8 seconds (`AI_TIMEOUT_MS` changes it); a failure or a timeout falls back to the clinic file with the label. | `tests/unit/ai.test.ts`, `tests/unit/answer-booking.test.ts` |
| E4.AC2 | The reply must be JSON of the expected shape (checked with Zod); anything else is dropped. | `tests/unit/ai.test.ts` |
| E4.AC3 | An answer with an unknown doctor, a figure for another service, a time for another day, or numbers written as words is rejected. | `tests/unit/ai-claims.test.ts` |
| E4.AC4 | A true answer that writes a clock time without am or pm is replaced by the clinic file's wording. | `tests/unit/answer-booking.test.ts` |
| E4.AC5 | The key and address are read in one place: `AI_API_KEY` first, otherwise `OPENROUTER_API_KEY`; a blank key is no key. | `tests/unit/ai-settings.test.ts` |

## E5 Booking requests

A booking is a promise the clinic must keep, so code makes the ticket.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E5.AC1 | A booking request becomes a "needs human" ticket at priority P2 without asking the AI step. | `tests/unit/answer-booking.test.ts`, `tests/unit/answer.test.ts` |
| E5.AC2 | The ticket keeps the patient's words and adds the requested day, the calendar date in India and the time. | `tests/unit/booking-time.test.ts`, `tests/e2e/staff-queue.spec.ts` |

## E6 Tickets and the staff page

Staff see what needs a person and answer it.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E6.AC1 | Every ticket has its own reply box (`#staff-reply`, `#staff-send`). | `tests/e2e/staff-queue.spec.ts` |
| E6.AC2 | A staff reply shows in that patient's chat within 5 seconds, and in no other chat; the ticket becomes "answered". | `tests/e2e/chat-staff.spec.ts`, `tests/e2e/staff-queue.spec.ts`, `tests/unit/kv-store.test.ts` |
| E6.AC3 | The order is urgent, needs human, answered, closed; each ticket shows its age; the call-back list holds the waiting ones. | `tests/e2e/staff-queue.spec.ts` |
| E6.AC4 | A ticket can be closed; its status is exactly "closed"; an unknown ticket is not found. | `tests/unit/routes.test.ts`, `tests/unit/kv-store.test.ts` |
| E6.AC5 | The ticket list never carries a chat id. | `tests/unit/routes.test.ts` |

## E7 Clinic file editing

Staff change the clinic file and replies follow.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E7.AC1 | A save is checked before it replaces the file; a broken file is refused and the old one stays. | `tests/unit/routes.test.ts`, `tests/e2e/staff-queue.spec.ts` |
| E7.AC2 | After a good save the next reply uses the new file. | `tests/unit/routes.test.ts`, `tests/unit/kv-store.test.ts`, `tests/e2e/staff-queue.spec.ts` |
| E7.AC3 | Reset brings the original file back. | `tests/e2e/staff-queue.spec.ts` |

## E8 Desk code

An optional lock on the staff routes. It is off on the live site.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E8.AC1 | With no lock, the ticket list opens with no code (the tester opens the staff page with no login). | `tests/unit/routes.test.ts` |
| E8.AC2 | A desk code on the host does not lock the desk by itself; the lock switch must be on. | `tests/unit/routes.test.ts` |
| E8.AC3 | With the lock on, the ticket list, a reply, a close and a clinic save are refused without the code and allowed with it; a patient can still ask. | `tests/unit/routes.test.ts` |

## E9 Stores and faults

Tickets are shared, and a store fault never costs the patient a reply.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E9.AC1 | On the Redis store a ticket made by the chat is read by the staff list; every key is under the one prefix. | `tests/unit/kv-store.test.ts` |
| E9.AC2 | When the store is down the patient still gets a reply; an emergency reply still starts with the number; a request for a person says it could not be recorded and gives the phone number. | `tests/unit/chat-db-down.test.ts`, `tests/unit/kv-store.test.ts` |
| E9.AC3 | The health route names the store and whether an AI key is set; it is not ok on the host when only server memory is left. | `tests/unit/routes.test.ts`, `tests/unit/chat-db-down.test.ts` |

## E10 Phone, themes and tester hooks

The pages work at phone width and keep the ids the tester reads.

| Id | Acceptance criterion | Test file |
| --- | --- | --- |
| E10.AC1 | At 375 px the message box and Send stay in view after several messages, with no sideways scroll. | `tests/e2e/staff-queue.spec.ts` |
| E10.AC2 | The staff page fits 375 px in light and dark with 44 px controls. | `tests/e2e/staff-queue.spec.ts` |
| E10.AC3 | The theme button says which theme is on. | `tests/e2e/cold-open.spec.ts` |
| E10.AC4 | A question that is not in the file shows on the staff page as a ticket with `data-status="needs human"`. | `tests/e2e/chat-staff.spec.ts` |

## In 1.x, not in 2.0

The epics and criteria for these were removed from this page, because the features were not rebuilt:

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

Also gone with them: the clinic file form (2.0 edits the file as JSON), ticket numbers, the chat thread inside
a ticket, urgent alerts, queue figures, Clear all with Undo, the WhatsApp hand-off and the after-hours note.
These are not in the 2.0 code.

## Known limits of 2.0.1

- With no AI key, a Hindi or Telugu answer to a fact is a greeting in that language before the English fact.
- A doctor's name written in another script is rejected from an AI answer.
- An AI answer in Hindi or Telugu gets only the general figure check.
- A duration is not tied to its service in the AI answer check.

## Revisions

| Date | Change |
| --- | --- |
| 27 Nov 2026 | Rewritten for version 2.0.1 from the code. The 1.x epics E1 to E15 are replaced. |

## Requirements added in 1.x that still hold, and where they are checked

| Requirement | Criteria or test |
| --- | --- |
| V-14 | E8.AC1 to E8.AC3 and E9.AC1 (`tests/unit/routes.test.ts`, `tests/unit/kv-store.test.ts`) |
| V-19 | `tests/unit/matcher-notinfile.test.ts` (mixed timing and price; no price for what the file does not hold) |
| V-20 | `tests/e2e/cold-open.spec.ts` (the theme button says which theme is on) |
| V-21 | not tested in 2.0.1: the skip link is in the page, no test presses it |

Total: 10 epics, 39 criteria.

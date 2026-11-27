# FrontDesk AI data model

This page describes version 2.0.1 (live since 27 Nov 2026), written from the code: `src/lib/types.ts`,
`src/lib/db.ts`, `src/lib/kv.ts` and `migrations/001_init.sql` in the code repository.

## Entities

| Entity | What it is | Kept in |
| --- | --- | --- |
| Clinic file | The one source of facts: clinic, hours, doctors, services, policies | The store; the file that ships with the app is the original |
| Message | One line of a chat: from the patient, the assistant or staff | The store |
| Ticket | A question that needs a person | The store |
| Chat id | A random id made by the patient's browser | The browser (localStorage) and on each message and ticket |

### Clinic file sections

| Section | Fields | Needed for a save |
| --- | --- | --- |
| `clinic` | `name`, `address`, `phone`, `emergency_phone`; optional `landmark`, `whatsapp`, `languages`, `parking`, `email`, `website` | the first four |
| `hours` | day to opening hours, as text | at least one day |
| `doctors` | `name`, `role`, `days` | at least one doctor with a name |
| `services` | `name`, `price`; optional `note`, `duration`, `aliases` (other names patients use) | at least one service with a name and a price |
| `policies` | name to text | present |
| `emergency_rules`, `bot_rules`, `note` | optional | no |

Other fields in the file are kept as they are. A save larger than 100,000 characters is refused.

### Message

| Field | Type | Notes |
| --- | --- | --- |
| `id` | text | key |
| `chat_id` | text | the patient's chat |
| `role` | text | `user`, `assistant` or `staff` |
| `text` | text | |
| `source` | text or empty | `clinic-file` (shown with the label), `clinic-file-unsure` (no label), `ai`, `guard`, `staff` |
| `created_at` | time | |

### Ticket

| Field | Type | Notes |
| --- | --- | --- |
| `id` | text | key |
| `chat_id` | text | never sent to the staff page |
| `status` | text | `urgent`, `needs human`, `answered` or `closed` |
| `priority` | text | `P1`, `P2` or `P3` |
| `text` | text | the patient's words; a booking adds the requested day, date and time |
| `reply` | text or empty | the staff reply |
| `created_at` | time | the age on the staff page is worked out from this |

## Stores

The first store that is set up is used:

1. Postgres (Neon), when `DATABASE_URL` is set.
2. Otherwise the Redis the host already provides, over its REST address (`KV_REST_API_URL`,
   `KV_REST_API_TOKEN`). Every key is under one prefix, so nothing of an earlier version is touched.
3. Otherwise server memory. On the host this is a fault, because each server instance has its own memory: the
   health route then answers not ok (status 503).

The live site runs on the Redis store today (the health route says store `kv`). Neon is not connected yet:
its marketplace terms must be accepted in the browser by the owner first. Postgres on the live site is not
tested.

| Store | Shape |
| --- | --- |
| Postgres | Tables `clinic_data` (one row, the file as JSON), `messages` (index on chat and time), `tickets` (index on time). See `migrations/001_init.sql`. |
| Redis | `<prefix>clinic`: the clinic file as JSON. `<prefix>tickets`: a hash, ticket id to ticket as JSON. `<prefix>msgs:<chat id>`: a list of messages as JSON, oldest first. |
| Server memory | The same three things in one server instance. Not shared, not kept. |

## Status and priority

| Case | Status | Priority |
| --- | --- | --- |
| Emergency | `urgent` | `P1` |
| Booking request | `needs human` | `P2` |
| Not in the clinic file, or the AI step says it is not found | `needs human` | `P3` |
| Staff reply sent | `answered` | unchanged |
| Staff close | `closed` | unchanged |

A fact answered from the clinic file opens no ticket. (In 1.x every message opened a ticket.)

Order on the staff page: urgent, needs human, answered, closed. Staff can also choose newest first. The
call-back list holds the waiting tickets.

## How a message is answered

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

## Edited clinic file and reset

A save from the staff page is checked (table above). A file that fails the check is refused with the name of
the field, and the old file stays. Reset writes the original file, the one that ships with the app, back into
the store. If the store cannot be read, the chat answers from the original file.

## Browser storage

The browser keeps three small values and no ticket or message:

| Key | Where | What |
| --- | --- | --- |
| `frontdesk.chatId` | localStorage | The id of this browser's chat. It is the key to that chat and is never in the ticket list. |
| `frontdesk.theme` | localStorage | Light or dark, once chosen. |
| `frontdesk.deskCode` | sessionStorage | The desk code a staff member typed, for this tab only, when the lock is on. |

## Routes

| Route | Method | What it does |
| --- | --- | --- |
| `/api/chat` | POST | `chatId`, `message`, optional `useAi`. Returns `reply`, `source`, `label`, `ticketId`, `ticketStatus`, `stored`. |
| `/api/messages` | GET | The messages of one chat, by `chatId`. |
| `/api/tickets` | GET | The ticket list without chat ids; whether a desk code is on; whether the store is shared. |
| `/api/tickets` | POST | A staff reply, a close, a clinic save or a clinic reset. |
| `/api/clinic` | GET | The clinic file. |
| `/api/health` | GET | `ok`, the store (`neon`, `kv` or `memory`), whether an AI key is set, the clinic name. |

The staff routes need the desk code in a request header only when `FRONTDESK_DESK_LOCK` is on and
`FRONTDESK_DESK_CODE` is set. The lock is off on the live site.

## In 1.x, not in 2.0

These data items were removed from this page because 2.0 does not hold them: the localStorage keys for
tickets, messages and settings (2.0 keeps only the chat id in the browser), the patient's name and phone, the
staff note, ready-made replies, feedback on a reply, ticket numbers, the per-browser private key, the
seven-day deletion of shared tickets, the assistant name and the clinic's own quick replies, and voice clips.

## Revisions

| Date | Change |
| --- | --- |
| 27 Nov 2026 | Rewritten for version 2.0.1 from the code. |

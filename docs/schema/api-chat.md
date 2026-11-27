# The API routes: requests and replies

Version 2.0.1. Five routes under `src/app/api/` of the code repository. All take and give JSON. Bodies are
checked with Zod. No route ever returns a key, a token or a connection address.

## POST /api/chat

Request:

| Field | Type | Rule |
| --- | --- | --- |
| `chatId` | string | 1 to 100 characters. Made in the patient's browser |
| `message` | string | 1 to 2,000 characters |
| `useAi` | boolean | Optional. `false` switches the AI step off for this message |

Reply, status 200:

| Field | Type | Meaning |
| --- | --- | --- |
| `reply` | string | The text for the patient |
| `source` | `guard`, `ai` or `clinic-file` | Which step answered |
| `label` | string or absent | "Answered from the clinic file" on a matcher reply that gives a fact |
| `ticketId` | string or absent | The ticket opened for this message, when one was recorded |
| `ticketStatus` | `urgent`, `needs human` or null | The status of that ticket |
| `stored` | boolean | `false` when the store could not be read or written; the reply is still given |

400 for a body that is not JSON or fails the check. A store failure is never an error here.

## GET /api/messages?chatId=...

200 `{ "messages": [ ... ] }`, oldest first. Each message: `id`, `chat_id`, `role` (`user`, `assistant`,
`staff`), `text`, `source`, `created_at`. 400 without a chat id (or one over 100 characters). 503 with an
empty list when the chat could not be read.

## GET /api/tickets

200 `{ "tickets": [ ... ], "deskCode": boolean, "shared": boolean }`. Each ticket is as in
[tickets.schema.json](tickets.schema.json): it never carries a chat id. `deskCode` says whether the desk
lock is on. `shared` is `false` when tickets are in one server's memory only. 401 when the lock is on and
the `x-desk-code` header is missing or wrong. 503 when the store could not be read.

## POST /api/tickets

One of four bodies:

| Body | What happens | Reply |
| --- | --- | --- |
| `{ "ticketId", "reply" }` | The reply (1 to 2,000 characters) is saved, the status becomes `answered`, a `staff` message is added to that chat | `{ "ticket" }` |
| `{ "ticketId", "close": true }` | The status becomes `closed` | `{ "ticket" }` |
| `{ "clinic": { ... } }` | The file is checked ([clinic-file.schema.json](clinic-file.schema.json)), then saved | `{ "ok": true, "clinic" }` |
| `{ "clinicReset": true }` | The clinic file that ships with the app is put back | `{ "ok": true, "clinic" }` |

400 for a bad body, or "Invalid clinic data: <where>". 401 as above. 404 when the ticket is not found. 503
when the store could not be written.

## GET /api/clinic

200 `{ "clinic": { ... } }`. When the store is down: the clinic file that ships with the app, with
`"stored": false`.

## GET /api/health

`{ "ok", "mode", "ai", "clinic", "error"? }`. `mode` is `neon`, `kv` or `memory`. `ai` says whether an AI
key is set. `clinic` is the clinic's name. Status 200 when ok, 503 when not: the store could not be
reached, or the store is server memory on the host.

## Settings (names only)

`DATABASE_URL`; `KV_REST_API_URL`, `KV_REST_API_TOKEN`; `AI_API_KEY`, `AI_BASE_URL`, `AI_MODEL`;
`OPENROUTER_API_KEY`; `AI_TIMEOUT_MS` (8000 when not set); `FRONTDESK_DESK_LOCK`, `FRONTDESK_DESK_CODE`.

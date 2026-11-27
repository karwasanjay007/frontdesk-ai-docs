# FrontDesk AI: tech stack

This page describes version 2.0.1 (live since 27 Nov 2026), written from the code repository's
`package.json`, `.env.example` and `src/`. Versions 1.0.0.0 to 1.3.0.3 were a static app (plain HTML, CSS
and JavaScript, a rules engine in the browser, one server function, localStorage). That code no longer runs.

| | |
| --- | --- |
| Expected users | the tester's automated browser (one run with the AI key and one without); about 100 patients a month for a real clinic |
| Last revised | 27 Nov 2026 |

## Choices

| Part | Choice | Why |
| --- | --- | --- |
| Framework | Next.js 15, App Router, React 19 | Pages and server routes in one project |
| Language | TypeScript 5 | The types of a ticket, a message and the clinic file are written once and checked |
| Styling | Tailwind CSS 3 | Light and dark from the same classes |
| Checks on input | Zod 3 | The chat body, a clinic save and the AI reply are checked before use |
| AI step | The `openai` client against an OpenAI-compatible address | The provider can be changed by environment variables; the step is optional |
| Store 1 | Postgres (Neon), `@neondatabase/serverless` | Used when `DATABASE_URL` is set |
| Store 2 | The Redis the host already provides, over its REST address | Used when no Postgres address is set; no client library, plain requests |
| Store 3 | Server memory | For local work only; on the host the health route is then not ok |
| Unit tests | Vitest 2 | 430 tests locally |
| Browser tests | Playwright | 13 tests; they run locally and against the live link |
| Question run | `scripts/run-tests.ts` with `tsx` | 30 questions, once as set up and once with the AI step off |
| Host | Vercel | Unchanged from 1.x |

## Stores, in order

1. Postgres (Neon), when `DATABASE_URL` is set.
2. Otherwise the Redis the host already provides, over its REST address (`KV_REST_API_URL`,
   `KV_REST_API_TOKEN`). Every key is under one prefix, so nothing of an earlier version is touched.
3. Otherwise server memory. On the host this is a fault, because each server instance has its own memory: the
   health route then answers not ok (status 503).

The live site runs on the Redis store today (the health route says store `kv`). Neon is not connected yet:
its marketplace terms must be accepted in the browser by the owner first. Postgres on the live site is not
tested.

## Environment variables (names only)

| Name | Use |
| --- | --- |
| `DATABASE_URL` | Postgres address. When set, Postgres is the store. |
| `KV_REST_API_URL`, `KV_REST_API_TOKEN` | The host's Redis. Used when `DATABASE_URL` is not set. |
| `AI_API_KEY`, `AI_BASE_URL`, `AI_MODEL` | The AI step. Optional. |
| `OPENROUTER_API_KEY` | Used for the AI step when `AI_API_KEY` is not set. The live site uses this one. |
| `AI_TIMEOUT_MS` | Time allowed for the one AI attempt. Default 8000. |
| `FRONTDESK_DESK_CODE`, `FRONTDESK_DESK_LOCK` | The desk code and its switch. The code is asked for only when the switch is on. Off on the live site. |

No value of any of these is written in a document, in the code or in a commit.

## AI step

One attempt per message, stopped at 8 seconds. The model is asked for JSON. Code checks the shape with Zod,
then rejects an answer with an unknown doctor, a figure for another service or a time for another day. A
rejected or failed answer costs nothing: the keyword matcher answers from the clinic file. Emergency,
medical, rule-break and booking messages never reach the model.

The model on the live site is the one set on the host; its name is not recorded in this repository. Which
model answers best was measured for 1.x only; it was not measured again for 2.0 (not tested).

## Monthly cost

Not measured for 2.0. The stores are on free plans and the AI step is one short call for some messages.

## Security

- The AI key is read on the server only. The health route says whether a key is set, never the key.
- The desk code is compared as digests of equal length and is never sent to a browser.
- The ticket list never carries a chat id; a chat is read only with its own id.
- A chat message is limited to 2000 characters; a clinic save is checked and limited in size.
- The staff page is open on the live site, because the tester opens it with no login. A clinic would switch
  the lock on.

## Diagram

See [ARCHITECTURE.md](ARCHITECTURE.md) and [docs/diagrams/](docs/diagrams/).

## Not used in 2.0

| Not used | Why |
| --- | --- |
| localStorage for tickets and messages | They are in the shared store; the browser keeps only the chat id |
| A rules engine in the browser | The checks and the matcher run on the server |
| Browser speech and a server voice | Voice was not rebuilt |

## Revisions

| Date | Change |
| --- | --- |
| 27 Nov 2026 | Rewritten for version 2.0.1 from the code. |

## Settings read by name

No value is written in the code or the documents. Besides the names above, the code reads `OPENROUTER_API_KEY` and `OPENROUTER_MODEL` (when the host holds an OpenRouter key and no `AI_API_KEY`, the AI step uses openrouter.ai), `FRONTDESK_DESK_CODE` with `FRONTDESK_DESK_LOCK`, `KV_REST_API_URL` with `KV_REST_API_TOKEN`, `DATABASE_URL`, `AI_TIMEOUT_MS`, and `VERCEL`, which the host sets itself and which tells the health route it is on the host.

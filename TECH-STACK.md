# FrontDesk AI: tech stack

This page describes version 1.7.2, the version on the live link.

The rule behind every choice: the cheapest ready-made part that meets [the requirements](PRD.md#requirements),
and nothing that can stop an answer. The answer is worked out in the browser from the clinic file. Every
service below adds to that answer or shares it, and the pages keep working when a service is missing. How the
parts fit together, with the diagrams, is in [the architecture](ARCHITECTURE.md).

## Choices

| Part | What is used | Why | What it costs |
| --- | --- | --- | --- |
| Pages | Plain HTML, CSS and JavaScript. No framework and no build step. The chat page is at `/`, the staff page at `/staff` | Two screens need nothing more, and the names the brief's tester reads stay under direct control | Nothing |
| Answer rules and the guard | Plain JavaScript in `core.js`, run in the browser. The server loads the same file to check the model's text | The answer is ready at once and is the same with and without the AI model | Nothing |
| Server | Seven small Node.js functions in `api/` (Node 20 or later), each a route | The only safe place for a key, and the only place rules about who may read a ticket can be enforced | Nothing on the host's free plan |
| Store | Postgres on Neon, reached through the one package the product depends on, `@neondatabase/serverless`. Plain SQL as tagged templates, so a value always travels as a parameter. Used when `DATABASE_URL` is set | Tickets, messages and the clinic file are shared by every device, in three small tables | Nothing on the free plan |
| Store, fall-back | When `DATABASE_URL` is not set and a Redis address and token are (`KV_REST_API_URL` or `UPSTASH_REDIS_REST_URL`, with `KV_REST_API_TOKEN` or `UPSTASH_REDIS_REST_TOKEN`), the same routes use Redis over its REST interface. With neither, the ticket and message routes answer 503, the clinic route returns the shipped file, and each page works from its own browser | The product can be hosted where only one kind of store is on offer, or none | Nothing on a free plan |
| AI model | OpenRouter, model `deepseek/deepseek-v4.1-flash` unless `OPENROUTER_MODEL` names another. Reasoning off, temperature 0, one attempt, 8 seconds in all. It rewords a drafted answer and is never sent the patient's message | Low cost and quick with reasoning off. The model can be changed without a code change | Pay per use |
| Voice in | The browser's own speech recognition first. If it does not answer, a short recorded clip goes to OpenRouter, model `google/gemini-2.5-flash-lite` unless `OPENROUTER_AUDIO_MODEL` names another | The browser's service is free but fails on some devices and networks | Nothing, or pay per use for the backup |
| Voice out | The browser's own voice first. Where it has none for the language, the reply's text goes to OpenRouter, model `openai/gpt-audio-mini` unless `OPENROUTER_TTS_MODEL` names another | Some browsers have no Hindi or Telugu voice | Nothing, or pay per use for the backup |
| In the browser | localStorage for this browser's own copy and its settings, sessionStorage for the desk code | The pages work when the store cannot be reached | Nothing |
| WhatsApp, Call, Directions | Plain links built from the clinic file: `wa.me` on a phone, `web.whatsapp.com` on a computer, `www.google.com/maps/search/` for Directions | No account and no key | Nothing |
| Fonts | Google Fonts, from `fonts.googleapis.com` and `fonts.gstatic.com`: Baloo 2, Noto Sans, Noto Sans Devanagari, Noto Sans Telugu, Fraunces, Inter | Hindi and Telugu replies render properly on every device | Nothing |
| Hosting | Vercel: static files plus the functions in `api/`, one project | Free, HTTPS, and no settings file is needed | Nothing on the free plan |
| Tests | Playwright scripts and plain Node.js scripts in `tests/` | They read the pages the way the brief's tester does | Nothing |

The tables, the browser keys and how long each thing is kept are in
[the data model](docs/data-model.md#where-data-is-kept). The files are in
[the architecture](ARCHITECTURE.md#the-files), and the hosting set-up in
[the architecture](ARCHITECTURE.md#hosting).

## Monthly cost

| Item | Cost a month |
| --- | --- |
| Hosting, free plan | $0 |
| Postgres on Neon, free plan | $0 |
| AI model through OpenRouter | pay per use, at the provider's list price for the model |
| Voice backups through OpenRouter | pay per use, only when a browser's own speech fails |
| Fonts, links, browser storage, tests | $0 |

Only an answer taken from the clinic file is sent for rewording. Emergency, medical, refusal, booking, not-sure
and ask-for-a-person replies never reach the model, so they cost nothing. With no key at all the product costs
nothing to run.

## Security in brief

- **Secrets.** `OPENROUTER_API_KEY`, `DATABASE_URL`, the Redis address and token, and `FRONTDESK_DESK_CODE`
  live only in the host's environment. None is in page code or in the repository. No route returns a key.
- **Who may read a ticket or a chat.** The browser that created it, with its private key, and staff who give
  the clinic's desk code. The server keeps a SHA-256 hash of the browser's key, never the key. The desk code is
  compared in constant time, and the list of every device's tickets is closed until a code has been set.
- **What a caller may change.** A stored ticket's patient, question and first reply never change. Only staff
  with the desk code can save the clinic file, and a file with a part missing is refused.
- **SQL.** Every query passes its values as parameters and has a 6 second limit.
- **Limits.** Each route has a size limit on what it accepts. The chat, ticket, message and clinic routes
  count requests per caller per minute. The two voice routes have no such count; see
  [known limits](REPORT.md#known-limits).
- **The model's text.** It is shown only after the guard passes it, on the server and again in the browser:
  no number, name or promise that is not in the draft or the clinic file.
- **Dependencies.** One package on the server. None in the pages.

What the product keeps about a patient, and for how long, is in [the README](README.md#privacy).

## Not used, and why

- **A framework or a build step.** Nothing it adds is needed for two pages.
- **An ORM.** Three small tables are read and written with a handful of plain SQL statements.
- **Named accounts and a sign-in page.** A patient should be able to ask at once, and the brief's check opens
  the staff page directly. One desk code guards the shared ticket list.
- **A model that answers on its own.** The run with the key removed would fail, and a model can invent a
  price. The rules answer first and the model only rewords.
- **Streaming the model's reply.** A reply filled in word by word could be read before it is complete.
- **A paid speech service as the first choice.** The browser's own speech is free and needs no key.
- **A service worker.** There is no offline cache; a chat that cannot reach the clinic is of no use.
- **A scheduler.** Nothing runs on a timer on the server. A ticket's age is worked out when the staff page is
  drawn.
- **The WhatsApp Business API.** A plain link needs no account and no approval.

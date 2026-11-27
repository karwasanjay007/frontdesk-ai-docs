# FrontDesk AI: tech stack

DRAFT for Sanjay's review, written from the decisions G1 to G30 (now PRD.md section 2.5 and stage 3) while `PRD.md` is
written in parallel. Cheapest ready-made option that meets the PRD. Defaults and accounts:
`sai-resources/business/stack-and-accounts.md`.

| | |
| --- | --- |
| Expected users | the grader's automated browser (two runs: with the key and without it); about 100 patients a month for a real clinic |
| Budget a month | $0 |
| Last revised | 20 Nov 2026 |

## Constraints from the PRD

- Graded by an automated browser that reads fixed ids, a class and data attributes (`#chat-input`,
  `#chat-send`, `.bot-msg`, `?q=`, `#ticket-queue`, `[data-ticket]`, `data-status`, `#staff-reply`,
  `#staff-send`).
- Must answer the same with the AI key removed (spec item 8, "the lower score counts"). So the answer is
  worked out without the model first (G3, G4), and the model may only rephrase it.
- Every price, doctor and timing comes from the clinic file currently loaded, never hard-coded (G18).
- `/staff` is a real path (G13). Tickets and staff replies cross between the two pages in one browser (G12).
- No login, dialog or splash in front of a graded element (G17).
- Works at 375 px, light and dark at 4.5:1 in every state (G23, G24).

## Choices

| Layer | Choice | Why | Cheaper or simpler alternative considered | Cost a month |
| --- | --- | --- | --- | --- |
| Front end | Plain HTML, CSS and JavaScript, no framework, no build step. Chat page at `/` (`index.html`); staff page at `/staff` (`staff/index.html`); shared scripts loaded by both | Smallest and fastest; every graded id and class is under direct control; a real `/staff` folder works on Vercel and on any local static server with no rewrite rules | React with Vite: a build step and more weight, no benefit for two screens | $0 |
| Rules engine | Plain JavaScript in the browser. Reads the clinic file and answers in the fixed order of G4: emergency, ignore-your-rules or discount, symptoms, booking, answerable from the file, otherwise not sure | The file answer is ready at once and is identical with and without the key, so the no-key run scores the same as the keyed run | Model-only answers: the no-key run would fail every question | $0 |
| Back end | One Vercel serverless function `api/chat` (Node.js). It receives the drafted answer, the facts the rules engine found, the language and the current clinic data, never the patient's words; it calls the model with the key from the Vercel environment; it returns only the model's text. A GET returns `{ ok: true }` so the chat page can wake it on load. Time limit 5 s on the model call, so the browser's 6 s limit (G3) is never the one that trips | One small function is the only place a key can live safely (spec: "never in page code") | A model call from the browser: exposes the key. A separate server: more to run and pay for | $0 (Vercel Hobby) |
| Voice backup functions | Two more Vercel serverless functions. `api/tts.js` reads a reply aloud when the browser has no voice for the language (text of the reply only, 700 characters at most, 15 s limit). `api/transcribe.js` turns a short recorded clip into text when the browser's own speech recognition refuses (12 s limit); the text then goes through the same rules as typed text. Both call audio models through OpenRouter with the same key; model names in `OPENROUTER_TTS_MODEL` and `OPENROUTER_AUDIO_MODEL` | The browser's speech services are free but fail for some languages and networks | Browser speech only: no Telugu voice in Chrome on Windows | Pay per use, a few cents a month at this size |
| Ticket store function (since 1.3.0.0) | A fourth Vercel serverless function, `api/tickets.js` (Node.js). It holds the rules for shared tickets (the patient's browser key, the clinic's desk code, cleaning, merging, a rate limit), kept apart from the store. It answers 503 when no store is configured, and both pages then behave as before | Staff on another device can see and answer a ticket; the rules run on the server, where a browser cannot change them | Reading the store from the browser: the store token would reach the page | $0 (Vercel Hobby) |
| Ticket store (since 1.3.0.0) | Upstash Redis, added through the Vercel Marketplace on the free plan with automatic upgrade off. Reached over its REST API; no package is installed. One record per ticket for seven days, an ordered list of ticket ids, and a small set of ids per browser key. Environment variables `KV_REST_API_URL` and `KV_REST_API_TOKEN` (added by the store) and `FRONTDESK_DESK_CODE` (set by the clinic); names only, the values live in the Vercel environment | No package and no build step; the free plan costs nothing; a ticket record expires after seven days | A SQL database: tables, a client package and row-level security for one small list | $0 (free plan) |
| Shared store (since 1.4.0.0) | `api/_store.js`, the one module every route reads and writes through. Postgres on Neon, added through the Vercel Marketplace, reached with `@neondatabase/serverless` when `DATABASE_URL` is set; plain SQL as tagged templates, so a value always travels as a parameter. Tables `messages`, `tickets` and `clinic_data`, created if missing; each query has a 6 second limit. With no `DATABASE_URL` it falls back to the host's Redis (the 1.3 ticket store, same keys); with neither, the routes answer as before. Routes `api/clinic.js`, `api/messages.js`, `api/health.js` | Tickets, messages and the clinic file shared by every device, in one place with plain SQL | An ORM: one more package for three small tables | $0 (free plan) |
| Browser add-ons (since 1.1.0.0) | `staff/tools.js` (search, filters, sort, alerts, queue figures, CSV export, ready-made replies, private note), `staff/form.js` (the form for the clinic file) and `sync.js` (mirrors tickets to the store and fetches staff replies; loaded on `/` and `/staff`). Plain JavaScript, each in its own file | Additions sit beside the baseline scripts, so the ids and statuses the tester reads stay as they were | Editing `staff.js` and `chat.js` in place: every earlier test would need a new look | $0 |
| AI model | DeepSeek V4.1 Flash through OpenRouter, model `deepseek/deepseek-v4.1-flash` (env `OPENROUTER_API_KEY`, optional `OPENROUTER_MODEL`), reasoning off, temperature 0.2, at most 700 output tokens. Chosen by Sanjay on 20 Nov 2026. See "AI model: evidence" below | Low cost, and quick with reasoning off, which suits the 5 s limit; `OPENROUTER_MODEL` can switch the model without a code change | Google Gemini `gemini-3.5-flash-lite` free tier: the first draft's recommendation, kept as the considered alternative. Z.ai `GLM-4.7-Flash`: free but limits not published | Cents (see "Monthly cost") |
| AI output guard | Plain JavaScript in the browser (G7, `guard()` in `core.js`). A model reply is shown only if every number and every capitalised name in it is in the draft or the clinic file, it names no doctor outside the file, makes no promise or topic (ticket, call back, discount, offer, free, %, branch, refund) the draft did not, names no medicine, still contains the facts the rules engine found, and is in the patient's language | The model can never put an invented price or a medicine in front of a patient | Trusting the model's instructions alone: one bad reply fails spec items 2, 4 or 6 | $0 |
| Storage | Browser localStorage on the site's own origin holds the working copy: tickets, staff replies and the edited clinic file. Since 1.4.0.0 the shared copy is in the store (see "Shared store"). The chat page listens for the `storage` event, so a staff reply sent on `/staff` shows in the chat within a second (G12) | The tester uses one browser; nothing extra to sign up for or fail | A database such as Supabase: not used. Since version 1.4.0.0 tickets, messages and the clinic file are in Postgres on Neon (see the "Shared store" row) | $0 |
| Clinic file | `clinic-data.json` shipped with the site as the default. Staff can paste or edit it on `/staff`; the edit is stored in localStorage and used at once by both pages (G18) | Answers follow whatever data is loaded, which is what the hidden-data test needs | Hard-coded answers: work only on the sample file | $0 |
| Speech | The browser's own Web Speech API: speech recognition for the microphone button and speech synthesis for the speaker button, in en-IN, hi-IN and te-IN; hidden where the browser lacks it (G30) | No key and no cost; typing always works without it | A cloud speech service: a second key, a cost and another call that can fail | $0 |
| WhatsApp link | A link built in the browser (`waUrl()` in `core.js`) from the clinic file's `whatsapp` number with the question written in (G28): `wa.me` on a phone, and `web.whatsapp.com` (WhatsApp Web) on a computer, where `wa.me` stops at a landing page. The staff card's WhatsApp link for a call-back number is built the same way | No account or key | The WhatsApp Business API: an account, approval and cost | $0 |
| Fonts | Google Fonts, loaded by `index.html` from `fonts.googleapis.com` (the style sheet) and `fonts.gstatic.com` (the font files): Baloo 2 for headings and Noto Sans, Noto Sans Devanagari and Noto Sans Telugu for text, as `brand/BRAND.md` sets | Hindi and Telugu replies render properly (G19) | System fonts only: Telugu can fall back badly | $0 |
| Automations | None | Nothing runs on a schedule; ticket ageing is computed when `/staff` is drawn | n8n: nothing for it to do in this build | $0 |
| Hosting | Vercel, project `frontdesk-ai`, on its own `*.vercel.app` address. Never sai4b.com. Documents kept off the site (`.vercelignore`) | Free, HTTPS, static files and four functions (`api/chat`, `api/tts`, `api/transcribe`, `api/tickets`) in one project | Netlify: same cost, but the account and habits are on Vercel | $0 |
| Offline | Not needed beyond the app shell. The page loads once and the rules engine needs no network; nothing in the PRD asks for an offline install, so no service worker is planned | The no-key path already works without the model; a patient chat with no network cannot reach the clinic anyway | A service worker that caches the app's files: adds a stale-build trap during the build for no graded gain | $0 |
| Tests | Playwright scripts in the code repository's `tests/`, run against the local site and the live link: `tests/brief.mjs` (the literal spec test), the eight sample questions with and without the key, a contrast scan and the widths | Mirrors how the grader checks | Clicking by hand: misses the no-key run and the exact words | $0 |
| Code repository | `frontdesk-ai`, private on GitHub (karwasanjay007) | History and rollback; private while the competition is open | | $0 |
| Documents repository | `frontdesk-ai-docs`, private until shared; one baseline commit when made public | Documents stay out of the code and off the site | | $0 |

**Sign-in.** None (G17). No login on the chat page or on `/staff`. A sign-in screen would stand between
the grader and a graded element. Server storage for tickets was added in version 1.3.0.0; the list of every
device's tickets needs the clinic's desk code. Named staff sign-in is still phase 2 (Q-3), not built now.

**Total a month:** cents for the competition; the cost table below gives the figures.

## AI model: evidence

Checked on the web on 20 Nov 2026 (official pages unless marked). Sanjay chose OpenRouter with DeepSeek V4.1
Flash the same day, in place of the Gemini recommendation of the first draft.

| Provider | What the provider publishes today | Verdict |
| --- | --- | --- |
| OpenRouter, DeepSeek V4.1 Flash | The models list (openrouter.ai/api/v1/models, read 20 Nov 2026) gives `deepseek/deepseek-v4.1-flash` at $0.30 per million input tokens and $1.20 per million output tokens, with a 1,048,576 token context. The same list gives the dated `deepseek/deepseek-v4.1-flash-1017` at $0.15 and $0.60. No free variant is listed | Chosen (Sanjay, 20 Nov 2026). Paid from the first call; reasoning off in every request |
| Google Gemini API | The pricing page (aistudio.google.com/docs/pricing) lists "Free of charge" on the free tier for every Flash and Flash-Lite text model: `gemini-3.8-flash`, `gemini-3.6-flash`, `gemini-3.5-flash-lite`, `gemini-3.1-flash-lite`, `gemini-3-flash-preview`, `gemini-2.5-flash`, `gemini-2.5-flash-lite`. Free-tier content is used to improve Google's products. `gemini-2.5-flash` is deprecated: from 1 Dec 2026 its requests go to `gemini-3.5-flash-lite`. The rate-limits page (aistudio.google.com/docs/rate-limits) no longer prints per-model free numbers; it says to read the current limits in AI Studio, and says a request over a limit gets a rate-limit error | Considered alternative: `gemini-3.5-flash-lite` on the free tier, the first draft's recommendation; not chosen |
| DeepSeek | The pricing page (api-docs.deepseek.com/quick_start/pricing) publishes no free tier and no free credit amount; charges come from a "topped-up balance or granted balance". `deepseek-flash` (DeepSeek-V4.1-Flash-1017): $0.15 input (cache miss) and $0.60 output per million tokens off-peak, double that at peak | Not used directly; the build reaches DeepSeek V4.1 Flash through OpenRouter |
| Z.ai (GLM) | The pricing page (docs.z.ai/guides/overview/pricing) lists `GLM-4.7-Flash` and `GLM-4.5-Flash` as free for input and output. It states no rate or quota limits. Third-party sites (not official) report about one request at a time | Not chosen: free, but no published limit to plan against |

**At the limit.** When the model is over its free limit, slow, failing or the key is missing, the
function returns an error or times out. The browser then shows the rules engine's file answer with the
label "Answered from the clinic file" (G15). Every reply the model writes also passes the guard (G7);
a reply that fails the guard is dropped the same way. Emergency, advice, refusal, not-sure and booking
replies never go to the model at all (G4). So hitting the limit lowers polish, never correctness.

**Privacy note.** The model receives only the drafted answer, its facts, the language and the clinic file,
never the patient's words. The clinic is fictional and no patient data is real during the competition.

## Monthly cost

| Item | Competition (grader, two runs of 28 questions) | Real clinic, about 100 patients and 500 messages a month |
| --- | --- | --- |
| Vercel Hobby: static site and one function | $0 | $0 (well inside the Hobby allowance; a commercial clinic would need Vercel Pro, $20) |
| OpenRouter `deepseek/deepseek-v4.1-flash` | Cents: two runs of 28 questions, and only answerable questions call the model | About 500 calls of roughly 2,000 input and 150 output tokens at $0.30 and $1.20 per million: about $0.39 |
| localStorage, Web Speech API, `wa.me` links | $0 | $0 |
| Upstash Redis through the Vercel Marketplace, free plan, automatic upgrade off (since 1.3.0.0) | $0 | $0 on the free plan; automatic upgrade is off |
| GitHub private repositories | $0 | $0 |
| **Total** | **Cents** | **About $0.39 for the model; $0 for the rest** |

The real-clinic token sizes are an estimate from the size of the clinic file, to be measured in step 9.

## Security

- Secrets: the model key lives only in the Vercel project's environment (`OPENROUTER_API_KEY`, a sensitive
  Production variable, hidden in the dashboard). It is to be rotated after the competition. It is never in page code, the repository or a
  document.
- The function never returns the key, never echoes the request headers, and returns only `{ text }` or
  `{ error }` with a short code. Its error messages never include the provider's raw response.
- Inputs are size-limited in the function: the body to 60,000 characters, the draft to 1,500 characters,
  the facts to 20 entries and the clinic data to 12,000 characters; a bad body gets a 400 and the browser
  uses the file answer.
- The model's output passes the guard (G7) before display, and is shown as text, never as HTML.
- Every text from the clinic file, a staff reply or a patient message is escaped before display.
- Access control: `/staff` is open by design for the grader (G17) and shows only the tickets of its own
  browser. Since version 1.3.0.0 the list of every device's tickets needs the clinic's desk code, checked on
  the server. Named staff sign-in is phase 2.
- Database: since version 1.3.0.0, Upstash Redis as the ticket store, reached only by `api/tickets.js`. It is
  not a SQL database, so row-level security does not apply; the function's own rules decide who reads what.
- Ticket store secrets (since 1.3.0.0): `KV_REST_API_URL`, `KV_REST_API_TOKEN` and `FRONTDESK_DESK_CODE` live
  only in the Vercel project's environment. `api/tickets.js` also accepts `UPSTASH_REDIS_REST_URL` and
  `UPSTASH_REDIS_REST_TOKEN` as other names for the store's address and token, used when the `KV_` pair is not set. The desk code is typed by staff and kept in the browser for the
  session only. The server keeps a hash of each patient's browser key, never the key itself.
- Dependencies: none in the page. Playwright is a development dependency only. An audit by a different
  model runs before release (R1).

## Diagram

`docs/diagrams/c4-context.svg`, `c4-container.svg`, `c4-component.svg` and `deployment.svg`, each from its
PlantUML source (SVG only), embedded in ARCHITECTURE.md.

## Not used, and why

- A framework or build step: nothing it adds is needed for two pages, and it hides the graded ids behind
  generated markup.
- A SQL database and named staff login: the tester uses one browser. A small ticket store was added in
  version 1.3.0.0 (see "Choices"); named staff login is phase 2 for a real clinic (Q-3).
- Streaming the model's reply: a `.bot-msg` filled word by word could be read before it is complete (G2).
- A cloud speech service: the browser's own speech is free and needs no key.
- A service worker: no offline install is asked for.
- Gemini and Z.ai as the main model: see "AI model: evidence".

## Revisions

| Date | Change |
| --- | --- |
| 20 Nov 2026 | First draft, from decisions G1 to G30; model limits checked on the web the same day |
| 20 Nov 2026 | Model changed to DeepSeek V4.1 Flash through OpenRouter (Sanjay's choice), reasoning off; price read from the OpenRouter models list the same day; Gemini kept as the considered alternative; back end, guard, cost, security and diagram rows brought in line with the code |
| 22 Nov 2026 | Brought up to version 1.3.0.0: the ticket store (Upstash Redis through the Vercel Marketplace, REST, no package), the function `api/tickets.js`, the three environment variable names, the browser add-ons `sync.js`, `staff/tools.js` and `staff/form.js`; storage, sign-in, security and "Not used" lines corrected |

## Version 1.5.0.0 (27 Nov 2026)

- Store: Postgres on Neon through `@neondatabase/serverless` (see "Shared store"); environment variable
  `DATABASE_URL` (name only, the value lives in the host's environment). The Redis names stay as the fallback.
- Model step: temperature 0, one attempt, 8 seconds in all (the browser waits 9 seconds), the guard of `core.js`
  run on the server, at most `FRONTDESK_CHAT_PER_MINUTE` requests a minute (60 when not set).
- Styles and add-ons: `brand.css`, `layout.css` with `layout.js`, `compact.css` with `compact.js`, `shared.js`;
  plain CSS and JavaScript, each in its own file beside the baseline.
- Versions 1.5.0.1 to 1.6.0.1: no new technology, package or environment name; the phone chat polish is in
  `compact.css` and `compact.js`, the doctor rule in `core.js`.
- Version 1.7.0.0: no new technology, package or environment name; the desktop chat polish is in `compact.css`
  and `compact.js` (768 px and up).

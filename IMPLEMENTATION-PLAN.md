# FrontDesk AI: implementation plan

This page describes version 1.7.2, the version on the live link.

How to run the product, test it and release it, and the order it was built in. The parts are described in
[the architecture](ARCHITECTURE.md) and the reasons for each in [the tech stack](TECH-STACK.md#choices).

## Run it

The product is a static site with server functions in `api/`. There is no build step.

1. Install Node.js 20 or later, then run `npm install` in the code folder. It installs one package.
2. Start the site on port 4173, the address the tests expect:
   - pages only: any static file server pointed at the code folder;
   - pages and the routes in `api/`: `vercel dev --listen 4173`.
3. Open `http://localhost:4173/` for the patient chat and `http://localhost:4173/staff` for the staff page.

Nothing needs to be set for the chat to answer. Each environment variable switches one thing on:

| Variable | What it switches on | Without it |
| --- | --- | --- |
| `DATABASE_URL` | The shared store in Postgres | The Redis variables are tried next |
| `KV_REST_API_URL` and `KV_REST_API_TOKEN`, or `UPSTASH_REDIS_REST_URL` and `UPSTASH_REDIS_REST_TOKEN` | The shared store in Redis when `DATABASE_URL` is not set, and a request count shared by every server instance | With no store at all, tickets and the clinic file stay in this browser, and the chat and the staff page still work |
| `FRONTDESK_DESK_CODE` | The staff list of tickets from every device, and saving the clinic file for every device | The staff page shows this browser's tickets only |
| `OPENROUTER_API_KEY` | The AI model's rewording of an answer, and the two voice backups | Every answer is the rules' own text; voice uses the browser's own speech only |
| `OPENROUTER_MODEL`, `OPENROUTER_AUDIO_MODEL`, `OPENROUTER_TTS_MODEL` | Another model for rewording, for voice in and for voice out | The models named in [the tech stack](TECH-STACK.md#choices) |
| `FRONTDESK_CHAT_PER_MINUTE` | Another limit on rewording requests per caller per minute | 60 |

`GET /api/health` says which store is in use and whether a model key is present. What each store holds is in
[the data model](docs/data-model.md#where-data-is-kept), and each file's job in
[the architecture](ARCHITECTURE.md#the-files).

## Test it

The tests are in `tests/` in the code folder. There is no test runner and no test script in `package.json`:
each file is run on its own with `node`, prints one `PASS` or `FAIL` line per check and ends with a verdict
line. Which test proves which acceptance criterion is in [the epics](docs/epics.md).

**Engine and server tests** (`tests/*.cjs`) need no browser and no running site:

```
node tests/engine.cjs
node tests/store.cjs
```

**Browser tests** (`tests/*.mjs`) use Playwright, which must be installed. Each takes the address of the site
as its first argument, and uses `http://localhost:4173/` when none is given:

```
node tests/brief.mjs
node tests/brief.mjs https://<the live link>/
```

Two flags matter for the tests of the specification:

| Flag | What it does |
| --- | --- |
| `--block-api` | Stops every request to `/api/`, so the AI call fails even when a key is set. The run then shows the product answering from the clinic file alone |
| `--nokey` | Says the site was started with its AI key removed, so the no-key requirement is checked and not only reported |

`tests/desk-chat.mjs` also saves pictures of the page when `FD_SHOTS` is set.

**Tests that only make sense on the live link.** `tests/sync-live.mjs`, `tests/sync-wait-live.mjs`,
`tests/sync-watch-live.mjs` and `tests/tickets-api-live.mjs` use the real store; the wait and watch tests
need a person to reply on the staff page while they run. They leave test tickets in the store, which are
removed after the keeping time in [the data model](docs/data-model.md#where-data-is-kept).

What to run when:

- After a change: the test of the part that changed, then `tests/brief.mjs` and `tests/questions.mjs`, each
  with and without `--block-api`.
- Before a release: every file in `tests/` against the local site.
- After a release: the browser tests again against the live link. The output of each run is saved as
  `tests/last-<name>-live.txt`.

The saved results are summarised in [the report](REPORT.md#latest-results-on-the-live-link), with what was
[not tested](REPORT.md#not-tested).

## Release it

1. Write the new version number on the first line of the `VERSION` file.
2. Run every engine test and every browser test against the local site. Release only when each ends in a pass.
3. Compare the build with what is live, feature by feature, using the table of what the product supports in
   [the README](README.md). A build that lacks anything the live link has is not released.
4. Deploy with `vercel deploy --prod`. Tests, documents and the specification are kept out of the deploy by
   `.vercelignore`.
5. Call `GET /api/health` on the live link. It must answer `ok: true`, with the store named `postgres`.
6. Open `/` and `/staff` on the live link, each loaded afresh, on a phone width and a desktop width, in light
   and dark.
7. Run the browser tests against the live link, the tests of the specification both with and without `--block-api`,
   and save the results.
8. Bring the documents in line with the code, add one entry to [the changelog](CHANGELOG.md), and write the
   results into [the report](REPORT.md#latest-results-on-the-live-link).

Rules that hold for every release:

- A new feature goes in its own file beside the existing ones, with a small hook, so the replies and the ids
  kept stable for automated tests stay as they are.
- A new feature gets a new test file. An earlier test is not edited to make a change pass.
- Nothing that is live is removed, hidden or replaced by a smaller version.
- No dialog, login or splash is put in front of the chat or the staff page.

## The order it was built in

1. The specification captured word for word, and a test for each of its "Done when" lines, failing because nothing
   was built.
2. The two pages, the clinic file loader and the rules that answer from the file.
3. The safety rules in a fixed order: emergency, rule change or discount, medical, booking, answer, not sure.
4. Tickets, the staff page and a staff reply that reaches the patient's chat.
5. The question asked from the page address, and the label on answers that come from the file.
6. The AI model's rewording step, with the guard that checks its text.
7. The optional features: Hindi and Telugu, booking tickets, ageing and priority, the clinic file editor, quick
   replies.
8. Going further: after hours, the call-back list, WhatsApp, another kind of business, voice in and out.
9. The phone layout, light and dark, and contrast.
10. Asking for a person, the assistant's name and the clinic's own quick replies.
11. Staff tools: search, filters, sort, alerts, figures, CSV export, ready-made replies, the private note, and
    the form for the clinic file.
12. Ticket numbers, the chat thread in a ticket, feedback on a reply.
13. Tickets shared across devices, with a private key per browser and the desk code.
14. The shared store in Postgres for tickets, messages and the clinic file.
15. The brand's look, the staff link in the top bar and the clinic information pinned on the right.
16. The phone chat reworked: starter cards, a slim composer and the "+" menu. Then the desktop chat to match.
17. Answer fixes found by review: an unknown doctor, the phone number when asked, the call-back line when no
    number is known.

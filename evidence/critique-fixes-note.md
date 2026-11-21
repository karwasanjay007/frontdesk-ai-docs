# Critique fixes: evidence note (21 Nov 2026)

This note cites results from the code pull request
https://github.com/karwasanjay007/frontdesk-ai/pull/1
(branch `cursor/hackathon-critique-fixes-4a98`). It does not invent new live runs.

## What the code PR changed (relevant to docs)

- **"Answered from the clinic file" label:** shown only when a real file answer's model path failed or
  was skipped (`modelOk` and `kind === 'answer'`). Fixed emergency, medical, refuse and not-sure
  templates stay unlabelled.
- **Fixtures:** 18 paraphrased English + Hindi/Telugu cases in `tests/engine.cjs` and `tests/audit.mjs`.
- **Intent:** aligners brand questions return not sure.
- **UI:** full-bleed ChatGPT-style patient chat; clinic info in a drawer; tester hooks unchanged.
- **Mobile:** `scrollWidth` 375 reported in the PR.

## Tests reported on that PR (verbatim from the PR description)

```
$ node tests/engine.cjs
ALL PASS

$ node tests/brief.mjs http://127.0.0.1:4199/ --nokey
ALL PASS (31 checks)
```

These are local runs against the PR branch, not a re-run saved into `last-*.txt` in this docs repo.
The existing `evidence/last-*.txt` files remain the last saved live-link results from before this PR.

## Spec page recapture

Attempted against https://dashboard.freedomwithai.com/redesign/v3/projects on 21 Nov 2026: the page
requires sign-in ("Sign in to start your hackathon"). No FrontDesk AI "Done when" text was visible
without auth. Status in SPEC-CHECK.md: **UNVERIFIED / To do**.

## Out of scope / still UNVERIFIED

Server tickets, staff auth, multi-tenant, PII encryption, rate limits, speech audio quality, and a full
live retest after the Vercel deploy of the UI/label changes.

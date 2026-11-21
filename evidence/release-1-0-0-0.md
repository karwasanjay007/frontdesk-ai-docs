# FrontDesk AI Version 1.0.0.0 — release evidence

**Owner:** Sanjay Karwa, Smart AI for Business  
**Date:** 21 November 2026  
**Patient chat:** https://frontdesk-ai-xi-gray.vercel.app  
**Staff:** https://frontdesk-ai-xi-gray.vercel.app/staff  
**Code baseline:** private repository `frontdesk-ai`, tag `v1.0.0.0`  
**Docs baseline:** this repository, tag `v1.0.0.0`

## What this baseline includes

- Rules-first clinic answers with an honest hand-off to staff tickets.
- "Answered from the clinic file" only on real file answers when the model path failed or was skipped.
- Patient chat UI with full-bleed canvas, typing indicator, light and dark themes, and 375 px layout.
- Docs that warn tickets are same-browser localStorage only.

## Checks cited for this baseline

```
node tests/engine.cjs → ALL PASS
node tests/brief.mjs http://127.0.0.1:4199/ --nokey → ALL PASS (31 checks)
```

Live evidence files under `evidence/last-*-live.txt` remain the submission record from 21 Nov 2026.
Projects-page re-capture under a signed-in session remains To do.

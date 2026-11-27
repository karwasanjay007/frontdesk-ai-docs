# FrontDesk AI: evidence index

This folder holds test results. Only five files are results of version 2.0.1, the version that is live. Every
other file is a result of versions 1.0.0.0 to 1.3.0.3, kept as the record. That code no longer runs, so those
files say nothing about the live product.

## Results of 2.0.1 (live link, 27 Nov 2026)

| File | What it is | Result |
| --- | --- | --- |
| `v2-0-1-questions-30-live.txt` | 30 questions, once with the AI key and once with the AI step off | 30 of 30 in both runs; lower 30 of 30 |
| `v2-0-1-e2e-live.txt` | The browser tests | 12 of 13; the miss assumes an empty queue |
| `v2-0-1-contrast-live.txt` | Contrast, light and dark, chat and staff | All 6 checks pass |
| `v2-0-1-brief-live.txt` | The brief checker written for 1.x, run on 2.0.1 | 27 of 29 |
| `v2-0-1-bonus-live.txt` | The bonus checker written for 1.x, run on 2.0.1 | 14 of 15 |

The misses are explained in `SPEC-CHECK.md` and `REPORT.md`. The local run of the brief checker (28 of 29) has
no file here.

## Results of 1.x, kept as the record

| Files | Family |
| --- | --- |
| `last-brief-*.txt` | The nine must-do items and the "How we check" lines, with the key and with the model blocked |
| `last-questions-*.txt` | The eight sample questions |
| `last-bonus-*.txt` | The six bonus items |
| `last-audit-*.txt` | Further questions in the tester's style |
| `last-engine.txt`, `last-sweep.txt`, `last-mixed.txt`, `last-other-clinic.txt`, `last-person-engine.txt` | The 1.x rules engine, without a browser |
| `last-emergency-live.txt`, `last-wording-*.txt` | Emergency, timing and price wording |
| `last-booking-*.txt` | Booking requests |
| `last-i18n-*.txt` | The language menu |
| `last-voice-live.txt` | Voice input |
| `last-widths-live.txt`, `last-controls-live.txt` | Page widths, tap targets, the theme button and the skip link |
| `last-contrast-*.txt` | Contrast, three ways of measuring |
| `last-person-live.txt`, `last-callback-live.txt` | Asking for a person, and call-back details |
| `last-order-live.txt` | A further 1.x live check; its family was not looked up for this index |
| `last-form-live.txt`, `last-tools-live.txt`, `last-tools2-live.txt`, `last-extras3-live.txt`, `last-staffclear-live.txt` | Staff page additions of 1.1 to 1.3 |
| `last-sync-*.txt`, `last-tickets-*.txt` | Tickets shared across devices in 1.3 |
| `release-1-0-0-0.md` | The release note of the first baseline |

# FrontDesk AI: visual critique

Reviewed 20 November 2026 against the live site (https://frontdesk-ai-xi-gray.vercel.app and /staff) with Playwright at 375x740, 768x1024, 1440x900 and 1920x1080, light and dark, after five questions (Saturday timings, root canal price, "I have severe swelling and bleeding", a Tuesday 5 pm booking with Dr Rao, hair transplants) and with the resulting tickets on /staff. Screenshots were looked at in a temporary folder and deleted. No code was changed.

## What the graded checks pin down

These hold today and every change below is written to keep them:

- No sideways scroll: `scrollWidth` equalled the viewport at all four widths, chat and staff, both themes.
- Composer in view: the form's bottom edge sat at 732 of 740, 991 of 1024, 861 of 900 and 1041 of 1080.
- `.bot-msg` is the reply. Its `textContent` already includes "Answered from the clinic file" (the `<small class="src-label">` sits inside it). Nothing new may go inside `.bot-msg`, and nothing new may be given the class `bot-msg` or be added after the last reply as a `.bot-msg`. Extras go in `.msg-tools` or on the `.msg-row`, which is what the code already does. Pseudo-elements (`::before`, `::after`) do not appear in `textContent` or `innerText`, so icons drawn in CSS are safe.
- The staff reply box and send button (`#staff-reply`, `#staff-send`) must stay once, inside the open ticket's `[data-ticket]`.

## Overall verdict

It already looks like a calm, credible product: one teal brand colour with a gold rule, sensible type (Baloo 2 for the brand, Noto Sans for text, so Hindi and Telugu render properly), 44 px touch targets everywhere, real dark tokens rather than an inverted filter, and a side panel that shows real clinic facts. What holds it back from feeling like a real clinic product is mostly proportion and emphasis: on wide screens the conversation is stretched across 1,100 to 1,500 px; on a phone the clinic's name and emergency number disappear; the two action buttons under every reply are heavier than the reply itself; and the safety-critical states (emergency, needs a call back) look almost the same as an ordinary answer. The staff page is a tidy form, not yet a helpdesk.

Reference points used: Intercom Messenger and WhatsApp Business (bubble rhythm, quick replies, quiet message actions), Practo and Zocdoc (clinic identity, open status, call and directions shortcuts), Front and Zendesk (ticket queues and states).

## Priority 1: fix before judging

### 1. Wide screens: put the conversation in a centred reading column

Seen: at 1440 the chat column is 1080 px and at 1920 it is 1480 px. Bot bubbles sit at the far left (max 720 px) and user bubbles at the far right, so the eye jumps 700 to 1,000 px between question and answer. The input runs 1,280 px wide with a short placeholder at its left end. This is the single biggest "unfinished" signal on a judge's desktop.

Change (styles.css, inside the existing `@media (min-width: 1100px)` block):

```css
.chat-log, .composer { --col: 820px; padding-inline: max(32px, calc((100% - var(--col)) / 2)); }
```

Both the messages and the composer then share one 820 px column centred in the chat area; the background still fills the space, so nothing looks empty.

Graded risk: none. Widths only shrink, the composer stays in normal flow at the bottom, `.bot-msg` is untouched.

### 2. Phone: keep the clinic's name and the emergency number on screen

Seen at 375: `.brand-tag` (the clinic name), `.open-chip` and the `.fine` line (which holds the emergency number) are all hidden by the `max-width: 480px` block. A patient on a phone, which is most patients, sees "FrontDesk AI" but never which clinic it is, whether it is open, or the emergency number. Real clinic chats (Practo, WhatsApp Business) lead with the business name and an open status.

Change:

```css
@media (max-width: 480px) {
  .brand-app { display: none; }                         /* the product name matters less than the clinic */
  .brand-tag { display: block; font: 700 1.05rem/1.1 var(--head); color: var(--on-brand); }
  .fine { display: block; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; margin-top: 4px; }
}
```

and in chat.js `labels()`, on narrow screens set the fine line to the short form, for example `Emergency: 98000 00123` (keep the `#emg-link` anchor as it is). If you want the open state too, show the open chip as a 10 px dot before the clinic name with `::before` on `.brand-tag` driven by a class, rather than as text.

Graded risk: low. The fine line adds about 18 px to the composer; the layout is a flex column with `100dvh`, so the log shrinks and the send button stays in view. Re-run `tests/widths.mjs` because the header must still fit in 375 px: removing "FrontDesk AI" and showing "Bright Roots Dental" at 1.05rem is about the same width.

### 3. Make the emergency reply look like an emergency

Seen: `kind-emergency` only swaps the bubble border to 2 px red. Everything else (white bubble, grey label, Listen button) is identical to a price answer. Separately, and worth raising with whoever owns the rules: "I have severe swelling and bleeding" was treated as a medical question, not an emergency (the pattern in core.js needs swelling near face, neck, cheek or jaw, or bleeding with "heavy" or "not stop"), so the patient got a booking offer and the ticket was "Needs human", not "Urgent". That is a content decision, not visual, but a judge may well type exactly that.

Change (styles.css):

```css
.kind-emergency .bot-msg { background: var(--urgent-bg); color: var(--ink); border: 2px solid var(--urgent); }
.kind-emergency .bot-msg .src-label { color: var(--urgent); }
.msg-row.kind-emergency::before { content: "Emergency"; align-self: flex-start; margin-bottom: 4px; font: 700 .75rem var(--body); text-transform: uppercase; letter-spacing: .04em; color: var(--urgent); }
```

and in chat.js `botRow`, when `opts.kind === 'emergency'`, add a primary call button in `.msg-tools` (outside the bubble): `<a class="tool-btn call" href="tel:9800000123">Call 98000 00123</a>` styled `background: var(--urgent); color: #fff; border-color: transparent` (dark: use `--urgent-bg` text on `--urgent`, check contrast).

Graded risk: none for `.bot-msg`, provided the label is a pseudo-element on `.msg-row` (not inside the bubble) and the call link stays in `.msg-tools`. Run `tests/contrast.mjs` for the red-tinted bubble in both themes.

### 4. Quieten the Listen and WhatsApp buttons

Seen: every reply carries one or two full 44 px outlined buttons. After five questions the screen at 375 is about one third buttons; on the 1440 and 1920 views the buttons read as louder than the answers. "Continue on WhatsApp" has no WhatsApp cue, so it reads as a generic button.

Change: keep the 44 px hit area, lose the box.

```css
.msg-tools { gap: 2px; margin-top: 2px; }
.tool-btn { border-color: transparent; background: transparent; color: var(--muted); font-size: .82rem; padding: 0 10px; gap: 6px; }
.tool-btn:hover { background: var(--surface-2); color: var(--ink); }
.tool-btn[data-speak]::before { content: ""; width: 16px; height: 16px; background: currentColor; -webkit-mask: url(/icons/speaker.svg) center/contain no-repeat; mask: url(/icons/speaker.svg) center/contain no-repeat; }
.tool-btn.wa { color: #0E7A3A; }               /* add a dark-mode token, e.g. --wa: #5BD68A */
.tool-btn.wa::before { /* same mask pattern with a WhatsApp glyph SVG */ }
/* older replies: keep the tools but fade them, so the latest answer leads */
.msg-row.bot:not(:nth-last-child(-n+2)) .msg-tools { opacity: .55; }
.msg-row.bot .msg-tools:hover, .msg-row.bot .msg-tools:focus-within { opacity: 1; }
```

Graded risk: none. The buttons live in `.msg-tools`, a sibling after `.bot-msg`, and stay 44 px tall. Run `tests/contrast.mjs`: muted text on the page background must stay at 4.5:1 (the existing `--muted` passes on `--bg`; the 0.55 opacity on older rows does not need to pass if they are still operable, but if the contrast test scans every button, drop the opacity rule).

### 5. The source label: style it as a badge, and check where it appears

Seen: "Answered from the clinic file" is 12.5 px grey text, readable but easy to miss. It also appears under "I am not sure about that, and I will not guess" and under the medical refusal, which reads as a contradiction (nothing was answered from the file), while the Saturday answer, worded by the model, has no label at all. A judge comparing three replies sees the trust signal behave inconsistently.

Change (visual, safe):

```css
.src-label { display: inline-flex; align-items: center; gap: 4px; margin-top: 8px; padding: 2px 8px 2px 6px; border-radius: 999px; background: var(--chip-bg); color: var(--chip-ink); font-weight: 600; }
.src-label::before { content: ""; width: 12px; height: 12px; background: currentColor; -webkit-mask: url(/icons/file-check.svg) center/contain no-repeat; mask: url(/icons/file-check.svg) center/contain no-repeat; }
```

Change (logic, flag first): in `ticketRows`, set `fromFile` only when the reply really quotes the file (`t.kind === 'answer'` and similar), and show the label on model-worded replies too, since those are checked against the file by `FD.guard`.

Graded risk: the styling is safe because the text inside `.bot-msg` does not change and the icon is a pseudo-element. The logic change is not safe until checked: `tests/lib.mjs` exports `LABEL = 'Answered from the clinic file'`, so the brief tests look for this label. Read "How we check" in the brief before changing which replies carry it.

## Priority 2: polish that judges will notice

### 6. Quick-reply chips at 375: show that they scroll

Seen: six chips are 573 px wide in a 351 px track. "Insurance" is cut at the edge with no fade, no snap and only a thin scrollbar (invisible on touch phones). It works, but nothing says there is more.

```css
.quick { scroll-snap-type: x proximity; scroll-padding-inline: 12px; scrollbar-width: none; padding-inline-end: 24px;
  -webkit-mask-image: linear-gradient(to right, #000 calc(100% - 28px), transparent); mask-image: linear-gradient(to right, #000 calc(100% - 28px), transparent); }
.quick::-webkit-scrollbar { display: none; }
.chip { scroll-snap-align: start; }
@media (min-width: 600px) { .quick { -webkit-mask-image: none; mask-image: none; flex-wrap: wrap; } }
```

Optional: put "Book a visit" first, styled as the one filled chip (`background: var(--primary); color: var(--on-primary)`), because it is the action a clinic most wants.

Graded risk: none. The mask is visual only and the chips stay buttons with `data-q`.

### 7. Use the empty space beside the chat at 1920

Seen: the side panel is 440 px wide with three cards; at 1080 px tall the lower 250 px is empty and the footer floats mid-panel. At 1440 the Timings card wraps "10:00 am to 2:00 pm, emergencies only" over three lines because the label column is wide.

Change, in index.html at the top of `<aside class="side">`, add a status card and an emergency card (no `bot-msg` anywhere):

```html
<section class="card status-card"><p class="status-line"><span class="dot"></span><span id="side-open">Open now</span> · closes 8:00 pm</p>
  <div class="side-actions"><a class="btn btn-primary" id="side-call">Call</a><a class="btn btn-ghost" id="side-wa">WhatsApp</a><a class="btn btn-ghost" id="side-map">Directions</a></div></section>
<section class="card emergency-card"><h2>Emergency</h2><p>Severe swelling, heavy bleeding or a broken tooth: call <a id="side-emg">98000 00123</a>, open on public holidays too.</p></section>
```

```css
.side-actions { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; margin-top: 10px; }
.emergency-card { border-color: var(--urgent); background: var(--urgent-bg); }
.side-foot { margin-top: auto; }
.kv { grid-template-columns: minmax(0, 8.5rem) 1fr; }
.side-docs li { display: grid; grid-template-columns: 40px 1fr; gap: 10px; }   /* initials avatar per doctor */
```

Fill the closing time and the map link from the clinic file in `side()`. Doctor initials in a 40 px teal circle make the Doctors card feel like a real practice page.

Graded risk: none. All new markup is outside `#chat-log`.

### 8. Greeting and empty state

Seen: with no history, the phone shows the greeting bubble and then about 440 px of blank grey above the chips. The greeting is a plain bubble identical to an answer.

Change: make the greeting a welcome card, keep its class `bot-greeting` (not `bot-msg`), and add a bot avatar so the assistant has a face.

```css
.greeting-row { max-width: min(560px, 100%); }
.bot-greeting { padding: 16px; border-radius: 18px; }
.bot-greeting::before { content: ""; display: block; width: 40px; height: 40px; margin-bottom: 10px; border-radius: 12px; background: var(--chip-bg) url(/icon.svg) center/24px no-repeat; }
.msg-row.bot:not(.greeting-row):not(.from-staff) { padding-left: 34px; position: relative; }
.msg-row.bot:not(.greeting-row):not(.from-staff)::before { content: ""; position: absolute; left: 0; top: 2px; width: 26px; height: 26px; border-radius: 8px; background: var(--chip-bg) url(/icon.svg) center/16px no-repeat; }
```

When the log holds only the greeting, it can also show the chips as two-column cards in the log on a phone (a separate `.starter` list after the greeting row, removed on the first question), so the empty space carries the next step. Put that list outside `.msg-row.bot` so it is never taken for a reply.

Graded risk: low. The greeting text is set by `labels()` through `textContent`; a `::before` keeps working. Any starter list must not use `.bot-msg` and must be removed on first send so it never sits after the answer.

### 9. Message rhythm and grouping

Seen: every row has the same 10 px gap, so a question, its answer and its buttons do not read as one exchange. Bubbles use 16 px radius with a 4 px "tail" corner; fine.

```css
.msg-row.user { margin-top: 14px; }
.msg-row.user:first-child, .greeting-row + .msg-row.user { margin-top: 4px; }
.bot-msg, .bot-greeting { box-shadow: 0 1px 2px rgb(0 0 0 / .05); }
.user-msg { box-shadow: 0 1px 2px rgb(0 0 0 / .12); }
```

A small time under each user bubble (`<time class="t">10:42</time>` appended to the user row, outside any `.bot-msg`) adds realism; never put a time under the last bot row, so the last thing in the log is still the reply plus its tools.

Graded risk: none for spacing. For timestamps, only on user rows.

### 10. Composer on a phone

Seen: the placeholder "Type your question, for example: What time do you open on Saturday?" is cut to "Type your question, for e" at 375. The mic, input and Send are well sized and stay pinned.

Change: in chat.js `labels()`, use a short placeholder when `matchMedia('(max-width: 480px)')` matches, for example "Ask about timings, prices, doctors". Give the input a pill shape to match the chips (`border-radius: 999px; padding: 0 18px`) and the Send button the same (`border-radius: 999px`). Keep the visible word "Send".

Graded risk: low. Keep `#chat-input` and `#chat-send`; check that no test finds the input by its placeholder text before shortening it.

## Priority 3: staff page

### 11. Staff ticket list: states that read at a glance

Seen: good start. Badges are colour-coded, urgent sorts first, the open ticket has a teal ring. Gaps:

- Counts show "Urgent 0" in red and "Needs human 3" in amber with equal weight; zero counts should recede.
- Closed tickets look identical to open ones.
- The open ticket repeats the question twice (header and "Patient" line).
- The patient id ("PJN60I") is raw; label it.
- At 375 the counts wrap to two lines ("Closed 0" alone) above a five-line hint.

```css
.count.is-zero { background: transparent; color: var(--muted); border: 1px solid var(--line); }
.ticket[data-status="closed"] .t-head { opacity: .7; }
.ticket[data-status="urgent"] { border-left: 5px solid var(--urgent); }
.ticket[data-status="needs human"] { border-left: 5px solid var(--needs); }
.t-head { grid-template-columns: auto minmax(0, 1fr) auto; }   /* room for the time on the right */
@media (max-width: 480px) { .counts { flex-wrap: nowrap; overflow-x: auto; } .count { font-size: .75rem; padding: 3px 8px; } }
```

In staff.js add `is-zero` to a count when it is 0, drop the first `t-line` (Patient) in the open body, and print the patient as `Patient PJN60I`. Shorten the hint to one sentence ("Urgent first. Reply on the open ticket and the patient sees it in their chat.") and put the status rules in a `<details>`.

Graded risk: the four status words must stay exactly "urgent", "needs human", "answered", "closed" wherever the tests read them (`data-status` and badge text are unchanged here). `#staff-reply` and `#staff-send` stay inside the open `[data-ticket]`.

### 12. Staff page at 1440 and 1920

Seen: at 1920 the ticket column is about 1,430 px; the reply textarea is 1,370 px wide and the bot reply runs on one very long line. A master and detail split (Front, Zendesk) would be the ideal, but it would move the reply box out of the ticket's `[data-ticket]`, which the tests rely on, so do not do that.

Change instead:

```css
@media (min-width: 1100px) { .staff-layout { max-width: 1480px; margin-inline: auto; grid-template-columns: minmax(0, 1fr) 400px; } }
.t-body { max-width: 760px; }
```

Graded risk: none.

### 13. Call-back list and the clinic file

Seen: call-back items are underlined teal links with an odd gap above the meta line (the 44 px link height). The clinic file is a raw JSON textarea, which is honest but engineer-facing.

```css
.call-list li { padding: 8px 10px; border: 1px solid var(--line); border-radius: 10px; gap: 0; }
.call-list .link-btn { min-height: 32px; text-decoration: none; padding-block: 6px; }
.call-list .link-btn::before { content: ""; display: inline-block; width: 14px; height: 14px; margin-right: 6px; vertical-align: -2px; background: currentColor; -webkit-mask: url(/icons/phone.svg) center/contain no-repeat; mask: url(/icons/phone.svg) center/contain no-repeat; }
```

Keep the whole `li` as the hit area (make the button `display: block; width: 100%`) so the target stays at least 44 px. For the clinic file, add a one-line summary above it ("3 doctors · 14 prices · last saved just now") and keep the textarea visible.

Graded risk: low; keep a 44 px target. Leave `#clinic-json` visible in case a test edits it.

### 14. Staff empty states

Seen: "No tickets yet. Questions from the patient chat appear here." is plain muted text under a long hint, and the counts still show four zero pills.

```html
<div class="empty-state" id="queue-empty" hidden><svg ...inbox icon.../><p><strong>No tickets yet</strong></p><p class="muted">Questions from the patient chat appear here. <a href="/" target="_blank">Open the patient chat</a> to try one.</p></div>
```

```css
.empty-state { text-align: center; padding: 32px 16px; color: var(--muted); }
.empty-state svg { width: 40px; height: 40px; color: var(--line); }
```

Hide `.counts` when every count is zero. Keep the id `queue-empty`.

Graded risk: none, provided the id stays.

## Dark mode notes

Dark mode is good: real surface steps (`--bg` #0D1517, `--surface` #152124, bubbles #1A2A2D), the header keeps the gold rule, badges use tinted backgrounds rather than solid fills. Three refinements:

- The user bubble (#2E8C86 with near-black text) reads muddy next to the bright mint Send button (#46C2B8). Use one hue for both: `--user-bg: #3FB3A9; --user-ink: #04110F;` and check 4.5:1 with `tests/contrast.mjs`.
- Bubble borders (#2B3E41) almost vanish; use `--line: #33494D` or drop the border on dark bot bubbles and rely on the fill.
- The side panel uses `--surface-2` (#1C2B2E) behind cards of `--surface` (#152124), so cards are darker than their background, which reads as holes. In dark, swap: `.side { background: var(--bg); }` and keep cards on `--surface`.

Graded risk: none beyond rerunning the contrast test.

## Typography notes

- Body text at 16/24 is right for patients. The 12.5 px source label and 12.5 px fine print are the floor; do not go smaller.
- Baloo 2 is used only for the brand and card headings; use it also for "Tickets", the welcome card heading and the emergency label, so the voice is consistent.
- Side-panel `kv` labels in muted grey and values in ink is good; tighten the label column (item 7) so values do not wrap to three lines.

## Top eight, in order

1. Centre the chat in an 820 px reading column on wide screens (`padding-inline: max(32px, calc((100% - 820px) / 2))` on `.chat-log` and `.composer`).
2. On phones, show the clinic name instead of "FrontDesk AI" and keep a one-line emergency number in the composer.
3. Give emergency replies a red-tinted bubble, an "Emergency" label on the row and a Call button in `.msg-tools`; and raise that "severe swelling and bleeding" is not triaged as urgent.
4. Make Listen and WhatsApp quiet icon-and-text buttons (no border, muted, WhatsApp glyph), fading on older replies, 44 px hit area kept.
5. Style "Answered from the clinic file" as an icon badge; review, against the brief, why it shows on "I am not sure" replies and not on model-worded ones before changing that logic.
6. Chips at 375: edge fade, scroll snap, hidden scrollbar; "Book a visit" first and filled.
7. Fill the wide side panel: open status with Call, WhatsApp and Directions buttons, an emergency card, doctor initials, footer pinned to the bottom.
8. Staff page: mute zero counts, dim closed tickets, left status bar on urgent and needs human, remove the duplicated patient line, cap the layout at 1,480 px.

## Status after fixes (21 Nov 2026)

Fixed in the code repository commit "Review fixes: model cannot be steered, stricter guard, engine gaps, call-back details, design", then every test rerun on the live link: brief 28/28 with the key and 30/30 with the model blocked, questions 16/16, bonus 15/15 both ways, widths 20/20, contrast clean, engine ALL PASS (files in `../evidence/`).

Fixed:

- 1. Wide screens: the conversation sits in a centred reading column.
- 2. Phones: the clinic name and the emergency number stay on screen.
- 3. Emergency replies stand out, with a Call button beside the reply.
- 4. Listen and WhatsApp are quieter reply tools.
- 6. Quick-reply chips at 375 px snap as they scroll.
- 7. The space beside the chat at 1920 px is used, with a Today card.
- 11. Staff ticket states read at a glance, with status colours.

Open:

- 5. The source label as a badge: left as it is, because "Answered from the clinic file" is graded text and
  the review itself asks for the logic of where it shows to be checked against the brief before any change.
- 8. Greeting and empty state, 9. message rhythm and grouping, 10. composer on a phone: polish only, left
  for after the competition; no graded check depends on them.
- 12. Staff page at 1440 and 1920 px, 13. call-back list and the clinic file, 14. staff empty states: polish
  on the staff page, left for after the competition. The widths test passes on /staff at every width.

# FrontDesk AI data model

FrontDesk AI keeps everything in the browser. The clinic's facts come from one JSON file, and the
tickets, the staff replies and the edited clinic file live in `localStorage`, shared by the chat page
(`/`) and the staff page (`/staff`) of the same browser. There is no database. The one server function
only rewords a reply (see [schema/api-chat.md](schema/api-chat.md)). The source of truth is `core.js`,
`chat.js` and `staff/staff.js`; the schemas in [schema/](schema/) give the exact field rules.

![Data model](diagrams/data-model.svg)

## Entities

| Entity | Where | Key | What it holds |
| --- | --- | --- | --- |
| Clinic file | `/clinic-data.json`, or the edited copy in `frontdesk.clinic` | none (one in use) | `clinic`, `hours`, `doctors`, `services`, `policies`, `emergency_rules`, plus `note` and `bot_rules`, which the code does not read |
| Patient session | `frontdesk.patient` | the id itself | `"P"` + random base-36 characters in upper case (normally five), made the first time the chat page needs it |
| Ticket | `frontdesk.tickets[]` | `id` | one patient message and its reply, with status and staff replies (fields below) |
| Staff reply | `ticket.staffReplies[]` | position | `text` and `at` |
| Preferences | `frontdesk.lang`, `frontdesk.theme` | | the language menu and the theme |

Field rules: [schema/clinic-file.schema.json](schema/clinic-file.schema.json) and
[schema/tickets.schema.json](schema/tickets.schema.json).

### Clinic file sections

| Section | Required by `validateClinic()` | Read by the code for |
| --- | --- | --- |
| `clinic` | yes, an object | `name` (greeting, page title), `address` and `landmark` (location), `phone`, `whatsapp`, `emergency_phone` (contact, medical and emergency replies, links), `languages`, `parking` |
| `hours` | yes, an object | day timings, `lunch_break`, `public_holidays`, and the open or closed check (see "Opening hours") |
| `doctors` | no; a list when present | `name`, `role`, `days`: doctor and children replies |
| `services` | no; a list when present | `name`, `price`, `duration`, `note`: price and "do you offer" replies |
| `policies` | no | `appointments`, `walk_ins`, `payment`, `emi`, `insurance`, `children`, `first_visit`, `sterilisation` |
| `emergency_rules` | no | the text after the emergency number |
| `note`, `bot_rules` | no | not read; they reach the model only as part of the whole file sent with each request |

`validateClinic()` checks nothing inside the sections. A missing field the engine needs is printed as it
is in the reply.

### Ticket

Every message a patient sends makes one ticket, whatever the reply. `chat.js` builds it and
`addTicket()` adds `id`, `created` and `staffReplies`.

| Field | Written by | Value |
| --- | --- | --- |
| `id` | `addTicket()` | `"T"` + `Date.now()` in base 36 + two random base-36 characters, all upper case |
| `patient` | `chat.js` | this browser's `frontdesk.patient` id; the chat shows only its own tickets |
| `q` | `chat.js` | the message, trimmed |
| `reply` | `chat.js` | the model's text when it passed the guard, otherwise the rules engine's text |
| `source` | `chat.js` | `model` or `file` |
| `modelOk` | rules engine via `chat.js` | `true` when this was a real file answer that may go to the model; the UI shows "Answered from the clinic file" only when `source` is `file`, `kind` is `answer` and `modelOk` is true |
| `kind` | rules engine | `emergency`, `refuse`, `medical`, `booking`, `answer` or `notsure` |
| `status` | rules engine, then staff | `urgent`, `needs human`, `answered` or `closed` |
| `lang` | rules engine | `te` when the message has Telugu script, `hi` for Devanagari, else `en` (the language menu does not set it) |
| `bookingTime` | rules engine | for a booking, the day and time read from the message (for example `tomorrow 5:00 pm`) or `not given`; otherwise `""` |
| `afterHours` | `chat.js` | `true` when the clinic was closed at that moment |
| `afterHoursNote` | `chat.js` | the closed-now note in the ticket's language, or `""` |
| `callBack` | `chat.js` | `true` when the rules engine gave `needs human`; never changed later |
| `created` | `addTicket()` | `Date.now()` |
| `updated` | `replyToTicket()`, `setStatus()` | `Date.now()` of the last change; absent until then |
| `staffReplies` | `addTicket()`, then `replyToTicket()` | `[]`, then `{ text, at }` items, oldest first |

Nothing deletes a ticket.

## localStorage keys

| Key | Value | Written by | Read by |
| --- | --- | --- | --- |
| `frontdesk.clinic` | the edited clinic file, JSON | Save on the staff page (`saveClinic()`) | `clinic()` on both pages; removed by Reset |
| `frontdesk.tickets` | JSON array of tickets | `addTicket()`, `replyToTicket()`, `setStatus()` | both pages; a `storage` event re-renders the other page |
| `frontdesk.patient` | patient session id, plain text | `patientId()` | `chat.js` |
| `frontdesk.lang` | `en`, `hi` or `te`, plain text | the language menu (`setLang()`) | `getLang()`, default `en` |
| `frontdesk.theme` | `light` or `dark`, plain text | the theme button (`toggleTheme()`) | `applyTheme()`; when absent the system setting applies |

Every read and write is wrapped so that a blocked or full `localStorage` does not stop the page: a failed
read gives the default and a failed write is ignored.

## Status

| Reply kind | Status given | Notes |
| --- | --- | --- |
| `emergency` | `urgent` | |
| `refuse` (rule change or discount) | `answered` | |
| `medical` | `needs human` | |
| `booking` | `needs human` | `bookingTime` set |
| `answer` | `answered` | includes the greeting and thanks replies |
| `notsure` | `needs human` | also when no clinic file could be loaded |

| Staff action | Status before | Status after |
| --- | --- | --- |
| Send reply (`replyToTicket()`) | `urgent` or `needs human` | `answered` |
| Send reply | `answered` | `closed` |
| Send reply | `closed` | `closed` |
| Close ticket (`setStatus(id, 'closed')`; the button shows on any ticket not yet closed) | any | `closed` |

A staff reply is also shown in the patient's chat under the ticket, marked as from the clinic team.

## Priority on /staff

The queue is sorted by status: `urgent`, then `needs human`, then `answered`, then `closed`; within a
status, newest first. The first ticket is open by default. A `needs human` ticket older than two hours is
marked "over 2 h". The call-back list holds every `needs human` ticket and every booking not yet closed,
oldest first. The page re-renders every 30 seconds unless the reply box has focus.

## The rules engine

`answer(message, data)` returns `{ kind, status, text, lang, facts, bookingTime, modelOk }`. The message is
lower-cased and tested in this fixed order; the first match wins.

| Step | Detected by | Result |
| --- | --- | --- |
| 1. Emergency | swelling with face, neck, cheek or jaw (either order), breathing words, bleeding that will not stop or is heavy, unconscious, fainted, accident, broken jaw (with Hindi and Telugu words) | `emergency`, `urgent`; the emergency number first, then `emergency_rules` |
| 2. Rule change or discount | instruction words (ignore or forget the rules, disregard, override, you are now, act as, pretend, system prompt, jailbreak) or discount words (discount, a number with %, percent, concession, cheaper, reduce or lower the price, bargain, coupon, price match, for free) | `refuse`, `answered`; a fixed refusal |
| 3. Medical | pain, ache, sensitivity, bleeding, swelling, infection, fever, medicine words, diagnosis questions, broken, chipped or loose tooth, unless the message asks how to book, cancel or reschedule | `medical`, `needs human`; no advice, offers a visit with phone and WhatsApp |
| 4. Booking | a booking word (book, appointment, schedule, reserve, slot, come in) and not a how-to-book question, together with a day or time read from the message or a request phrase (I want, I'd like, can I, please, need) | `booking`, `needs human`; no slot is confirmed; the `appointments` policy is added in English |
| 5. Answer | a greeting or thanks on its own; otherwise the parts found for children, insurance, services and prices, doctors, hours, location, parking, contact, payment or EMI, how to book, walk-ins, first visit, sterilisation and languages, joined | `answer`, `answered`; the values it used are kept as `facts` and only a composed answer may go to the model |
| 6. Not sure | nothing found; or a price asked for something the file does not price; or a named treatment the file lacks (Invisalign, veneer, laser, crown, extraction and others) | `notsure`, `needs human`; says it will not guess and that someone will call back within 2 working hours |

Every reply text is built from the clinic file in use; values are copied as written.

## The guard

When a reply of kind `answer` was sent to the model, the model's text is used only when `guard()` passes
all of these; otherwise the rules engine's text is shown.

- The text is not empty and is at most 900 characters.
- Every amount written `Rs` plus digits appears in the clinic file (spaces ignored).
- Every time written like `9:00 am` appears in the clinic file.
- Every doctor written `Dr. Name` appears in the clinic file.
- No `%`, "discount", "percent", "off the price" or "free of charge", unless it says "no discount" or
  "cannot offer a discount".
- No medicine or home remedy from a fixed list (paracetamol, ibuprofen, antibiotic, clove oil, warm salt
  and others).
- Every fact the rules engine found is still there, word for word (the part before the first comma, when
  it is at least four characters).
- For a Hindi or Telugu message, the text is in the same script.

`tokens()` also collects phone numbers, but `guard()` does not check them.

## Opening hours

The `hours` keys are read by name. A day matches a key when the key contains the day name (`sunday`) or
is a range written `day_to_day` (`monday_to_saturday`, which may wrap past Sunday). The first matching
key in file order wins. `lunch_break` and `public_holidays` are used only when the question says lunch or
holiday.

`openNow()` takes today's value on the device clock. The clinic counts as closed when there is no value,
when the value says "closed", "emergency only" or "emergencies only", or when the time is outside the
first `h:mm am to h:mm pm` range in the value. Lunch breaks and public holidays are not part of this
check. `nextOpening()` looks at today and the next seven days for the next opening with a range and gives `today`,
`tomorrow` or the day name, with the start time.

After hours changes only the note. The ticket gets `afterHours: true` and an `afterHoursNote` ("We are
closed right now and open tomorrow at 9:00 am. Our team will call you back when we open."). The reply,
kind and status are never changed. The chat shows the note below a reply that needs a call (status not
`answered`, or a booking), but not for an emergency. The staff page shows "after hours" on the ticket.
The header chip shows "Open now" or "Closed now", refreshed every minute.

## Edited clinic file and reset

On the staff page the clinic file in use is shown as JSON. Save parses it, runs `validateClinic()` and,
when both pass, writes the whole object to `frontdesk.clinic`. From then on `clinic()` returns the edited
copy instead of the shipped file, on both pages, whenever the stored value is an object with a `clinic`
section; the chat page picks it up at once through the `storage` event. The edited copy replaces the
original as a whole; nothing is merged. Reset removes `frontdesk.clinic`, so the shipped
`/clinic-data.json` is used again. When `/clinic-data.json` cannot be fetched, there is no original file
(`window.FRONTDESK_CLINIC` is read as a fallback, but no page sets it) and every message gets the
not-sure reply unless an edited copy is stored.

## Validation of the brief's clinic file

`clinic-data.json` in the code folder was checked against
[schema/clinic-file.schema.json](schema/clinic-file.schema.json) on 20 November 2026 with a small
hand-written Node check (types, required fields, lists and nested objects), and against the same rules as
`validateClinic()`.

- Schema: pass, 82 values checked, no errors.
- `validateClinic()` rules: pass.
- The schema refuses the same four broken files that `validateClinic()` refuses: an empty object, no
  `hours`, `services` that is not a list, and `doctors` that is not a list.

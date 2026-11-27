# FrontDesk AI data model

FrontDesk AI keeps its working copy in the browser. The clinic's facts come from one JSON file, and the
tickets, the staff replies and the edited clinic file are kept in `localStorage` as the working copy of the chat
page (`/`) and the staff page (`/staff`). Since version 1.4.0.0 the shared copy of tickets, messages and the
clinic file is in Postgres (see "Added in version 1.4.0.0 to 1.5.0.0" below). Since version 1.3.0.0 a patient's tickets are also
mirrored to a server store for seven days (see "Added in version 1.3.0.0" below). One server function
only rewords a reply (see [schema/api-chat.md](schema/api-chat.md)); a second, `api/tickets.js`, keeps the
shared tickets. The source of truth is `core.js`,
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
| `frontdesk.alerts` (since 1.1.0.0) | "1", plain text; absent when alerts are off | the alerts switch on /staff (`staff/tools.js`) | `staff/tools.js` |
| `frontdesk.notes` (since 1.2.0.0) | JSON object, ticket id to private staff note | Save note on /staff (`staff/tools.js`) | `staff/tools.js`; never shown to the patient |
| `frontdesk.synckey` (since 1.3.0.0) | this chat browser's private key for its tickets on the server, plain text | `sync.js`, made the first time the key is needed | `sync.js` |
| `frontdesk.alldevices` (since 1.3.0.0) | "1" when "Show tickets from all devices" is on; absent when off | the switch on /staff (`sync.js`) | `sync.js` |

One more key is kept in **sessionStorage**, not localStorage:

| Key | Value | Written by | Read by |
| --- | --- | --- | --- |
| `frontdesk.deskcode` (since 1.3.0.0) | the clinic's desk code as typed by staff on /staff, plain text. Its value is never written in any document | `sync.js`, when staff enter the code | `sync.js`, sent as the `x-desk-code` header to `api/tickets`. It lasts for the browser session only and is gone when the tab closes |

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

## Added in version 1.1.0.0 (22 Nov 2026)

- **Clinic file, optional `assistant` section:** `name` (text, at most 40 characters) and `quick_replies`, a list of
  at most eight `{ label, question }` pairs (label at most 24 characters, question at most 120). A file without the
  section keeps the standard name and the seven standard quick replies.
- **Ticket `kind`:** a new value, `human`, for a patient who asks for a person. Its status is `needs human`.
- **Browser storage:** `frontdesk.alerts` = "1", written only when staff turn urgent alerts on.

The diagram `diagrams/data-model.svg` shows all three.

## Added in version 1.2.0.0 (22 Nov 2026)

- **Browser storage:** `frontdesk.notes` = a JSON object that maps a ticket id to that ticket's private staff
  note. It is written only when a note is saved. The note is never shown to the patient and is not part of the
  ticket.

Nothing else in the data changed in 1.2.0.0. The queue figures, the CSV export and the filter by ticket type
are worked out from the tickets when the staff page is drawn.

## Added in version 1.3.0.0 (22 Nov 2026)

**In the browser**

| Item | Where | Value |
| --- | --- | --- |
| `feedback` | a ticket field | the patient's "Helpful" or "Not helpful" on that reply |
| `remote: true` | a ticket in a staff browser | marks a ticket that came from another device; these leave the list when the switch goes off |
| `frontdesk.synckey` | localStorage, chat browser | this browser's private key for its tickets on the server |
| `frontdesk.alldevices` | localStorage, staff browser | whether "Show tickets from all devices" is on; off by default |
| `frontdesk.deskcode` | sessionStorage, staff browser | the desk code typed by staff, kept for the browser session only and gone when the tab closes |

Ticket numbers (T-001, T-002 and so on) are worked out from the order the tickets were opened. The
ticket `id` is unchanged.

**On the server (the ticket store behind `api/tickets.js`)**

| Record | Key | What it holds | Kept for |
| --- | --- | --- | --- |
| Ticket | one record per ticket id | the ticket as mirrored by the chat, plus `owner`, a hash of the browser key that created it | seven days |
| Order | one ordered list | every ticket id by time, read newest first | the newest 300 ids |
| Per owner | one small set per `owner` | the ticket ids of that browser key | see `api/tickets.js` |

Rules the function applies:

- A ticket's question, reply and patient never change once stored.
- The server keeps only the hash of a browser key, never the key.
- Requests are rate limited.
- With no store configured the function answers 503 and both pages work from the browser alone.

**Who can read what**

| Who | Can read and change |
| --- | --- |
| A patient's browser, with its own key | only the tickets that key created |
| Anyone who knows a ticket id or a patient id, without the key | nothing |
| A staff browser with the clinic's desk code | the list of every device's tickets, and it can reply to them |
| A staff browser without the code, or before the clinic has set a code on the server | only the tickets of its own browser |

On the server every ticket is deleted after seven days.

## Added in version 1.4.0.0 to 1.5.0.0 (27 Nov 2026)

The shared store, `api/_store.js`: Postgres on Neon when `DATABASE_URL` is set (plain SQL, values as
parameters, each query limited to 6 s), else the host's Redis under the 1.3 keys, else none.

| Table | Columns | Kept |
| --- | --- | --- |
| `messages` | `id` BIGSERIAL key, `chat_id`, `role` (patient, bot, staff), `text` (at most 6,000 characters), `source`, `owner` (hash of the browser key), `created_at`; index on (`chat_id`, `id`) | seven days, at most 400 a chat |
| `tickets` | `id` key, `patient`, `owner`, `status`, `priority`, `text`, `reply`, `data` JSONB (the whole ticket, shape in [schema/tickets.schema.json](schema/tickets.schema.json)), `created_at`, `updated_at`; index on `owner` | seven days, at most 300 in the shared list |
| `clinic_data` | `id` (always 1), `data` JSONB (the clinic file, shape in [schema/clinic-file.schema.json](schema/clinic-file.schema.json), at most 100,000 characters), `updated_at` | until reset |

| Route | Method | Who | Answer |
| --- | --- | --- | --- |
| `/api/clinic` | GET | anyone | `{ clinic, saved, store }`; the shipped file when the store is down |
| `/api/clinic` | POST `{ clinic }` or `{ reset: true }` | staff, `x-desk-code` | `{ ok, clinic, saved }`; 400 with the failing part, 401, 403, 429, 503 |
| `/api/messages` | POST `{ chatId, role, text, source }` | the browser key in `x-patient-key` | `{ message }` |
| `/api/messages?chat=<id>` | GET | that browser key, or staff | `{ messages }`, oldest first |
| `/api/health` | GET | anyone | `{ ok, store: "postgres" \| "redis" \| "none", ai }` |

New browser keys, all written by `shared.js`:

| Key | Where | What |
| --- | --- | --- |
| `frontdesk.clinic.server` | localStorage | the clinic file in this browser came from the server |
| `frontdesk.clinic.own` | localStorage | staff saved or reset on this device only; the server copy is not applied over it |
| `frontdesk.msgseen` | localStorage | which messages were already sent to `/api/messages` |

`frontdesk-urgent` is not a stored key: it is the tag of the urgent-ticket notification in `staff/tools.js`.
`frontdesk.alldevices` now defaults to on: the shared view is off only when staff untick it on that device.
No shape of the clinic file or the ticket changed.

Versions 1.5.0.1 to 1.6.0.1: no key, table, field or schema added or changed; the thumbs on a phone write the same
`feedback` field ("up" or "down") as the Helpful and Not helpful buttons.

Version 1.7.0.0: no key, table, field or schema added or changed; the desktop icon buttons write the same
`feedback` field.

Version 1.7.1.0: no key, table, field or schema added or changed; the desktop "+" menu and the staff link words
store nothing.

Version 1.7.2.0: no key, table, field or schema added or changed. `callback-ask.js` and `mark-link.js` read and
write no `localStorage` or `sessionStorage` key. `sync.js` reads the desk code key it already had before it asks
for the list. The phone number added to an answer comes from the clinic file's `clinic.phone`. The chat's fine
print now states what was already so: the server keeps the messages for 7 days.

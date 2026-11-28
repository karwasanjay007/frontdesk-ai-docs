# FrontDesk AI: what changed in each version

This is the only page that tells the history; every other page describes today's version.

## Two versions

| Version | Date | Where | What it is |
| --- | --- | --- | --- |
| 1.7.2.0 | 27 November 2026 | branch `main` | Today's version, the one on the live link |
| 1.0.0.0 | 21 November 2026 | branch [`first-release-1.0.0.0`](../../tree/first-release-1.0.0.0) | The first release, with its documents as they stood then |

What today's version has that the first release did not:

| In the user's words | First release 1.0.0.0 | Today 1.7.2.0 |
| --- | --- | --- |
| Where tickets live | In one browser | In Postgres, shared across devices; staff see every device's tickets with a desk code |
| The clinic file after a staff edit | Changed in that browser | Saved for every device with the desk code |
| The chat's messages | In the browser only | Also kept on the server for 7 days, so staff see the whole chat in a ticket |
| Asking for a person | The not-sure reply | Its own reply, ticket and quick reply |
| Staff queue tools | A list in priority order | Search, filters, sort, figures, ready-made replies, private notes, CSV export, urgent alerts |
| Editing the clinic file | A JSON box | A form as well, with the assistant's name and quick replies |
| Tickets | No numbers | Ticket numbers, the chat thread, Helpful and Not helpful |
| Phone chat | Controls round the message box | Starter cards, a "+" menu, a slim message box |
| Desktop chat | A clinic information drawer | Clinic information pinned on the right, answers in a column |
| The model step | Called from the page's own function | Temperature 0, one attempt, 8 seconds, checked by the guard on the server as well, with a request limit |
| A doctor the clinic does not have | Could be answered with another doctor | Said so, with the clinic's own doctors |
| Mixed questions such as timings and prices | Not answered together | Both parts in one reply |

The versions follow, newest first.

## Version 1.7.2.0 (27 November 2026)

- A question that asks for the phone number gets the number, also when it asks for something else as well.
- While no phone number is known, the call-back form says so and offers the clinic's own number, in English, Hindi and Telugu.
- The small print under the message box wraps instead of being cut, and says that messages are kept on the clinic's server for 7 days.
- The staff page opened without the desk code no longer asks the server for a list it will be refused.
- The mark in the top bar opens the maker's site in a new tab.

## Version 1.7.1.0 (27 November 2026)

- The "+" menu is on the desktop too, as a small popover above the message box.
- New chat and the microphone moved into that menu; the message box reads "+", the question, Send.
- The staff link in the top bar is in Hindi and Telugu as well.

## Version 1.7.0.0 (27 November 2026)

- The desktop chat matches the phone: answers as plain text in a column beside the clinic information.
- Listen, Helpful and Not helpful are icon buttons that keep their names.
- A slim message box, and the emergency number as a red phone pill.
- Starter cards three to a row on an empty chat.

## Version 1.6.0.1 (27 November 2026)

- The starter cards take the full width of the phone.
- The language box shows "EN" whole, and the welcome line wraps.

## Version 1.6.0.0 (27 November 2026)

- On a phone an empty chat shows the quick replies as starter cards in two columns.
- A slim row holds the emergency number, the language and the theme.
- Thumbs up and down for feedback; answers are plain text with the clinic mark.
- A not-sure reply and a talk-to-a-person reply no longer carry the label "Answered from the clinic file".

## Version 1.5.0.1 (27 November 2026)

- Asking for a doctor who is not at the clinic gets "We have no Dr. ..." with the clinic's own doctors, never another doctor's days.

## Version 1.5.0.0 (27 November 2026)

- The phone chat redesigned: a slim message box with a "+" menu that opens as a bottom sheet.
- The quick replies sit in one sideways row once the chat has begun.
- Messages take most of the screen.

## Version 1.4.0.1 (27 November 2026)

- No green anywhere: the Answered status and the Open now sign are slate blue.

## Version 1.4.0.0 (27 November 2026)

- Tickets, chat messages and the clinic file are kept in Postgres and shared across devices.
- A clinic file saved by staff with the desk code applies on every device from the next question.
- The brand look: new colours, fonts and mark.
- A staff link in the top bar, and the clinic information pinned on the right on a wide screen.

## Version 1.3.1.0 (27 November 2026)

- An emergency reply begins with the emergency number in every language.
- A treatment asked with a variant the clinic file does not hold, such as laser, gets the not-sure reply and a ticket.
- Discount and rule-change requests typed in Roman-script Hindi are refused.
- The model's wording is checked against the clinic file on the server as well as in the browser.

## Version 1.3.0.3 (25 November 2026)

- A staff reply or a patient question sent just before its tab goes to the background is no longer held back.

## Version 1.3.0.2 (25 November 2026)

- "What are your timings and prices?" is answered with both.
- The theme button says which theme is on.
- The skip link puts the cursor in the message box.

## Version 1.3.0.1 (23 November 2026)

- Every tap target on both pages is at least 44 px.

## Version 1.3.0.0 (22 November 2026)

- Ticket numbers on the staff page.
- "Chat so far" under an open ticket shows the patient's whole chat.
- Helpful and Not helpful under a reply.
- Tickets shared across devices: staff see every device's tickets after entering a desk code.

## Version 1.2.0.0 (22 November 2026)

- The queue at a glance: open, urgent, over 2 hours, answered, average wait, feedback.
- Export CSV, ready-made replies, a filter by ticket type and a private staff note.
- A booking request that names a day also gets that day's hours.

## Version 1.1.0.0 (22 November 2026)

- Asking for a person or a call back gets its own reply and ticket, with a seventh quick reply, "Talk to a person".
- Staff can search, filter by status and sort the tickets, and turn on urgent alerts.
- "Edit with a form" beside the JSON box of the clinic file.
- The clinic file can set the assistant's name and the clinic's own quick replies.

## Version 1.0.0.0 (21 November 2026)

The first release.

- The patient chat: answers only from the clinic file, in English, Hindi or Telugu, typed or spoken, with Listen to hear a reply.
- Emergency number first, no medical advice, no discounts; what the file cannot answer becomes a ticket with a promised call back.
- The staff page: tickets in priority order with their age, a reply that reaches the patient's chat, a call-back list, Clear all tickets with Undo.
- A clinic-file editor, quick replies, a clinic information drawer, light and dark.
- Tickets and the clinic file were kept in the browser only.
- Shortly after, with the number unchanged, more ways of wording an emergency, timings and prices were understood.

A rebuild numbered 2.0.0 and 2.0.1 went on the live link on 26 and 27 November 2026 and was withdrawn on 27 November 2026 because it lacked features of the earlier version; the live link went back to the earlier version.

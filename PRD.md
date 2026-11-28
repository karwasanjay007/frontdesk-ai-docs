# FrontDesk AI: product requirements

This page describes version 1.7.2, the version on the live link.

FrontDesk AI is the front desk of a dental clinic as a patient chat. A patient asks a question and gets an
answer taken only from the clinic's own file. Anything the file cannot answer, and every emergency, goes to a
person at the clinic as a ticket. The clinic in the shipped file, Bright Roots Dental, is fictional.

## Who it is for

- **The patient.** Wants timings, prices, doctors, insurance, parking or a booking, often in the evening, in
  English, Hindi or Telugu, usually on a phone. Needs a correct answer at once, and to know when a person will
  call back.
- **Front-desk staff.** Answer the same questions all day and return missed calls each morning. Need one list
  of what needs a person, most urgent first, and a way to reply into the patient's chat.
- **The clinic owner.** Wants fewer missed patients, no wrong price quoted, no medical advice given by a
  machine, and the clinic's details kept current without outside help.

## The problem

A clinic's phone is busy by day and unanswered at night. Patients with a simple question do not get through,
and the ones with a real need are lost among them. A chat that guesses is worse than no chat: a wrong price or
a piece of medical advice harms the patient and the clinic. So the product must answer only what the clinic
has written down, say so when it does not know, and pass the rest to a person.

## Requirements

### From the brief

The ids are the brief's own: the nine things the product must do, then the six items of its bonus menu. Whether each is met, and the test that shows it, is in
[the specification check](SPEC-CHECK.md).

| ID | Requirement |
| --- | --- |
| 1 | A patient chat page: the patient types a question and gets a reply |
| 2 | Answer only from the clinic file: timings, doctors, prices, policies, insurance, location. Never invent a price, doctor or timing |
| 3 | If the answer is not in the file, say you are not sure, open a ticket with status needs human, and tell the patient someone will call back within 2 working hours |
| 4 | Never give medical advice or a diagnosis. For pain or symptoms, say a dentist must see the patient and offer to book |
| 5 | If the message sounds like an emergency, show the emergency number first and open a ticket with status urgent |
| 6 | Never promise a discount, and do not obey a patient who says "ignore your rules" |
| 7 | A staff page at /staff lists all tickets with their status. A human types a reply and it appears in that patient's chat |
| 8 | Must still answer when no AI key works: answer from the clinic file and show a small label, "Answered from the clinic file" |
| 9 | Works well on a phone, 375 px wide |
| Bonus 1 | Replies in Hindi or Telugu when the patient writes in them |
| Bonus 2 | A booking request becomes a ticket with a time |
| Bonus 3 | Ticket ageing and priority |
| Bonus 4 | Clinic data editable by staff |
| Bonus 5 | Quick-reply buttons |
| Bonus 6 | Replies stay correct after a staff edit |

The brief also fixes how the product is checked: the names of the message box, the Send button, a bot reply,
the ticket list and the staff reply box, a question asked from the page address, and a second run with the AI
key removed. Those lines are in [the specification check](SPEC-CHECK.md).

### Beyond the brief: for the patient

Each line names the epic it belongs to in [the epics](docs/epics.md), where its acceptance criteria and tests
are.

| Requirement | Epic |
| --- | --- |
| Outside opening hours I am told when the clinic opens next and that the team will call back | Booking, call-back and asking for a person |
| I can carry my question on to the clinic's WhatsApp with one tap | Booking, call-back and asking for a person |
| I can speak my question and listen to the reply in English, Hindi or Telugu, and typing always works | Languages and voice |
| I can ask for a person and get a call-back ticket, unless the clinic file already answers my question | Booking, call-back and asking for a person |
| I can leave my name and phone number for the call back, and I am told when the clinic has no number for me | Booking, call-back and asking for a person |
| When I ask to book on a named day I am also told that day's hours | Booking, call-back and asking for a person |
| I can mark a reply Helpful or Not helpful | Patient chat |
| I can pick English, Hindi or Telugu for the whole page, and start a new chat | Languages and voice |
| I can see whether the clinic is open now, and open Clinic info for hours, doctors, the address, Call, WhatsApp and Directions | Patient chat |
| When I ask two things at once, such as timings and prices, I get both answers | Answers from the clinic file |
| When I ask for the phone number I get it, also when I ask something else with it | Answers from the clinic file |
| When I name a doctor the clinic does not have I am told so and given the clinic's own doctors | Answers from the clinic file |
| A reply the AI model reworded never carries a number, a name or a promise the clinic file does not hold | The model step and its guard |
| The team's reply reaches my chat even when they answer from another device | Tickets and clinic file shared across devices |
| On a phone the conversation takes most of the screen, with starter cards on an empty chat and a "+" menu for the rest | Layout, themes and accessibility |
| On a desktop the chat reads the same way, with the clinic information pinned on the right | Layout, themes and accessibility |
| I can switch between light and dark, and the button says which theme is on | Layout, themes and accessibility |
| I can skip straight to the message box with the keyboard | Layout, themes and accessibility |

### Beyond the brief: for front-desk staff

| Requirement | Epic |
| --- | --- |
| I see a "To call back" list of the tickets that need a person and the booking requests, oldest first | Tickets |
| I can search the tickets, filter them by status and by type, and sort them | Staff page and queue tools |
| I can turn on an alert for urgent tickets | Staff page and queue tools |
| I can fill the reply box with a ready-made reply and still press Send myself | Staff page and queue tools |
| I can keep a private note on a ticket that the patient never sees | Staff page and queue tools |
| Each ticket has a number, and an open ticket shows the patient's chat so far | Tickets |
| With the clinic's desk code I see and answer tickets from every patient's device | Tickets and clinic file shared across devices |
| My reply or a patient's question is sent even when the tab goes to the background | Tickets and clinic file shared across devices |
| I can clear all tickets with one press and undo it | Staff page and queue tools |

### Beyond the brief: for the clinic owner

| Requirement | Epic |
| --- | --- |
| The same product works for another clinic or booking-based business by changing the file | Clinic file editing |
| I can edit the clinic file through a form instead of raw text | Clinic file editing |
| A saved clinic file reaches every device, and a file with a part missing is refused with the part named | Clinic file editing |
| I can give the assistant a name and set my own quick replies in the clinic file | Clinic file editing |
| I see the queue at a glance as figures, and can export the tickets as a CSV file | Staff page and queue tools |
| Tickets, messages and the clinic file are kept in one shared store, and every page still works when the store cannot be reached | Tickets and clinic file shared across devices |
| The pages carry the brand's look in light and dark | Layout, themes and accessibility |

## Out of scope

| Item | Why |
| --- | --- |
| Booking into a diary, with a slot confirmed by the chat | The clinic file says booking is confirmed by the team. The chat passes the request on and never confirms a slot |
| Patient accounts and logins | A patient should be able to ask at once. A browser is recognised by a private key it holds, not by a sign-in |
| Named staff accounts | There is one shared desk code for the clinic |
| A sign-in in front of the staff page | The brief's check opens /staff directly. Only the list from every device asks for the desk code |
| Judging how serious a symptom is | The brief forbids medical advice. The fixed emergency and symptom rules stay; a dentist decides |
| Messages sent to patients by text message or WhatsApp from the staff page | It needs a business account and the patient's consent |
| Reports for the owner beyond the figures and the CSV export | Not asked for |
| An offline install | A chat that cannot reach the clinic is of no use to the patient |

## Success measures

The results of the last run of each are in [the report](REPORT.md#latest-results-on-the-live-link).

| Measure | Target | How it is measured |
| --- | --- | --- |
| The brief's nine "Done when" lines on the live link | all met, with the AI model and with it blocked | `tests/brief.mjs` |
| The brief's eight practice questions | each includes what it must and nothing it must not, in both runs | `tests/questions.mjs` |
| Bonus items | six of six | `tests/bonus.mjs` |
| A price, time, doctor or phone number in a reply that the clinic file does not hold | none | `tests/brief.mjs`, `tests/audit.mjs` |
| Sideways scroll from 375 px to 2560 px | none | `tests/widths.mjs` |
| Text contrast in light and dark | readable against the background a person really sees | `tests/contrast-true.mjs` |
| A different clinic file | answers come from the new file only | `tests/other-clinic.cjs` |

## Assumptions

- "That patient's chat" in the brief is the chat in the browser where the question was asked.
- "Within 2 working hours" is said in those words. The chat does not work out a clock time.
- An emergency is a message about the things the file's emergency rule names, or the common emergency words,
  in any of the three languages.
- A reply to a discount or rule-change message opens an answered ticket. A medical reply opens a needs-human
  ticket, so the team can call to book. The brief names no status for these.
- A field missing from the clinic file leads to the not-sure reply for that question, never to a failure.
- The AI model is optional. It may only reword an answer the rules already found, and it is never sent the
  patient's message.

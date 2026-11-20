# FrontDesk AI: review against the leading products

Reviewed 20 to 21 Nov 2026 (IST) against https://frontdesk-ai-xi-gray.vercel.app and /staff, the code in
`frontdesk-ai` (core.js, chat.js, staff/staff.js, api/chat.js) and `PRD.md`. No code was changed. Six test
questions were asked on the live site; their tickets live only in the reviewer's own browser storage. The
probe script and its output are in the session scratchpad (`ind/probe.mjs`, `ind/probe-out.txt`).

A caution on the sources: almost every page about this category is written by a vendor. What follows
reports what vendors say they offer, not tested results, and quotes no vendor's performance figures.

## 1. What the category offers as standard

| Capability | What the sources say | Source |
| --- | --- | --- |
| Always-on answers | Calls and messages answered at any hour, including nights, weekends and holidays | [JustCall dental](https://justcall.io/product/ai-voice-agent/dental/), [myaifrontdesk](https://www.myaifrontdesk.com/blogs/boost-your-dental-clinic-s-efficiency-with-a-virtual-receptionist-500f0) |
| Emergency triage | Flags words such as pain or swelling and routes to the on-call dentist; gives the emergency line first | [JustCall dental](https://justcall.io/product/ai-voice-agent/dental/), [Hyperleap clinics guide](https://hyperleap.ai/blog/whatsapp-clinics-hospitals-india-patient-communication) |
| Intake before handoff | "Collects the basics": name, contact, reason for visit, preferred doctor | [Hyperleap clinics guide](https://hyperleap.ai/blog/whatsapp-clinics-hospitals-india-patient-communication) |
| Booking | Either a request routed to staff who confirm by phone, or live slot picking from the diary ("Patients only see available slots") | [Hyperleap dental](https://hyperleap.ai/blog/whatsapp-dental-clinics-india), [Whautomate](https://whautomate.com/pages/whatsapp-appointment-bot-for-dental-clinics) |
| Reschedule and cancel | Shares the cancellation policy; better products let the patient move or cancel in the chat | [Hyperleap dental](https://hyperleap.ai/blog/whatsapp-dental-clinics-india), [Whautomate](https://whautomate.com/pages/whatsapp-appointment-bot-for-dental-clinics) |
| Reminders | WhatsApp reminders 24 hours and 1 hour (or 2 hours) before, with confirm or reschedule buttons | [Whautomate](https://whautomate.com/pages/whatsapp-appointment-bot-for-dental-clinics), [NXC WABA guide](https://waba.nxccontrols.in/blog/whatsapp-business-api-for-healthcare-appointment-reminders-patient-support-more-2026) |
| Booking confirmation content | Doctor's name, date, time, clinic address and a Google Maps link | [NXC WABA guide](https://waba.nxccontrols.in/blog/whatsapp-business-api-for-healthcare-appointment-reminders-patient-support-more-2026) |
| Prices and insurance | General cost ranges, accepted insurers, payment plans; complex cases go to the billing team | [Hyperleap dental](https://hyperleap.ai/blog/whatsapp-dental-clinics-india), [JustCall dental](https://justcall.io/product/ai-voice-agent/dental/) |
| Handoff with context | Transfer to staff "with context" so the patient does not repeat themselves; shared team inbox | [JustCall dental](https://justcall.io/product/ai-voice-agent/dental/), [Whautomate](https://whautomate.com/pages/whatsapp-appointment-bot-for-dental-clinics) |
| Languages | English and Spanish plus a dozen others (US); Hindi, Tamil, Telugu, Kannada, Bengali, Marathi named for India; Practo runs consultations in 15 Indian languages | [JustCall dental](https://justcall.io/product/ai-voice-agent/dental/), [Practo, YourStory](https://yourstory.com/2021/04/practo-launches-online-consultations-15-indian-languages) |
| Channels | Website, WhatsApp, Instagram, Messenger, SMS from one system | [Hyperleap clinics guide](https://hyperleap.ai/blog/whatsapp-clinics-hospitals-india-patient-communication), [Contra case](https://contra.com/community/D5XOPS7l-ai-receptionist-automation-for-dental-clinic) |
| Practice software | Two-way sync with Dentrix, Eaglesoft, Open Dental (US) | [JustCall dental](https://justcall.io/product/ai-voice-agent/dental/) |
| Other extras | Intake form link, aftercare notes, review link for happy patients, payment links in chat | [Hyperleap dental](https://hyperleap.ai/blog/whatsapp-dental-clinics-india), [Whautomate](https://whautomate.com/pages/whatsapp-appointment-bot-for-dental-clinics) |
| Privacy | US: HIPAA and a business associate agreement on higher plans. India: plain-language notice before collection, consent that can be withdrawn | [JustCall dental](https://justcall.io/product/ai-voice-agent/dental/), [AM Legals](https://amlegals.com/health-data-and-the-dpdp-act-a-practical-guide/), [KPMG India](https://kpmg.com/in/en/insights/2025/12/the-privacy-prescription-impact-of-dpdp-act-and-rules-in-healthcare-and-life-sciences-sector.html) |

India context: BotPenguin's case study of an Indian dental centre runs a WhatsApp bot and a website bot
side by side ([BotPenguin](https://botpenguin.com/case-studies/happy-family-dental-care)); WhatsApp is the
channel Indian clinic owners expect.

## 2. What we already do (seen on the live site)

| Expectation | FrontDesk AI | Evidence |
| --- | --- | --- |
| Instant answers at any hour, only from the clinic file | Yes, with the model and without it ("Answered from the clinic file") | Probe: "Can I pay online?" gave the file's payment line with the label |
| Emergency first | Emergency number first and an urgent ticket | core.js `emergency` template; graded tests in tests/brief.mjs |
| Will not guess; hands to a human | "I am not sure about that, and I will not guess..." and a needs human ticket | Probe: Invisalign and "Do you have any reviews?" both opened needs human tickets |
| No medical advice; offers to book | Yes | core.js `medical` template |
| Prices, insurance, payment, EMI, policies | Yes, from the file | core.js templates; clinic-data.json |
| Booking request with a time | Ticket with "requested tomorrow 5:00 pm", callback promised, policy quoted | Probe: booking question; staff card shows the time |
| Reschedule and cancel policy | Yes, quoted from the file | Probe: "Can I reschedule my appointment?" |
| Hindi and Telugu | Selector plus automatic detection, translated templates | index.html `#lang`; core.js `hi` and `te` |
| Quick-reply buttons | Six: Timings, Prices, Doctors, Insurance, Location, Book a visit | Probe |
| Open now / closed now and after-hours note | Yes | Probe: "Open now" chip; chat.js `closedNote` |
| Continue on WhatsApp | Link beside each handed-off reply | Probe |
| Voice | Listen button on replies; microphone where the browser supports it | Probe; index.html `#mic` |
| Shared inbox for staff | /staff: ticket queue by priority, age ("just now"), counts, a "To call back" list, reply and close | Probe of /staff |
| Owner edits the clinic data | "Clinic file" editor with save and reset | Probe of /staff |
| Refuses discounts and "ignore your rules" | Yes | core.js `refuse` |
| Phone width, light and dark | Yes | tests/widths.mjs, tests/contrast.mjs (per CHECKPOINT.md) |

On the standard list, the chat itself is at or above what the WhatsApp booking bots offer for answering and
handing off. The gaps are in what happens after the handoff.

## 3. What we lack

| Gap | Seen where | Note |
| --- | --- | --- |
| G-A. No contact details on a ticket | Every needs human and booking reply promises "someone will call you back", but the chat page has only `#lang` and `#chat-input`; the ticket holds a patient code (for example P5YZEE), no name or phone | The leading products collect name and contact before the handoff. Today the staff member has no number to call |
| G-B. Tickets live in one browser | core.js stores tickets in localStorage; PRD 2.2 puts a server store in phase 2 | Staff on another device see nothing. Every product in section 1 has a shared inbox. Grading works because the tester uses one browser |
| G-C. No real slot booking or diary | PRD 2.2, by design | The bot must not invent a slot; the file says book on phone or WhatsApp |
| G-D. No reminders or confirmations sent out | PRD 2.4 parked | Needs a WhatsApp Business account, approved templates and consent |
| G-E. No directions link | Probe: "Is there a Google Maps link?" got "We don't have a Google Maps link to share" | Confirmations in the category carry a maps link |
| G-F. No privacy notice | Fine print speaks of answers and the emergency number only | Indian guidance: plain-language notice before collection |
| G-G. No owner summary | Staff page has counts, not "what the bot could not answer" | Owners use unanswered questions to improve the file |
| G-H. No answer feedback or review link | Not present | Common extra; the file holds no review link, so a review link cannot be added without inventing one |
| G-I. One channel only | Website; WhatsApp is a hand-off link | WhatsApp, Instagram and SMS are standard abroad and in India |
| G-J. No staff sign-in | /staff is open | Fine for the competition (PRD 2.2); not for a real clinic |

## 4. Which gaps to close today, ranked

Risk rule from the brief: anything that changes a `.bot-msg` text, a status, a selector or the first
visit is risky. chat.js already puts extras beside the reply, never inside it; every item below keeps to that.

| Rank | Change | Value | Effort | Risk to graded items | Guard rails |
| --- | --- | --- | --- | --- | --- |
| 1 | **Optional call-back details (G-A).** Beside a needs human or booking reply, a small "Name and phone for the call-back (optional)" form, saved onto that ticket and shown on the staff card and in "To call back" | High: makes the promise in the reply real | Small (one form, two fields, one ticket field) | Low | Outside `.bot-msg`; never takes focus from `#chat-input`; not shown on first visit; no change to status or reply text; ids that do not clash with the graded ones; Enter in the form must not submit `#chat-form` |
| 2 | **Directions link (G-E).** "Open in Maps" in the "Find us" side card, a Google Maps search URL built from `clinic.address` | Medium | Tiny | Very low | Side card only, not in replies; built from the file's address so nothing is invented, and it follows a staff edit of the address |
| 3 | **Privacy line (G-F).** One sentence in the fine print: messages are kept in this browser and shared with the clinic team only; do not type medical records | Medium for an owner, low for marks | Tiny, in three languages | Very low | Fine print only; keep the emergency number first in the fine print; no price, medicine or percentage in the text |
| 4 | **"Could not answer" list (G-G).** On /staff, the needs human questions grouped, with a link to the Clinic file editor | Medium: feeds bonus item 4 in practice | Small | Low | A separate section; must not use `data-ticket`, `#ticket-queue` or `data-status`, so the tester's ticket count stays exact |
| 5 | **WhatsApp reply from the staff card (G-D, partly).** When item 1 has a phone, a "Send on WhatsApp" link on the card (wa.me with the staff reply as text) in addition to the in-chat reply | Medium | Small, depends on item 1 | Low | `#staff-send` keeps its present behaviour and status change; the new link is extra and opens only on click |

Not today (value high but effort or risk too great before 2:00 pm): a server ticket store (G-B), slot
booking against a diary (G-C), outbound reminders (G-D in full), new channels (G-I), staff sign-in (G-J).
Each changes where tickets live or puts something in front of a graded element. They belong in phase 2 of
the PRD, where G-B is already listed (Q-3). A status filter on the staff queue was also considered and left
out: if it ever defaults to anything but "all", it hides `[data-ticket]` rows from the tester.

## 5. After any change

Rerun live, with the key and with the API blocked: tests/brief.mjs, tests/questions.mjs, tests/bonus.mjs,
tests/widths.mjs at 375 px and tests/contrast.mjs in both themes, and check that the last `.bot-msg`
text for the 8 sample questions is unchanged.

## Sources

- JustCall, AI voice agent for dental: https://justcall.io/product/ai-voice-agent/dental/
- Hyperleap, dental chatbot use cases: https://hyperleap.ai/blog/whatsapp-dental-clinics-india
- Hyperleap, WhatsApp for clinics and hospitals: https://hyperleap.ai/blog/whatsapp-clinics-hospitals-india-patient-communication
- Whautomate, WhatsApp appointment bot for dental clinics: https://whautomate.com/pages/whatsapp-appointment-bot-for-dental-clinics
- NXC Controls, WhatsApp Business API for healthcare: https://waba.nxccontrols.in/blog/whatsapp-business-api-for-healthcare-appointment-reminders-patient-support-more-2026
- Contra, AI receptionist automation for a dental clinic: https://contra.com/community/D5XOPS7l-ai-receptionist-automation-for-dental-clinic
- myaifrontdesk, virtual receptionist for dental clinics: https://www.myaifrontdesk.com/blogs/boost-your-dental-clinic-s-efficiency-with-a-virtual-receptionist-500f0
- BotPenguin, Happy Family Dental Centre case study: https://botpenguin.com/case-studies/happy-family-dental-care
- YourStory, Practo consultations in 15 Indian languages: https://yourstory.com/2021/04/practo-launches-online-consultations-15-indian-languages
- AM Legals, health data and the DPDP Act: https://amlegals.com/health-data-and-the-dpdp-act-a-practical-guide/
- KPMG India, DPDP Act and Rules in healthcare: https://kpmg.com/in/en/insights/2025/12/the-privacy-prescription-impact-of-dpdp-act-and-rules-in-healthcare-and-life-sciences-sector.html

## Status after fixes (21 Nov 2026)

Fixed in the code repository commit "Review fixes: model cannot be steered, stricter guard, engine gaps, call-back details, design", then every test rerun on the live link: brief 28/28 with the key and 30/30 with the model blocked, questions 16/16, bonus 15/15 both ways, widths 20/20, contrast clean, engine ALL PASS (files in `../evidence/`).

Fixed (the top 5 of section 4):

- 1, G-A. Optional name and phone for the call-back beside a needs human or booking reply, saved on that
  ticket and shown on the staff card and in "To call back".
- 2, G-E. Directions link in the "Find us" card, built from the file's address.
- 3, G-F. Privacy line in the fine print, in English, Hindi and Telugu.
- 4, G-G. "Could not answer" list on /staff, outside `#ticket-queue`.
- 5, G-D partly. Call and WhatsApp links on the staff card when the patient left a phone.

Open, phase 2 or not built:

- G-B. Server ticket store: phase 2 (PRD Q-3); it changes where tickets live, which the graded staff reply
  depends on.
- G-C. Real slot booking: not built; the bot must never invent a slot, and the file says to book on the
  phone or WhatsApp.
- G-D in full. Outbound reminders and confirmations: not built; they need a WhatsApp Business account,
  approved templates and consent.
- G-H. Answer feedback or review link: not built; the clinic file holds no review link, so one cannot be
  added without inventing it.
- G-I. More channels: not built; the website is the graded channel, WhatsApp is a hand-off link.
- G-J. Staff sign-in: not built; a sign-in screen would stand between a fresh-browser tester and /staff.

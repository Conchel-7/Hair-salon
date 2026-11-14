# What changed from the old version (blueprint 17)

## Fewer, simpler modules: 45 → 32

| Old | New |
|---|---|
| 13 separate WhatsApp "send" modules, many with copy-pasted text | 9, each with one clear job |
| 9 Data Store writes, each with its own (different!) price formula | 1 price formula in module 7 |
| Opening-hours rules in 3 copies (filter + 2 messages) | 1 copy in module 8 |
| Settings (hours, bank details, email) scattered through modules | All in module 3 |
| A 1,700-word AI prompt with statuses, overwrite rules and exceptions | A shorter prompt: understand the message, extract details, write a friendly answer |
| No error handling | 5 small safety nets so customers never get silence |

## Bugs fixed

1. **Wrong prices were being saved.** The price formulas had hidden line breaks before some service
   names (`"\nWig installation"`), so they never matched. Wig installation and Hair treatment
   bookings were saved with **R0**, and other paths kept an old price.
2. **Rescheduling skipped the booking rules.** A customer could move an appointment to a Sunday,
   to 22:00, or to a date in the past. The same rules now apply to new bookings and reschedules.
3. **Moving an appointment by an hour said "unavailable".** The calendar search found the
   customer's *own* appointment and treated it as a clash. Their own event is now ignored.
4. **Customers could never book again after their appointment.** The old appointment stayed
   "active" forever. Appointments in the past now no longer count.
5. **Customers waiting to pay got the same canned reply to every message** – the AI never ran,
   so "what's your address?" got "please send your payment screenshot". Now the AI answers.
6. **Any photo got "I already have your proof of payment"** – even from someone who never booked.
   The reply now depends on the situation.
7. **Voice notes, stickers and videos got no reply at all.** They now get "please type your message".
8. **If the AI failed or returned bad JSON, the customer got silence**, and after 3 errors in a row
   Make switches the scenario off. Both cases now send a polite "please send that again" message.
9. **A "yes" could cancel an appointment by accident.** Cancelling now only happens when the bot's
   previous message was the cancellation question *and* the customer agreed.

## Easier for customers

- Details can be sent **bit by bit** ("Box braids please" … "Saturday at 9"), not all in one message.
- Natural dates and times work: "tomorrow", "next Friday", "15/11", "2pm". No 4-digit year needed.
- Clear reasons when a time doesn't work, e.g. *"Box braids takes about 5 hours and we close at 17:00,
  so the latest start time is 12:00."*
- Dates are shown as "Saturday 21 November 2026" instead of "2026-11-21".
- The "Hi! 👋 I'm Maia…" intro is no longer repeated on every message.
- Proof of payment can be a **PDF** as well as a photo.

## Security

The approval link and token used to be written straight into the email module. They are now in
module 3 and **not** stored in this public repo. Because the old token was in a shared file,
consider changing `approval_token` (in module 3 *and* in your approval scenario) to something
long and random.

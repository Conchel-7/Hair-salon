# Test checklist (v5)

Run every test from your own WhatsApp before customers use the bot. Use dates next week (not
Sunday). If a test fails, open the run in Make's **History**: module **6** shows what the AI
understood and module **8** shows what Make decided. To start from a clean slate, send
`delete my data` (or delete your number's record in the Data Store).

Tick each box. Do not launch until every box is ticked.

## A. Instructions and price list

| # | Send | Expected | ✓ |
|---|---|---|---|
| 1 | `Hi` | Welcome, 10 services with price / time / latest start, "send ONE message with…", example, privacy note | ☐ |
| 2 | `How much is hair colouring?` | From R650, about 3 hours, latest start 14:00 | ☐ |
| 3 | `Main menu` | Same welcome as test 1 | ☐ |
| 4 | `asdfgh` | "Sorry, I did not quite understand… Send *menu*" (or a polite AI reply) | ☐ |
| 5 | A voice note | "I can only read typed messages" | ☐ |

## B. Booking and the YES step

| # | Send | Expected | ✓ |
|---|---|---|---|
| 6 | `Thando, Make-up, next Tuesday, 10:00` | "Please check your booking…" with all details + "Reply YES to confirm" | ☐ |
| 7 | `yes` | "Booking request received", stylist name, bank details. PENDING event in the stylist's calendar | ☐ |
| 8 | `Can I book another one?` | "You already have an appointment…" | ☐ |

## C. Booking rules (send `cancel`, `yes` first to free your number)

| # | Send | Expected | ✓ |
|---|---|---|---|
| 9 | `Thando, Hair colouring, next Tuesday, 15:00` | "latest start time is 14:00" | ☐ |
| 10 | `Thando, Make-up, next Tuesday, 10:15` | "on the hour or half hour" | ☐ |
| 11 | `Thando, Resizing, next Tuesday, 07:30` | "We open at 08:00" | ☐ |
| 12 | `Thando, Make-up, Sunday, 10:00` | Closed on Sundays | ☐ |
| 13 | `Thando, Make-up, 25 December, 10:00` | Salon closed that day | ☐ |
| 14 | `Wig installation` → `Thando` → `next Tuesday at 10` | Asks only for what's missing, then the YES question | ☐ |
| 15 | At the YES question, send `make it 11:00` instead | A new YES question with 11:00 | ☐ |

## D. Several stylists

| # | Do | Expected | ✓ |
|---|---|---|---|
| 16 | Book Make-up next Wednesday 10:00 (+ `yes`) | Stylist 1 | ☐ |
| 17 | From a second phone, same time (+ `yes`) | Stylist 2 | ☐ |
| 18 | Add an `OFF` event next Wednesday 10:00–11:00 in stylists 3 and 4's calendars; book the same time from a third phone | "all our stylists are booked" | ☐ |
| 19 | From the first phone's slot: book Make-up next Wednesday 11:00 from another phone | Stylist 1 again (short jobs free the stylist) | ☐ |

## E. Payment and approval

| # | Do | Expected | ✓ |
|---|---|---|---|
| 20 | Send a photo after booking | "received your proof of payment"; email with photo + APPROVE / REJECT | ☐ |
| 21 | Send another photo | "I already have your proof of payment" | ☐ |
| 22 | Tap REJECT | Browser "REJECTED"; WhatsApp "couldn't verify your payment" | ☐ |
| 23 | Tap APPROVE in the **same** email | Browser "Nothing changed" (the link is no longer valid) | ☐ |
| 24 | Send a new photo, tap APPROVE in the **new** email | Browser "APPROVED"; WhatsApp "booking confirmed"; calendar CONFIRMED | ☐ |
| 25 | Change one letter of `code=` in the link and open it | "Nothing changed" | ☐ |

## F. Reschedule, cancel, privacy

| # | Send | Expected | ✓ |
|---|---|---|---|
| 26 | `Can I move my appointment?` | Asks for the new date and time | ☐ |
| 27 | A new day and time | "Your appointment has been moved"; old event gone, new one created | ☐ |
| 28 | `cancel` → `no` | Appointment stays | ☐ |
| 29 | `delete my data` (while booked) | "You have an upcoming appointment… keep your details until then" | ☐ |
| 30 | `cancel` → `yes` | "has been cancelled"; event deleted | ☐ |
| 31 | `delete my data` | "Your details have been deleted"; your Data Store record is gone | ☐ |

## G. Break it on purpose (proves the safety nets)

Do these one at a time and **undo each change afterwards**.

| # | Break | Send | Expected | ✓ |
|---|---|---|---|---|
| 32 | Module 3: change one letter in `stylist_2_calendar` | Book a time when stylist 1 is busy | Customer gets the apology; you get a "⚠ Booking bot problem" email naming stylist 2's calendar | ☐ |
| 33 | Gmail connection: disconnect it in Make → Connections | Send a payment photo | Customer is asked to send it again / email it | ☐ |
| 34 | Make 4 bookings fail in a row (repeat test 32 four times) | — | The scenario is **still ON** afterwards | ☐ |

## H. Clean-up scenario

| # | Do | Expected | ✓ |
|---|---|---|---|
| 35 | In the Data Store, set a test record's `last_seen` to `2020-01-01 10:00` and `date` to `2020-01-01`. Run the clean-up scenario once | That record is deleted, all others stay | ☐ |

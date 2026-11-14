# Test checklist

Run these from your own WhatsApp before switching the scenario on for real customers.
Use a test day next week. Tick each one off. If one fails, open the run in Make's
*History* and look at module **6** (what the AI understood) and module **8** (what Make decided).

To start again from a clean slate, delete your phone number's record in the Data Store.

## Booking

| # | Send | Expected |
|---|---|---|
| 1 | `Hi` | Welcome, list of 4 services with prices, asks what you'd like |
| 2 | `What time do you close on Saturday?` | 17:00 answer, no booking started |
| 3 | `I'd like box braids` | Asks for your name, date and time |
| 4 | `I'm Thando, next Tuesday at 9` | "Booking request received", correct date, 09:00 – 14:00, R700, bank details. A `PENDING` event appears in Google Calendar |
| 5 | `Can I also book a wig install?` | "You already have an appointment…" |

## Booking rules (start from a clean record)

| # | Send | Expected |
|---|---|---|
| 6 | `Knotless braids on Sunday at 10, name Lebo` | Closed on Sundays |
| 7 | `Box braids tomorrow at 15:00, I'm Lebo` | Latest start time is 12:00 |
| 8 | `Hair treatment tomorrow at 7am, Lebo` | We open at 08:00 |
| 9 | Book a time that already has an event in the calendar | "That time is already taken" |

## Payment

| # | Send | Expected |
|---|---|---|
| 10 | A photo (after test 4) | "Thank you… sent it to the salon". Email with the photo + Approve/Reject links arrives |
| 11 | Another photo | "I already have your proof of payment" |
| 12 | A voice note | "I can only read typed messages" |

## Reschedule and cancel

| # | Send | Expected |
|---|---|---|
| 13 | `Can I change my appointment?` | Asks for the new date and time |
| 14 | `Make it an hour later` (or the same day, 1 hour later) | "Your appointment has been moved". Old event gone, new one in the calendar |
| 15 | `Move it to Sunday` | Closed on Sundays, appointment unchanged |
| 16 | `I want to cancel` | "Are you sure… Reply YES to cancel" |
| 17 | `No` | Appointment stays |
| 18 | `Cancel please` then `Yes` | "Has been cancelled". Event deleted from the calendar |
| 19 | `Yes` again | Normal answer – nothing else is cancelled |

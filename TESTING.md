# Test checklist (v3)

Run these from your own WhatsApp before customers use the bot. Pick dates next week (not Sunday).
If one fails, open the run in Make's *History*: module **6** shows what the AI understood and
module **8** shows what Make decided. To start from a clean slate, delete your number's record
in the Data Store.

## Instructions and price list

| # | Send | Expected |
|---|---|---|
| 1 | `Hi` | Welcome + all 10 services with price, time and latest start + "send ONE message with…" + example |
| 2 | `How much is hair colouring?` | From R650, about 3 hours, latest start 14:00 |
| 3 | `Main menu` | The same welcome / price list |

## Booking rules

| # | Send | Expected |
|---|---|---|
| 4 | `Thando, Hair colouring, next Tuesday, 15:00` | "latest start time is 14:00" |
| 5 | `Thando, Wig making, next Tuesday, 13:30` | "latest start time is 13:00" |
| 6 | `Thando, Make-up, next Tuesday, 10:15` | "on the hour or half hour" |
| 7 | `Thando, Resizing, next Tuesday, 07:30` | "We open at 08:00" |
| 8 | `Thando, Make-up, Sunday, 10:00` | Closed on Sundays |
| 9 | `Thando, Make-up, 25 December, 10:00` | Closed that day |
| 10 | `Wig installation` then `Thando` then `next Tuesday at 10` | Asks only for what's missing, then books |

## Several stylists (`stylists` = 4)

| # | Do | Expected |
|---|---|---|
| 11 | In Google Calendar, add 3 events next Wednesday 10:00–12:00. Then book `Make-up, next Wednesday, 10:00` | Booked (4th stylist free) |
| 12 | From a second phone (or after cancelling), book the same time again | "all our stylists are booked" |

## Payment and approval

| # | Do | Expected |
|---|---|---|
| 13 | Send a photo after booking | "received your proof of payment"; email arrives with APPROVE / REJECT |
| 14 | Tap APPROVE | Browser "APPROVED"; WhatsApp "booking confirmed"; calendar title CONFIRMED |
| 15 | Tap APPROVE again | Browser "Nothing changed" |

## Reschedule and cancel

| # | Send | Expected |
|---|---|---|
| 16 | `Can I move my appointment?` | Asks for the new date and time |
| 17 | A new day and time | "Your appointment has been moved"; old event gone, new one in the calendar |
| 18 | `cancel` then `no` | Appointment stays |
| 19 | `cancel` then `yes` | "has been cancelled"; event deleted |

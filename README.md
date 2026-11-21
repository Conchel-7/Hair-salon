# Hair Salon – WhatsApp Booking Bot (Make.com)

> **Setting it up? Start with [LAUNCH.md](LAUNCH.md)**, then [TESTING.md](TESTING.md).

A WhatsApp assistant that lets salon customers **ask questions, book, reschedule and cancel
appointments, and send proof of payment** — built in Make.com with Claude, Google Calendar,
a Make Data Store and Gmail.

The scenario blueprint is in [`make/salon-booking.blueprint.json`](make/salon-booking.blueprint.json).
The copy in this repo has personal details replaced with placeholders (this repo is public) –
see [Setup](#setup).

---

## How it works (the big idea)

The old version had 45 modules because the AI was asked to *decide everything* (prices,
statuses, what to keep, what to overwrite), and then many routers double-checked it.

The new version splits the job in two:

| The **AI** only does language | **Make** does all the rules |
|---|---|
| Works out what the customer wants: `book`, `reschedule`, `cancel_request`, `cancel_confirm` or `chat` | Prices and durations |
| Pulls out name, service, date, time – understands "tomorrow at 2pm" | Opening hours, closed days, notice period |
| Writes friendly answers to questions | Calendar availability, creating / moving / deleting events |
| | Every message that confirms, moves or cancels something |

Because Make (not the AI) writes every message that *promises* something, the bot can't
accidentally tell a customer they're booked when they're not.

### The flow (32 main modules + 58 safety-net modules)

```
1 WhatsApp message in
2 Get customer record ── 3 SALON SETTINGS ── 4 Router: what kind of message?
│
├── TEXT ─ 5 AI: understand ─ 6 Read AI answer ─ 7 Booking details ─ 8 Booking check ─ 9 Save conversation ─ 10 Router
│                                                                                           │
│      ├── Ready to check calendar ─ 11/30/31/32 Is stylist 1/2/3/4 free? ─ 33 Pick first free stylist ─ 13 Router
│      │        ├── Free  ─ 14 Create event ─ 15 Save booking ─ 16 Send details ─ 17 Remove old event (reschedule only)
│      │        └── Taken ─ 18 Send "slot taken"
│      ├── Cancel confirmed ─ 19 Delete event ─ 20 Delete record ─ 21 Send "cancelled"
│      └── Everything else ─ 22 Send reply
│
├── PROOF OF PAYMENT (photo/PDF while awaiting payment)
│        23 Download ─ 24 Email owner (approve/reject links) ─ 25 Mark "payment_review" ─ 26 Send "thank you"
│
└── ANYTHING ELSE (voice notes, stickers, random photos) ─ 27 Send "please type"

Safety nets (error handlers): 50 AI is down → polite retry message · 51/52 AI answer unreadable → polite retry
53 old event already gone → carry on · 54 event already deleted → carry on
```

### What gets saved per customer (Data Store, key = phone number)

| Field | Meaning |
|---|---|
| `name`, `service`, `date`, `time`, `price` | The booking (or the details collected so far) |
| `status` | `draft` → `awaiting_payment` → `payment_review` → `confirmed` *(set by the approval scenario)* |
| `add calender_event-id` | The Google Calendar event (empty while still a draft) |
| `reply` | The last message the bot sent – gives the AI conversation context and makes "YES" to cancel safe |
| `payment status`, `is_complete` | No longer used – can be deleted later |

---

## Setup

1. **Back up first.** In Make, open your current scenario → `…` → *Export Blueprint*, keep that file.
   Then **turn the old scenario OFF** (only one scenario can use the WhatsApp webhook).
2. Create a new scenario → `…` → *Import Blueprint* → choose the file.
   - Use the private file you were sent (it has your real calendar, emails and approval link) **or**
     use the repo file and replace the placeholders: `YOUR_GOOGLE_CALENDAR_ID` (modules 11, 14, 17, 19)
     and the owner email / approval link / token in module **3 SALON SETTINGS**.
3. Click each module with a red warning (if any) and re-select the connection / data store.
4. Open module **6 Read AI answer**: it has no data structure on purpose. Make shows the AI fields in
   grey until the scenario has run once – that's normal.
5. Run the [test checklist](TESTING.md), then switch the scenario ON.

---

## Day-to-day maintenance

**Almost everything lives in module 3 – `SALON SETTINGS (edit here)`:**

| Setting | Example | Used for |
|---|---|---|
| `salon_name`, `bot_name` | Conchel's Hair Salon, Maia | Greeting / AI personality |
| `services` | `• *Wig installation* – R450 · 1½ hrs · start by 15:30` | The price list customers see |
| `opening_time`, `closing_time` | 08:00, 17:00 | Booking rules |
| `closed_days` | `Sunday` or `Sunday, Monday` | Weekly closed days |
| `closed_dates` | `2026-12-25, 2026-12-26` | Public holidays / days off (YYYY-MM-DD) |
| `stylist_1_name` … `stylist_4_name` | Lerato | Stylist names customers are told |
| `stylist_1_calendar` … `stylist_4_calendar` | `abc…@group.calendar.google.com` | Each stylist's own Google Calendar |
| `days_ahead`, `notice_hours` | 60, 2 | How far ahead / how soon people can book |
| `extra_info` | Address, parking, cancellation policy… | Extra facts the AI may answer with |
| `payment_details` | Bank details | Sent after a booking request |
| `owner_email` | | Payment emails + customer help |
| `approval_link`, `approval_token` | | Approve / reject links in the email |

### Services, prices and times (estimates – please check them)

| Service | Price | Takes | Latest start |
|---|---|---|---|
| Wig installation | R450 | 1½ hrs | 15:30 |
| Direct install | R550 | 2½ hrs | 14:30 |
| Customization | R250 | 1½ hrs | 15:30 |
| Styling & curling | R250 | 1 hr | 16:00 |
| Make-up | R450 | 1 hr | 16:00 |
| Hair wash & cornrows | R250 | 1½ hrs | 15:30 |
| Wig treatment | R300 | 1½ hrs | 15:30 |
| Hair colouring | from R650 | 3 hrs | 14:00 |
| Wig making | from R600 | 4 hrs | 13:00 |
| Resizing | R150 | 30 min | 16:30 |

Prices are typical South African mall-salon prices for 2026 and durations are estimates –
change them to the salon's real numbers before going live.

### Adding a service or changing a price / duration

1. Module **3** → `services`: the line customers see (price, time, latest start).
2. Module **7 Booking details** → the same service name, spelled exactly the same, in the
   `price`, `minutes`, `minus_minutes` (same number with a minus) and `duration` formulas.

Latest start = closing time minus the duration (e.g. 17:00 − 3 hrs = 14:00). The bot enforces it
automatically; the `services` text just tells customers in advance.

### Stylists – one Google Calendar each

Each booking goes to the **first stylist who is free** for the whole appointment, checked in
their own calendar. Short jobs free a stylist up straight away for the next customer.

**Setting up the calendars (once):**
1. Stylist 1 uses the existing salon calendar.
2. In Google Calendar (computer) → *Other calendars* → **+** → *Create new calendar* →
   name it after the stylist. Repeat for stylists 2, 3 and 4. Give each a different colour.
3. For each new calendar: *Settings and sharing* → scroll to **Integrate calendar** → copy the
   **Calendar ID** (ends in `@group.calendar.google.com`).
4. Paste the IDs and the stylists' names into module **3** (`stylist_2_calendar`, …).
5. Data Store → your booking data structure → add two **text** fields: `stylist` and `calendar`.

**Only 3 stylists?** Set `stylist_4_calendar` to the same ID as `stylist_1_calendar` and
`stylist_4_name` to stylist 1's name.

**Stylist off for a day or a few hours?** Add an event called `OFF` in *their* calendar for that
time. Whole salon closed? Add the date to `closed_dates`.

---

## Known limits

- **Customers can't choose a stylist yet** – they get the first one who is free.
- Keep personal events out of the stylists' calendars: anything in a stylist's calendar blocks them.
- **Moving an appointment to a time that overlaps its own current slot** may go to a different
  stylist (the customer's own booking makes their current stylist look busy).
- **Calendar titles:** new events are called `PENDING - name - service`. When a *confirmed*
  customer reschedules, the new event is called `CONFIRMED - …` only if your approval scenario
  writes `confirmed` into `status`.
- **WhatsApp 24-hour rule:** a confirmation sent more than 24 hours after the customer's last
  message needs an approved WhatsApp template.

---

## Payment approval scenario

File: [`make/payment-approval.blueprint.json`](make/payment-approval.blueprint.json) – a **second,
separate scenario** that runs when the owner taps APPROVE or REJECT in the payment email.

```
1 Link clicked (webhook) ─ 2 Get customer record (only with the correct token) ─ 3 Router
   ├── Approve ─ 4 Calendar: CONFIRMED ─ 5 Status "confirmed" ─ 6 WhatsApp "booking confirmed" ─ 7 Browser: "APPROVED"
   ├── Reject  ─ 8 Status back to "awaiting_payment" ─ 9 WhatsApp "please send proof again" ─ 10 Browser: "REJECTED"
   └── Already handled / not found ─ 11 Browser: "Nothing changed"
```

Setup:
1. Turn **off** any old approval scenario.
2. Import this blueprint into a **new** scenario.
3. Click module 1 → **Add** a new webhook → copy its address.
4. In the booking scenario, module **3 SALON SETTINGS** → paste it into `approval_link` → Save.
5. Turn the approval scenario **ON** (Immediately).

The token in module 2's filter must match `approval_token` in the booking scenario's module 3.
WhatsApp only delivers the confirmation if the customer messaged in the last 24 hours – if not,
the browser page tells the owner to contact the customer directly.

See [CHANGES.md](CHANGES.md) for the list of bugs fixed compared with the old version.

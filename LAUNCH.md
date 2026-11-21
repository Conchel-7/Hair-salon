# Launch guide (v5)

Three scenarios work together:

| Scenario | File | Runs when |
|---|---|---|
| **Booking** | `make/salon-booking.blueprint.json` | A customer sends a WhatsApp message |
| **Payment approval** | `make/payment-approval.blueprint.json` | The owner taps APPROVE / REJECT in the payment email |
| **Daily privacy clean-up** | `make/privacy-cleanup.blueprint.json` | Every night |

Do the steps in order. Each one says **why**.

---

## 1. Data Store fields (before importing anything)

Make → **Data structures** → the structure used by the booking data store → **Add item** for each:

| Field | Type |
|---|---|
| `stylist` | Text |
| `calendar` | Text |
| `last_message_id` | Text |
| `last_seen` | Text |

*Why:* the new version saves these. If a field is missing, saving fails and customers get the apology.

## 2. Stylist calendars

1. Stylist 1 uses the existing salon calendar.
2. Google Calendar (on a computer) → *Other calendars* → **+** → *Create new calendar* → name it after the
   stylist. Repeat for stylists 2–4 and give each a colour.
3. Each calendar → *Settings and sharing* → **Integrate calendar** → copy the **Calendar ID**.
4. Turn on **2-step verification** on the Google account and on the email inbox that receives payment proofs.

*Why:* each stylist's calendar is checked separately so short jobs free them up for the next customer.

## 3. Import the three scenarios

| Scenario | Import into | After importing |
|---|---|---|
| Booking v5 | Your existing booking scenario (keeps the WhatsApp webhook) | Module 3: paste the 3 calendar IDs, the stylist names and your approval webhook address into `approval_link`. Check `approval_token` is a long random word |
| Payment approval v3 | Your existing approval scenario | Module 1: choose your existing **Payment approval** webhook. The secret in the filters of modules 4 and 8 must be the same word as `approval_token` |
| Privacy clean-up | A **new** scenario | Schedule: **every day at 02:00** |

*Why the secret must match:* each email link carries a code made from the booking + the secret. The approval
scenario rebuilds the code and only accepts the link if they match – so a link only works for its own
booking, and only while the payment is in review.

## 4. Scenario settings (gear icon at the bottom of each scenario)

| Setting | Booking | Approval | Clean-up | Why |
|---|---|---|---|---|
| Sequential processing | **ON** | **ON** | off | No double bookings |
| Allow storing of incomplete executions | ON | ON | off | Lets you replay a failed run from History |
| Number of consecutive errors | 10 (already in the file) | 10 | 10 | Much harder for the bot to switch itself off |
| Data is confidential | **ON after testing** | ON after testing | ON | Hides customers' messages from Make's History (POPIA) |

## 5. Make notifications

Profile picture (bottom left) → **Profile → Notifications** → email on for **errors**, **warnings** and
**scenario deactivated**.

*Why:* if anything goes wrong that the safety nets can't handle, Make tells you.

## 6. Test

Work through every box in [TESTING.md](TESTING.md), including section **G – break it on purpose**.

## 7. Pilot

Run with **one salon for 1–2 weeks**. Every day:
1. Open each scenario's **History** and look for warnings.
2. Read every "⚠ Booking bot problem" email and fix the cause.
3. Ask the salon what customers said.

Go wider only after a full week with no unexplained warnings.

## 8. Before you sell it

| Item | How |
|---|---|
| **WhatsApp template** for confirmations sent after 24 hours | business.facebook.com → WhatsApp Manager → Message templates → Create → *Utility*, name `booking_confirmed`. Until it's approved, the approval page tells the owner when to contact the customer directly |
| **POPIA** | The salon registers an Information Officer with the Information Regulator. Keep the privacy note in the welcome message. Have the wording checked once |
| **Make plan** | About 15 operations per message. Messages per day × 15 × 30 must fit in the plan |
| **Backups** | After every working change: *… → Export Blueprint*, save with the date in the file name |

---

## What the safety nets do

Every module that talks to Google, WhatsApp, Gmail or the Data Store has one:

| What failed | Customer gets | Owner gets |
|---|---|---|
| Data Store, calendar, downloading the photo | "Sorry, something went wrong on our side… the salon has been told" | "⚠ Booking bot problem: …" email saying which step and which customer |
| A WhatsApp message | – (it can't be delivered) | The same alert email, so they can contact the customer |
| The payment email to the owner | "Please send it again in a few minutes, or email it to …" | – |
| The AI | "Sorry, I'm having a small technical problem…" | – |
| The approval link | – | The browser page says what happened and what to do |

In every case the bot **keeps running** for the next customer.

## What no file can promise

- Google, WhatsApp, Make or Anthropic can have outages. The bot then apologises and alerts you – it cannot
  book until they are back.
- The AI can still misread a message. The YES step means a wrong booking can't reach the calendar
  without the customer seeing it first.
- Things nobody has thought of will show up in real use. That is what the pilot is for.

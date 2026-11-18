# WhatsApp "I'm busy" assistant (Make.com)

When someone messages your WhatsApp Business number, this Make.com scenario:

1. Welcomes them and says you're busy but will get back to them.
2. Asks, one question at a time, for their **name → surname → email → what they need**.
   Their **cellphone number is captured automatically** from WhatsApp, so they aren't asked for it.
3. Thanks them.
4. **Emails you** their name, surname, cellphone, email and their message.

It runs **24/7**: every new person who messages gets an instant reply, day or night.

---

## Example chat

> **Client:** Hi, I saw your post about AI automations
> **Bot:** Hi 👋 Thanks for contacting **Conchel AI Automation**! I'm busy right now, but I'll get back to you as soon as I can. While you wait, what's your **first name**?
> **Client:** Thabo
> **Bot:** Nice to meet you, Thabo! What's your **surname**?
> **Client:** Mokoena
> **Bot:** Thanks! What's your **email address**?
> **Client:** thabo@company.co.za
> **Bot:** Great. Briefly, **what would you like help with?** (e.g. automating bookings, a WhatsApp bot, lead follow-ups…)
> **Client:** I want to automate replies to my customers' questions
> **Bot:** Thank you, Thabo! 🙏 I've got all your details and will be in touch soon.

**You receive an email like this:**

```
Subject: New WhatsApp lead: Thabo Mokoena

Name:       Thabo
Surname:    Mokoena
Cellphone:  +27821234567   (https://wa.me/27821234567)
Email:      thabo@company.co.za
Message:    I want to automate replies to my customers' questions
```

---

## Quick setup: import the ready-made scenario

The file `whatsapp-assistant.blueprint.json` contains the scenario (modules 1–15). The final **Gmail › Send an email** step must be added by hand after module 15, as described in Route F below.

1. **Create the data store first** ([Step 1](#step-1-create-the-data-store-the-bots-memory) below).
2. In Make: **Create a new scenario → ⋯ → Import Blueprint** → choose the file.
3. **Module 1 (New WhatsApp message):** pick your existing webhook, or create one with your WhatsApp connection.
4. **Run once**, then send a WhatsApp message to your business number from another phone.
5. **Module 2 (Message details):** this is the only place that reads the WhatsApp data. Check its 3 values against what module 1 received (click the bubble on module 1 to see the data):
   - `from` = sender's number, `{{1.messages[1].from}}`
   - `text` = message text, `{{1.messages[1].text.body}}`
   - `senderId` = your Phone number ID, `{{1.metadata.phone_number_id}}`. You can also paste the ID straight in from Meta → WhatsApp Manager → API setup.
   If the data is laid out differently, re-map these 3 here and do the same for the filter just before module 2 (message type = `text`). Every other module reads from module 2, so nothing else needs to change.
6. **Modules 3, 6, 8, 10, 12, 15 (data store):** select the `WhatsApp Leads` data store.
7. **Modules 5, 7, 9, 11, 13, 14 (Send a Message):** select your WhatsApp connection. Open one of them and check that the message text is filled in.
8. **Add the email step:** click the **+** after module 15, add **Gmail › Send an email**, and fill it in as described in Route F below.
9. **Save**, check that **Sequential processing** is ON in scenario settings, and switch the scenario **ON**.

The step-by-step build below explains what each part does, and you can follow it if you prefer to build the scenario by hand.

## What you need before you start

- A **Make.com** account (the free plan works to test it).
- A **WhatsApp Business Cloud API** number connected through Meta (business.facebook.com → WhatsApp Manager).
  - **Important:** a number on the Cloud API normally *can't* also be used in the WhatsApp Business phone app.
    Meta's **Coexistence** option lets you keep using the app on the same number. If you use it, check that
    messages reach Make. Some Make users report the Watch Events trigger not firing with Coexistence.
    If that happens, use a separate number for the bot.
- A **Gmail** connection in Make (to send the lead emails to you).

---

## Step 1: Create the Data store (the bot's memory)

The bot needs to remember where each person is in the conversation.

1. In Make, open **Data structures → Add data structure**. Name it `WhatsApp Lead`. Add these fields, all type **Text** except the last one:
   | Field | Type |
   |---|---|
   | `step` | Text |
   | `firstName` | Text |
   | `surname` | Text |
   | `email` | Text |
   | `inquiry` | Text |
   | `updatedAt` | Date |
2. Go to **Data stores → Add data store**. Name it `WhatsApp Leads` and select the `WhatsApp Lead` structure.
   Each record's **key** will be the client's WhatsApp number.

---

## Step 2: Build the scenario

Create a new scenario and add these modules in order. The numbers in brackets (e.g. `{{2.step}}`) refer to module numbers.
In Make you pick them from the mapping panel instead of typing them.

### Module 1: WhatsApp Business Cloud › Watch Events (trigger)
- Create the connection (log in with the Facebook account that is admin of your WhatsApp Business account).
- Create the webhook and subscribe it to **messages**.
- Click **Run once**, then send a test WhatsApp message to your business number. Make now knows the output fields.
  You'll use two of them:
  - **From**: the sender's phone number (e.g. `27821234567`). Below it's called `FROM`.
  - **Text › Body**: the text they typed. Below it's called `TEXT`.

**Filter after module 1** (click the line after the module → Set up a filter):
- Label: `Only text messages`
- Condition: message **Type** *Equal to* `text`
  (this ignores delivery receipts, read receipts, images and voice notes).

### Module 2: Data store › Get a record
- Data store: `WhatsApp Leads`
- Key: `FROM`
- Under **Advanced settings**, turn on **Continue the execution of the route even if the module finds no record** so that new people still get through.

### Module 3: Router
Add **5 routes**. Set each route's filter on the line coming out of the router:

| Route | Filter label | Condition(s) |
|---|---|---|
| A | New conversation | `{{2.step}}` *Does not exist* **OR** (`{{2.step}}` = `done` **AND** `{{2.updatedAt}}` *Earlier than* `{{addDays(now; -7)}}`) |
| B | Got first name | `{{2.step}}` = `firstName` |
| C | Got surname | `{{2.step}}` = `surname` |
| D | Got email | `{{2.step}}` = `email` **AND** `TEXT` *Contains* `@` |
| E | Bad email | `{{2.step}}` = `email` **AND** `TEXT` *Does not contain* `@` |
| F | Got inquiry | `{{2.step}}` = `inquiry` |

(After someone finishes, their step is `done` and the bot stays quiet for 7 days so you can talk to them yourself.
If they message again after 7 days, it starts over.)

On every route, add these two modules:
- **WhatsApp Business Cloud › Send a Message** with **Receiver** = `FROM`, **Type** = Text, and the **Body** from the table below.
- **Data store › Add/replace a record** with Data store = `WhatsApp Leads`, Key = `FROM`, and **Overwrite an existing record = Yes**.
  Fill in **every** field each time as shown in the table below. A field you leave empty gets wiped.

#### Route A: New conversation
- **Send a Message:**
  `Hi 👋 Thanks for contacting Conchel AI Automation! I'm busy right now, but I'll get back to you as soon as I can. While you wait, what's your first name?`
- **Add/replace a record:** step = `firstName`, firstName / surname / email / inquiry = *(empty)*, updatedAt = `{{now}}`

#### Route B: Got first name
- **Send a Message:** `Nice to meet you, TEXT! What's your surname?`
- **Add/replace:** step = `surname`, firstName = `{{trim(TEXT)}}`, updatedAt = `{{now}}`

#### Route C: Got surname
- **Send a Message:** `Thanks! What's your email address?`
- **Add/replace:** step = `email`, firstName = `{{2.firstName}}`, surname = `{{trim(TEXT)}}`, updatedAt = `{{now}}`

#### Route D: Got email
- **Send a Message:** `Great. Briefly, what would you like help with? (e.g. automating bookings, a WhatsApp bot, lead follow-ups…)`
- **Add/replace:** step = `inquiry`, firstName = `{{2.firstName}}`, surname = `{{2.surname}}`, email = `{{lower(trim(TEXT))}}`, updatedAt = `{{now}}`

#### Route E: Bad email
- **Send a Message:** `Hmm, that doesn't look like an email address. Please type it again (e.g. name@example.com).`
- No data store module is needed here. The step stays `email`.

#### Route F: Got inquiry (thank them and email you)
1. **Send a Message:** `Thank you, {{2.firstName}}! 🙏 I've got all your details and will be in touch soon.`
2. **Add/replace:** step = `done`, firstName = `{{2.firstName}}`, surname = `{{2.surname}}`, email = `{{2.email}}`, inquiry = `TEXT`, updatedAt = `{{now}}`
3. **Gmail › Send an email** (connect your Gmail account):
   - To: *your Gmail address*
   - Subject: `New WhatsApp lead: {{2.firstName}} {{2.surname}}`
   - Content type: HTML
   - Body:
     ```html
     <h2>New WhatsApp lead</h2>
     <p><b>Name:</b> {{2.firstName}}<br>
     <b>Surname:</b> {{2.surname}}<br>
     <b>Cellphone:</b> +FROM (<a href="https://wa.me/FROM">chat on WhatsApp</a>)<br>
     <b>Email:</b> {{2.email}}<br>
     <b>Message:</b> TEXT</p>
     ```

### Step 3: Turn it on
- Scenario settings (⚙️ at the bottom): set **Sequential processing = ON**. This makes sure messages
  sent quickly one after another are handled in order.
- Click **Save**, then toggle the scenario **ON** (Immediately as data arrives).
- Test it from a different phone: send "Hi" and answer the questions. You should get the email at the end.
- To test again with the same phone, open **Data stores → WhatsApp Leads** and delete that number's record.

---

## Notes
- WhatsApp lets businesses send free-form replies within **24 hours** of the client's last message. This bot always replies right away, so that's fine.
- Make counts each module run as an operation. One full conversation (5 messages) uses about 25–30 operations.

# Make.com scenario: Customer Inquiry

**Flow:** Customer opens `index.html` (welcome + form) → submits name, surname, cellphone, email, inquiry →
**Webhook** (trigger) → **Email** to you with all the details → **Webhook response** thanks the customer on screen.

| # | Module | What it does |
|---|--------|--------------|
| 1 | Webhooks › Custom webhook | Trigger – receives `firstName`, `surname`, `cellphone`, `email`, `inquiry` |
| 2 | Email › Send an email | Emails you the customer's details (only runs if name, cellphone and email are filled in) |
| 3 | Webhooks › Webhook response | Returns "Thank you, {name}! …" which the page shows to the customer |

## Setup (about 5 minutes)

1. In Make.com: **Scenarios → Create a new scenario → ⋯ (bottom toolbar) → Import Blueprint** → choose `make/hair-salon-inquiry.blueprint.json`.
2. Click module **1 (Custom webhook)** → **Create a webhook** → name it `Salon inquiry` → **Save**. Copy the URL it shows.
3. Open `index.html` and paste that URL into `WEBHOOK_URL` (near the bottom).
4. Back in Make, with module 1 open, click **Redetermine data structure**, then open `index.html` in a browser and submit a test inquiry. Make will learn the fields.
5. Click module **2 (Send an email)** → add/choose your email connection (Gmail, Outlook, or SMTP) → replace `YOUR_EMAIL@example.com` in **To** with your own address.
6. Click **Save**, then switch the scenario **ON** (Immediately as data arrives).
7. Submit the form once more – you should see the thank-you message and receive the email.

## Putting the form online
Host `index.html` anywhere (GitHub Pages on this repo works: Settings → Pages → Deploy from branch), or link to it from Instagram/WhatsApp/your website.

# Recovery Radar — Razorpay Revenue Recovery Agent

Built for the Razorpay Buildathon — **Revenue Recovery** track.

## What it does

Recovery Radar automatically detects failed payments and decides the best way to win that revenue back — instead of a customer's failed payment being silently dropped, the system:

1. Catches the failure (via Razorpay's `payment.failed` webhook, or a demo "Simulate Payment Failure" trigger)
2. Decides a recovery strategy based on *why* the payment failed
3. Generates a fresh, working payment link
4. Prepares a personalized recovery message and picks the right channel to send it on
5. Tracks the outcome on a live dashboard: total failures, total recovered, and recovery rate

## Why this matters

Failed payments are usually treated as dead ends. Most of them aren't — a card decline, an expired card, and insufficient funds all need *different* responses to actually get recovered. Recovery Radar treats each failure differently instead of sending the same generic "your payment failed" email to everyone.

## How the decision engine works

| Failure reason | Strategy | Channel | Tone |
|---|---|---|---|
| Card declined | Immediate retry | Email | Helpful troubleshooting |
| Insufficient funds | Delayed retry (24h) | WhatsApp | Friendly reminder |
| Expired card | Manual review | SMS | Helpful troubleshooting |
| Bank error | Delayed retry (3 days) | Email | Urgent |

**Current build:** this decision logic is rule-based, chosen deliberately to keep the demo fast, reliable, and free to run within hackathon constraints.

**Designed for:** the architecture is built to slot in an LLM (Claude) in place of the rule table, so the same pipeline can reason about failure context dynamically — factoring in customer history, amount, timing, and past recovery attempts — rather than following a fixed lookup table. This is the natural next step beyond the hackathon scope.

## Tech stack

- **Backend:** Node.js + Express
- **Payments:** Razorpay (Payment Links / Orders / Webhooks APIs, Test Mode)
- **Frontend:** HTML/CSS/JS dashboard
- **Hosting:** Replit

## Running it locally / on Replit

1. Clone this repo
2. Install dependencies: `npm install`
3. Set the following environment variables (Replit Secrets or `.env`):
   - `RAZORPAY_KEY_ID`
   - `RAZORPAY_KEY_SECRET`
   - `RAZORPAY_WEBHOOK_SECRET`
4. Run the server: `npm start`
5. Open the app in your browser and either:
   - Click **"Simulate Payment Failure"** to trigger a demo case instantly, or
   - Point a real Razorpay Test Mode webhook at `/webhook/razorpay` to test with real test transactions

## Demo flow

1. Open the dashboard
2. Click **Simulate Payment Failure**, pick a failure reason
3. Watch the case appear with its assigned strategy, channel, and reasoning
4. Mark it as **Recovered** to see the recovery rate update live

## What's next (beyond the hackathon)

- Swap the rule-based engine for an LLM-driven reasoning layer (Claude), factoring in richer context per customer
- Send real emails/SMS/WhatsApp instead of logging messages
- Add customer-level recovery history so repeat failures get escalated differently
- A/B test which recovery tone/channel actually converts best over time

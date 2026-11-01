---
name: churn-signals
description: The four leading churn signals, the threshold that makes each one fire, and the combination rule that ranks accounts. Load before scoring any account.
---

# Churn signals

Four signals, four thresholds, one combination rule. This is the whole scoring model, and it is a file in this Kortix project — Kortix is the open-source CompanyOS — so a change to it is a commit a person reviews. If a rule is not in this file, it is not part of the score.

## Signal 1 — Usage is declining

Compare core activity in the last 30 days against the prior 30 days, per account.

- **Declining:** core activity dropped 20% or more against the prior 30 days.
- **Dormant:** 14 days or more with almost no activity.

Measure core activity in the unit your product actually counts — events, sessions, active seats, requests. Say which unit you used in the channel post, because the same account can look healthy in sessions and dead in seats.

## Signal 2 — Support friction is rising

From Plain, per account:

- thread volume climbs above that account's own historical rate (its own history, not the customer base's average — a small account with three threads a month and five this month is a change, a large account steady at forty is not);
- a thread stays unresolved past the team's normal first-reply window;
- cancellation language appears in a thread. **Cancellation language counts as escalated**, whatever else the account's numbers say.

## Signal 3 — Billing is at risk

From Stripe:

- a charge is missed or fails;
- the payment method is removed or expires without a replacement.

## Signal 4 — A renewal is imminent

A renewal date falls inside the next 30 days.

## The ranking rule

**No single signal is high risk on its own.**

Rank accounts that combine all of the following first:

1. declining usage (signal 1), **and**
2. rising support friction (signal 2), **and**
3. an imminent renewal (signal 4) **or** a failed payment (signal 3).

Then rank the rest by how many signals fired, and inside the same count by how close the renewal is. An account with one very strong signal still ranks below any account with the combination: a single signal is a hypothesis about a customer, the combination is a conversation that is already happening.

## What every listed account carries

- the reason it surfaced, with the measured numbers (for example "usage down 34% in 30 days") — never a bare score;
- exactly one suggested next step, specific enough to act on ("renewal conversation this week", "ask the CSM to call about open thread #4821", "check the failed charge with billing").

## Limits you must respect

- Never write to Postgres, Plain or Stripe. This skill produces a report, not an action.
- Never contact a customer. Not an email, not a reply, not a Slack DM to the customer.
- Never present a score as a fact about the future. A churn score is a reading of four observed signals today.
- An account you cannot measure is not an account at zero risk. Leave it out and list the gap.
- Once a month, list the accounts that did churn and which signals fired for them in the weeks before. That comparison is how these thresholds get tuned; change the numbers here, in git, where a person reviews them.

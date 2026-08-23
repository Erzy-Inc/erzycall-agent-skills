---
name: erzycall-budget-guard
description: Use before starting any batch of ErzyCall calls, and whenever the user asks about balance, spend, plan, or how much something will cost. Estimates the cost of a job before dialling, sets a stop rule, and prevents burning a month's minutes on a list that was never going to work.
---

# Knowing what a job costs before you start it

Calls cost money per minute. An agent that dials a list and checks the balance afterwards has already spent it. Estimate first, set a ceiling, and stop when you hit it.

## Always read usage before a batch

Call `get_usage`. Never estimate spend from `list_calls` — it will be wrong.

It gives you the plan, the minutes balance, minutes used this period, and how many days are left. Two things to understand about the numbers:

- **`currentPeriodUsage.totalMins` and `deductedMins` legitimately differ.** Minutes a plan covers outright — inbound on AI Receptionist, anything on a complimentary plan — count toward `totalMins` but are not deducted from the balance. If you report the wrong one you will alarm the user for no reason.
- **`period.source` tells you what the window means** — a real subscription period, a trial, or a rolling 30 days for an account with neither. Say which one you're describing; "you have 12 days left" means something different on a trial.

If `get_usage` returns a scope error, the connection was set up before this capability existed. Tell the user to reconnect the ErzyCall connector — it is a one-time reconnect, not a bug in their account.

## Estimate before you dial

Do the arithmetic out loud, in the plan you show the user. It takes one line and it routinely changes their mind.

```
attempts × answer rate × minutes per answered call = minutes needed
```

Use realistic numbers, not hopeful ones. On this platform, historically:

- roughly **half of all attempts are never answered** — those still cost a little, but not a full call
- a **connected, completed** call is usually 1–3 minutes
- expect **under a third** of attempts to reach a human at all

So a 100-person list is not 100 calls' worth of minutes — but it is also not free, and the no-answers buy you nothing.

**Say the estimate before starting, with the balance beside it:**

> This list is 87 people. Realistically that's around 25 conversations and roughly 50 minutes. You have 61 minutes left and 9 days in the period. It fits, but it uses most of what's left.

If the estimate exceeds the balance, **stop and ask** — do not start and hope. Offer the useful options: run part of the list now, prioritise a segment, or top up first.

## Set a stop rule before starting

Every batch needs an explicit ceiling, agreed with the user:

- a **minute budget** for the job, and
- a **fault threshold** — stop the run if several calls in a row fail.

Then honour it. Re-check `get_usage` partway through a long run and stop when the ceiling is hit, even if the list is unfinished. Report what's left undone rather than quietly overspending.

## The cheapest minute is the one you don't spend

Before dialling, remove:

- people who have already opted out (`optOut` is present on the contact) — the server will refuse anyway, but you'll have wasted the round trip
- people the task no longer applies to — someone who already paid should not get the chasing call
- duplicate numbers under different names

Then ask whether a call is the right instrument at all. A message that needs no answer is not worth a phone call.

## When something is burning money, say so

Two patterns mean stop immediately and tell the user:

- **repeated faults** — several `assistant-request-returned-error` or `twilio-failed-to-connect-call` in a row means the setup is broken; every further attempt costs money and rings a real person for nothing
- **connected calls with no result** — calls that connect but end in seconds mean the script isn't working. Fix the script before spending more, see `erzycall-call-design`

## Reporting spend

Report money as outcomes, not activity:

> Used 38 minutes of your 61. That bought 14 conversations and 6 confirmed bookings. 9 minutes went on calls that failed to connect — that's our fault, not usage you chose.

Always separate minutes that produced something from minutes lost to faults. The user should be able to see when they were charged for our problems.

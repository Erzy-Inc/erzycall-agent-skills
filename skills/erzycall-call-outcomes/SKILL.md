---
name: erzycall-call-outcomes
description: Use when placing a phone call through ErzyCall, or when reading the result of one. Defines what success means before dialling, and how to tell whether a finished call actually achieved anything. Use whenever you are about to call create_call, or have a call ID and need to know what happened.
---

# Reading what a call actually did

## The one rule that matters

**`status: "ended"` does not mean it worked.** It means the phone call is over. Most calls end without a human ever hearing your message.

Across the last 150 real calls on this platform:

| What happened | Share |
|---|---|
| Nobody answered | 52% |
| Failed on our side — never connected, or errored into silence | 24% |
| Human picked up and hung up (could be anything) | 17% |
| **Assistant actually delivered its message** | **6%** |

If you report "done" the moment a call ends, you will be wrong roughly nineteen times out of twenty. Never tell the user a call succeeded because it finished.

## Before you dial: write down what success is

Do this first, in one sentence, and keep it. It must be checkable from a transcript.

- Bad: "call the customer about the invoice"
- Good: "the customer says out loud whether they will pay by Friday"

If you cannot say what a successful transcript would contain, you are not ready to dial — ask the user what outcome they actually want.

## After the call: how to read the result

Call `get_call` and read **`endedReason`** first, then the transcript. Never read `status` alone.

| `endedReason` | What really happened | What to do |
|---|---|---|
| `assistant-ended-call` | The assistant finished its script. **Only this is a candidate for success.** Still read the transcript — finishing the script is not the same as achieving the objective. | Check the transcript against your written objective |
| `customer-ended-call` | A human hung up. Could be a completed conversation or an instant rejection. Ambiguous by itself. | Read the transcript. Short duration = they hung up on you |
| `customer-did-not-answer` | Nobody picked up. **This is not a failure and not a rejection.** | Retry later — see the retry table |
| `customer-busy` | Line engaged | Retry later |
| `twilio-failed-to-connect-call` | The call never reached the network. Our problem, not theirs. | Retry sooner. If it repeats for one number, the number may be bad |
| `assistant-request-returned-error` | **Our defect.** The person's phone rang and delivered silence. | Stop. Do not retry in a loop. Report it |
| `silence-timed-out` | The call connected but nobody spoke | Treat as a defect; investigate before retrying |

### Duration is your second signal

`durationSeconds: 0` with an `ended` status means nothing was said, whatever the reason claims. A "successful" call lasting three seconds did not accomplish anything.

### Then read the transcript

Ask two questions, in order:

1. **Did a human actually engage?** A transcript with only assistant turns means you talked to a voicemail or to nobody.
2. **Does it contain the outcome you wrote down?** Quote the line that proves it. If you cannot quote it, the objective was not met — say so.

## Retrying

| Situation | Retry? |
|---|---|
| No answer / busy | Yes — after a gap, at a different time of day. Attempts to the same number are capped server-side |
| Failed to connect | Yes, sooner — it is an infrastructure blip |
| Our assistant errored | **No.** Stop and report. Retrying a broken configuration burns money and rings real people for nothing |
| Person asked not to be called | **Never.** Suppress them — see `erzycall-contact-safety` |
| Objective met | No |

Two rules that override the table:

- **Never retry in a tight loop.** If the same number fails twice in a row for the same reason, stop and tell the user.
- **Nothing caps how often you dial one number.** The server will happily let you call the same person again and again. That restraint has to come from you — see `erzycall-campaign-runner`.
- A call refused because the person opted out comes back as `403 CONTACT_OPTED_OUT`, and is recorded with `endedReason: "contact-opted-out"`. That is a permanent stop, not a retryable failure.

## What to tell the user

Report outcomes, not activity. "I placed 40 calls" is not a result.

Say, in this shape:

> Reached 6 of 40 people. 3 confirmed for Tuesday, 2 asked to reschedule, 1 asked not to be contacted again (suppressed). 21 didn't answer and can be retried this evening. 8 failed to connect — that's on our side. 4 hit a fault and I've stopped retrying them.

Always separate:
- **reached and resolved** — the actual result
- **not reached** — retryable, no information gained
- **broke** — our fault, needs a human

Never present "not reached" as a negative answer. Nobody said no; nobody said anything.

## Timing

A call takes roughly a minute to actually ring after `create_call` returns. Do not poll in a tight loop and do not block. Create the call, do something else, come back for the result.

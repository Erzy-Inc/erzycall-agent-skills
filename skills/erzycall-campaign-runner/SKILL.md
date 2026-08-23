---
name: erzycall-campaign-runner
description: Use when calling more than one person through ErzyCall — a list, a batch, a follow-up round, or any repeated calling job. Covers preparing the list, pacing the run, waiting correctly, and deciding what to retry. Use whenever the task involves calling several people rather than one.
---

# Running a batch of calls

A batch is not a loop over `create_call`. Most of the work happens before the first dial and after the last one.

## 1. Prepare the list first

In this order:

1. **De-duplicate by phone number.** The same person appears twice under different spellings more often than you'd think. Two rows is not permission to call twice.
2. **Drop opted-out contacts** (`optOut` present). The server will refuse them anyway, but a rejection is a safety net, not a plan.
3. **Drop people the task no longer applies to.** Ask the user how to tell — "who should I skip?" is a cheap question that prevents an expensive mistake.
4. **Estimate the cost** and agree a ceiling — see `erzycall-budget-guard`.
5. **Write down what success is**, in one checkable sentence — see `erzycall-call-outcomes`.

Show the user the final count before you start: *"After removing 3 duplicates and 1 suppressed number, that's 46 people."*

## 2. Pace the run

- **Don't fire the whole list at once.** Space the calls out.
- **Keep concurrency low.** A handful in flight, not fifty.
- **Respect local hours** for the destination, not yours.
- **There is no server-side cap on how often you may dial one number.** Nothing will stop you from calling the same person ten times in an hour. Pacing is entirely your responsibility — treat two attempts in a day as the ceiling unless the user has explicitly asked for more.

## 3. Wait correctly

**A call takes roughly a minute to actually ring after `create_call` returns.** This is the mistake most likely to waste your time:

- Do **not** block waiting for one call to finish before starting the next.
- Do **not** poll `get_call` in a tight loop.
- Start the calls you're going to start, do something else, then collect results.

A call that still looks unstarted after a couple of minutes is stuck — flag it rather than retrying blindly.

## 4. Interpret each result

Read `endedReason`, not `status`. Full table in `erzycall-call-outcomes`. The short version:

| Outcome | Next |
|---|---|
| Assistant completed | Check the transcript against your objective |
| Human hung up | Read the transcript — it's ambiguous |
| No answer / busy | Retry later, different time of day |
| Failed to connect | Retry sooner — infrastructure blip |
| Assistant errored | **Stop this number.** Don't retry |
| Person asked to stop | **Suppress immediately** — see `erzycall-contact-safety` |

## 5. Know when to abort the whole run

Stop everything and tell the user if:

- **several calls in a row hit faults** — the setup is broken and every further call rings a real person for nothing
- **the minute ceiling is reached**
- **connected calls are ending in seconds** — the script isn't working; fix it before spending more
- **more than one person asks not to be called** — that's a signal the list or the premise is wrong, not just individual opt-outs

Aborting a bad run early is the single most valuable judgement in this skill. A batch that is failing does not improve by continuing.

## 6. Retry rounds

- Space retries by **hours, not minutes**, and try a different time of day — the same number at the same time gets the same voicemail.
- **Two failed attempts for the same reason is enough.** Stop and report.
- Never restart a whole list from the top. Retry only what genuinely warrants it, and never anyone who was reached or suppressed.

## 7. Report

Report the run once, as outcomes:

> 46 people. Reached 13: 8 confirmed, 3 want to reschedule, 2 said no. 1 asked not to be contacted again — suppressed. 24 didn't answer and are worth retrying this evening. 8 failed to connect on our side.

Group into **resolved**, **not reached** (retryable, no information), and **broke** (our fault). Never present "didn't answer" as a refusal — nobody said no, nobody said anything.

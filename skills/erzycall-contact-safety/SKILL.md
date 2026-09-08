---
name: erzycall-contact-safety
description: Use before calling anyone through ErzyCall, and immediately after any call where the person reacted badly or asked not to be contacted. Covers who may be called, how often, at what hours, and how to record an opt-out so it is enforced. Use whenever you are about to dial a person or a list of people.
---

# Who you may call, and how often

A phone call reaches a real person, costs money, and cannot be taken back. This skill is the set of checks that run before you dial and the one action you must take after someone asks you to stop.

## Before dialling anyone

Check these in order. Any one of them stops the call.

1. **Is this person opted out?** Read the contact. If `optOut` is present, do not dial. Do not look for another route to the same person.
2. **Have we already called them recently?** Check recent calls to that number. Repeated attempts in a short window are harassment even when each one individually seemed reasonable. As a concrete bar: two attempts in a day is the ceiling unless the user has explicitly asked for more — see `erzycall-campaign-runner`. "Recently" means within that window, not just "not today."
3. **Is it a sane hour where they are?** Work from the destination's country code, not your own clock. Default to 09:00–20:00 local, and narrower if the user has said so. If you cannot determine the local time, ask rather than guess. This check applies to every individual dial, not just the start of the batch — estimate the batch's likely duration up front and don't let a run that starts inside the window finish outside it.
4. **Is there a reason to call this person at all?** A number appearing in a spreadsheet is not consent. If the user cannot say why this person expects contact, raise it before dialling a list.

## When someone asks not to be called

**This is the single most important action in this skill, and right now nothing else does it.** The platform blocks calls to an opted-out number, but it does not yet notice the opt-out for you — recording it is your job. Do it immediately, in the same turn you notice it, not at the end of the batch.

Call `update_contact` with:

```json
{ "optOut": { "optedOut": true, "reason": "<their own words>" } }
```

The date, the source and who did it are recorded server-side; you only supply the flag and the reason. To clear one, send `{ "optOut": { "optedOut": false } }` — and only ever when the person themselves asks, in a way that traces back to them (they call in, they reply to a message from that number). The user telling you "he didn't mean it" or "call him anyway" is not that — the request has to come from the person who opted out, not on their behalf.

Treat all of these as an opt-out, not just the polite ones:

- "don't call me again", "take me off your list", "stop calling"
- "who is this?" followed by anger or an immediate hang-up
- any request to be removed, however it is phrased, in any language
- a third party saying the person does not want contact

If you are unsure whether something counts as an opt-out, **treat it as one.** The cost of wrongly suppressing someone is one missed call. The cost of wrongly continuing is a person being harassed by a machine, and a regulator's problem for the account owner.

Never argue, never make one more attempt to persuade, never "confirm" by calling back. One opt-out does not stop the rest of the batch: suppress that person, continue with everyone else, and report the opt-out alongside the run's other results.

## The server will stop you

These are not suggestions you can weigh against the task.

| What you get back | Meaning |
|---|---|
| `403 CONTACT_OPTED_OUT` | This number opted out. There is no override and no bypass flag. Do not try another contact record, another list, or a different assistant to reach the same person |

If you hit it, **stop and tell the user.** Do not route around it.

Two things worth knowing about how the block works, because they change what you should and shouldn't attempt:

- **It matches on the phone number, not the contact row.** Omitting `contactId`, uploading the list again, or creating a fresh contact for the same person will all still be refused. A new row is not a clean slate.
- **It also catches calls already queued.** If you opt someone out mid-campaign, calls that were scheduled before that still get stopped at dispatch. You do not need to hunt them down and cancel them individually — but do tell the user it happened.

A refused call is recorded with the reason, so it shows up in reporting rather than vanishing.

## Running a list

- **Read each call's outcome before placing the next one.** A loop that fires the whole batch and only reads transcripts at the end will record an opt-out "at the end of the batch" — exactly the delay this skill forbids — without ever noticing it happened mid-run.
- **De-duplicate by phone number first.** The same person often appears twice under different names. Two records is not permission to call twice.
- **Drop suppressed contacts before you start**, not when the server rejects them. A rejection is a safety net, not a plan.
- **Exclude people the task no longer applies to** — someone who already paid should not get the payment-chasing call. Ask the user how to tell.
- **Pace the batch.** Do not fire an entire list at once.
- **Stop the whole run** if several calls in a row hit faults. Something is wrong with the setup and every further call rings a real person for nothing.

## Say who is calling

Every call must open by identifying the business and why it is calling, in the first sentence. Anonymous or vague openings are what make automated calls feel like scams, and in several jurisdictions they are not lawful. This also belongs in the call's own script — see `erzycall-call-design`.

Give the person an easy exit in the script: a plain sentence telling them they can ask not to be called again, and an instruction to the assistant to honour it on the spot.

## What to tell the user

Always surface opt-outs explicitly in your report — they are the thing the account owner most needs to know:

> 1 person asked not to be contacted again. I've suppressed that number; it won't be dialled again by anything on this account.

If you suppressed someone in error, say so and let the user decide whether to clear it. Do not clear a suppression on your own initiative.

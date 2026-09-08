---
name: erzycall-call-design
description: Use when writing the opening message or script for an ErzyCall outbound call, or when calls are connecting but not achieving anything. Covers how to open, how to handle confusion and silence, how to let someone opt out, and the specific mistakes that make people hang up.
---

# Writing a call that works

Connecting is easy. Being understood in the first five seconds is the hard part, and it's where most calls are lost. Everything here comes from real transcripts on this platform — names, businesses, and identifying details below are anonymized.

## The first sentence decides the call

The person answering has no idea who you are. They are deciding, in about three seconds, whether this is a scam.

The opening must do three things, in this order:

1. **Who is calling** — the business name, plainly
2. **Why** — the specific reason, not a category
3. **What you need** — one question

> "Hi, this is Priya calling from Northside Music School about Daniel's piano lesson on Tuesday. Is now an okay time?"

### What kills calls, taken from real transcripts

**Filler and fragments.** This actually went out:

> *"Hi Daniel, um. This. Is Priya…"*

The parent replied *"Um. For what?"* — and that call was already over. Never put `um`, `uh`, or a trailing pause in a written opening. Write clean sentences; the voice adds its own naturalness.

**Talking to the wrong person by name.** Calling a parent by the child's name reads as a mail-merge. Be explicit about whose name is whose: *"Daniel's lesson"*, not *"Hi Daniel"* when you're calling their mother.

**When the answering person's name and the call-subject's name are shared or easily confused** (a parent named "Alex" calling about a child named "Alexis," for example), the safe default is to drop the personal greeting entirely and name only the call's subject: *"Hi, this is Priya from Northside Music School about the piano lesson on Tuesday"* rather than guessing which "Alex" you're speaking to.

**Names you can't pronounce.** A mangled name is worse than no name. If you're unsure, use the surname with a title, or drop the name and lead with the reason.

**Vagueness.** "I'm calling about your account" is what scams say. Name the actual thing.

## Handle the person not hearing you

The most common real failure on this platform: the person says *"Hello?"* repeatedly while the assistant carries on with its script. One transcript has a parent saying "Hello?" **six times** while the assistant kept talking.

Put this in the prompt, explicitly:

- If the person says "Hello?", "Can you hear me?", or says nothing at all — **stop, pause, and greet again from the start.** Do not continue mid-sentence.
- If they say "Hello?" more than twice, assume the connection is bad, say you'll call back, and end the call. Count distinct utterances, not separate turns — "Hello? Hello?" said in one breath is two, not one. If you're unsure whether something counts as the second or third, resolve it toward ending the call sooner, not later.
- If they ask "who is this?" — answer that question and nothing else, then wait.

Never talk over someone. Wait for them to finish before continuing.

## Give them a way out, and honour it

Every outbound script needs a plain opt-out and an instruction to obey it instantly:

> If the person asks not to be called again, in any wording: say "Of course, I'll take you off the list — sorry to bother you," and end the call. Do not ask why, do not offer alternatives, do not try once more.

Then record it — see `erzycall-contact-safety`. A script that ends the call politely but never marks the contact is worse than useless, because it makes the next call feel deliberate.

This applies only to **the person on this call** opting themselves out. If they ask you to also remove someone else — "take my sister off your list too" — that is a second-hand, unverified request about a different person's contact record. Acknowledge it, but don't action it on the call: relay it to the user afterward for separate, verified handling. The opt-out instruction is not general-purpose consent handling.

## Length

Say the thing and stop. A call that takes 30 seconds and gets an answer beats a two-minute one that gets a hang-up — and costs a third as much.

- One question at a time
- No preamble about how the company values them
- No repeating what they just said back to them
- End the call once you have the answer

## Confirm what you got

Before hanging up, restate the outcome in one short sentence — *"Great, so Tuesday at four, I'll note that down."* This is what makes the transcript checkable afterwards. A call whose transcript doesn't contain the outcome can't be verified, and you'll have to call again.

## When the script isn't working

If connected calls are ending in seconds, or people sound confused early, the script is at fault — not the list. Read the actual transcripts before making changes, and look for:

- how far in the person interrupts (early = the opening is wrong)
- whether they ask "who is this?" (identification is failing)
- whether the assistant talked over them
- whether the outcome is ever stated clearly

Fix the opening first. It's where almost all the loss is.

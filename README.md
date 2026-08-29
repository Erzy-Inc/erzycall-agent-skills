# ErzyCall Agent Skills

Instructions that ship alongside the [ErzyCall MCP server](https://app.erzycall.com/docs/mcp) and teach an AI agent how to use it well.

The MCP server describes *what* it can do — 31 tools, self-describing schemas. It cannot describe *judgement*: when to retry, when to stop, whether the objective was actually met, whether this number should be dialled at all. Every agent that connects has to invent that policy from scratch, in the moment, with no data.

These skills are that policy.

## The problem they solve

Across the last 150 real calls on the platform:

| What happened | Share |
|---|---|
| Nobody answered | 52% |
| Failed on our side — never connected, or errored into silence | 24% |
| Human picked up and hung up | 17% |
| **Assistant actually delivered its message** | **6%** |

An agent that reads `status: "ended"` as success is wrong roughly nineteen times out of twenty — and tells the user the job is done. No amount of tool documentation fixes that.

## The skills

| Skill | What it prevents |
|---|---|
| **erzycall-call-outcomes** | Reporting "done" for calls that reached nobody. Reading a result correctly |
| **erzycall-contact-safety** | Calling people who asked not to be called. Recording an opt-out the moment it happens |
| **erzycall-budget-guard** | Spending a month's minutes on a list that was never going to work |
| **erzycall-campaign-runner** | Blocking, hammering, and retry logic invented on the fly |
| **erzycall-call-design** | Openings that get hung up on in the first three seconds |

Read them in that order. The first two matter most.

## Using them

Each skill is a single `SKILL.md` with YAML frontmatter, in the format most agent harnesses read. Point your agent at this repo, or copy the directories into wherever your harness loads skills from.

They assume the agent is connected to the ErzyCall MCP:

```
https://app.erzycall.com/api/mcp
```

OAuth, no API key. Setup per client: https://app.erzycall.com/docs/mcp

## What is enforced vs advised

A skill is guidance. It shapes a cooperative agent; it cannot stop a careless one. Some of what these skills describe is also enforced server-side, and that distinction matters:

| Behaviour | Enforced by the server? |
|---|---|
| Refusing to call someone who opted out | **Yes** — `403 CONTACT_OPTED_OUT`, no bypass, matched on the number so a fresh contact row is not a clean slate |
| Blocking already-queued calls when an opt-out lands | **Yes** — caught at dispatch |
| Noticing that someone asked to stop, and recording it | **No** — this is the agent's job. It is the single most important action in `erzycall-contact-safety` |
| How often one number may be dialled | **Proposed platform feature, not yet shipped** |
| Spend ceilings | **No** — advisory only today |

Where a skill says "the server will refuse," it will. Where it says restraint is your responsibility, nothing is watching.

## Status

Written 2026-08-23/24, from real call transcripts and production failure data. Not yet validated against agents in the wild — the intended next step is adversarial testing: give a fresh agent a goal plus a skill, watch where it still does the wrong thing, fix the skill.

Field names and error codes were checked against what is actually deployed, not against what was planned.

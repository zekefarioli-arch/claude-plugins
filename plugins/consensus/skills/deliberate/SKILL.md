---
name: deliberate
description: Run a structured deliberation on a design or technical decision using independent subagents with forced stances (advocate, critic, pragmatist), a rebuttal round and a synthesis that keeps disagreements visible. Use this whenever the user asks for consensus, a debate, a second opinion, a devil's advocate, "challenge this", "poke holes in", "is this a good idea", or wants several perspectives before committing to an architecture, library, data model or trade-off. Do not use it for factual lookups or small reversible choices.
argument-hint: "[quick] <decision or question>"
---

# Deliberate

Structured deliberation for decisions where a single answer is likely to just agree with the framing it was given. Three subagents from this plugin argue fixed stances without seeing each other, then rebut each other once, then you synthesize.

The subagents are `consensus:advocate`, `consensus:critic` and `consensus:pragmatist`. They cannot see this conversation; everything they know comes from the brief you write.

## Modes

- **full** (default): round 1, rebuttal round, synthesis. Roughly 6 subagent calls.
- **quick**: round 1 and synthesis only. Use it when the argument starts with `quick` or the user asks for a fast take.

## Step 1: Frame the decision

Turn the request into one decision question with explicit options.

- Always list at least two options. If the user gave one proposal, add the strongest realistic alternative and "defer or do nothing" when that is a real option.
- Collect constraints: stack, scale, deadlines, team skills, budget.
- Collect **locked decisions**. Check `CLAUDE.md`, `docs/decisions/`, `docs/adr/` or similar in the current repo, plus anything the user said in this conversation. If the question would reopen a locked decision and the user did not ask to reopen it, stop and say so before running the debate. Deliberating on settled decisions wastes time and erodes them.
- Detect leading framing. If the user signals the answer they want ("I think X is better, right?"), note it for yourself and write the brief neutrally. Do not tell the subagents which option the user prefers; that is exactly the bias this skill exists to remove.

If something essential is missing (for example there is no way to tell what the options are), ask one question. Otherwise proceed and state your assumptions in the brief.

## Step 2: Write the brief

The brief is the only context the subagents get, so make it self-contained:

```
DECISION: <one question>
OPTIONS:
A. ...
B. ...
CONTEXT: <system, scale, stage of the project>
CONSTRAINTS: <hard constraints>
LOCKED DECISIONS (do not argue against these): <list, or "none">
RELEVANT FILES: <paths they should read, or "none">
ASSUMPTIONS: <anything you assumed>
```

## Step 3: Round 1 (independent)

Launch the three subagents **in parallel, in a single message**, each with the same brief plus the line "Round 1. Give your position in the required format." Parallel launch matters for independence as well as speed: none of them can anchor on another's answer.

## Step 4: Rebuttal round (full mode only)

Send each subagent the brief again, plus the other two agents' round 1 outputs, plus "Rebuttal round. Concede what is right, rebut what is wrong, update your position." Launch all three in parallel again.

Skip this round if all three agree in round 1 **and** the critic's confidence in its objection is below 30. In that case say so in the synthesis.

## Step 5: Optional external models

If the session has a multi-model consensus tool available (for example the `consensus` tool from PAL MCP), you may also send it the same brief with one model "for" and one "against", and treat the results as additional voices. Never require it. See `${CLAUDE_PLUGIN_ROOT}/docs/external-models.md`.

## Step 6: Synthesize

You are the judge, not a fourth debater. Rules that keep the synthesis honest:

- **Weigh arguments, not votes.** Two agents agreeing on a weak argument lose to one agent with a strong, evidenced one.
- **Keep disagreements visible.** Do not resolve a disagreement by averaging or by picking the more confident voice. If it is unresolved, say what evidence would resolve it.
- **Do not drift toward the user's preference.** If your recommendation matches what the user hinted at, check that the arguments, not the framing, got you there.
- **Always report the critic's strongest objection**, even when you recommend the option it targets, together with how to mitigate it.
- **Separate facts from estimates.** Flag any claim the agents made without evidence.

Write the synthesis in the user's language, using this structure:

```
## Question
<the decision, one line>

## Positions
| agent | final position | confidence | changed after rebuttal? |

## Where they agree
<short list>

## Unresolved disagreements
<each one, plus the evidence that would settle it>

## Strongest objection
<critic's main objection and how to mitigate it>

## Recommendation
<option, confidence (low, medium, high), and what would flip it>

## Cheapest way to validate
<the concrete spike or experiment, if any>
```

Keep it tight. The user wants a better formed opinion, not a transcript of the debate. Offer the raw agent outputs only if they ask.

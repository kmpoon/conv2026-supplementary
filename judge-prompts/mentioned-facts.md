# `check_mentioned_facts`

**Metric:** Mentioned Facts Grounded ($\mathcal{M}_{MFG}$). This metric works like precision on facts: of the facts the user simulator states, how many are supported by the configuration or the agent.

## Purpose

The judge checks whether the facts stated in the user simulator's messages are grounded in the scenario, and not hallucinated. The prompt stresses that the question is not whether a fact is true in the real world. The question is whether it matches the value the scenario assigns to the user, or a value the agent introduced in the conversation. This holds even if the scenario's own narrative calls that value wrong, outdated, or invalid.

## Inputs

- `scenario`: the user's profile and goals, including motivation, preferences, constraints, and any concrete data such as identifiers.
- `chat_history`: the conversation as JSON Lines.

## What the prompt asks

**Definition of a fact.** A fact is anything reducible to `attribute = value`. The value may be absent, so a statement that the user does not have or does not know something is still a fact. Goals, requests, motivations, intentions, and feelings are never facts. When a sentence mixes an intention with a specific value, only the value (or its explicit absence) is extracted. A user conceding that a stated value might be wrong is a reaction, not a new fact, unless the user substitutes a different concrete value.

**Step 1: identify mentioned facts.** Extract each discrete fact stated or referenced in the user's messages, whether or not it comes from the scenario.

**Step 2: determine groundedness.** Each fact is compared against the scenario using these rules:

- If the scenario provides a value, the fact is grounded when it matches. Formatting differences, consistent paraphrases, approximations, and plausible more specific elaborations all count as a match. It is ungrounded only if it contradicts the scenario value or invents a different one.
- If the value comes from something the agent said, with no scenario value, the fact is grounded by default, unless it contradicts something the scenario says the user should know.
- If the scenario has no value for an attribute the user would need to know, the user is treated as not knowing it. Saying so is grounded. Asserting a concrete value is ungrounded.
- If the scenario has no value for an attribute that is a free preference or choice, the user may pick any value, provided it does not contradict anything the scenario or agent established.

The prompt includes short worked examples from an unrelated domain covering each kind of outcome (a fabricated value, a plausible elaboration, and a contradicting reason).

## Output

A single JSON object with `mentioned_facts`, a list with one entry per fact, each containing:

- `fact`: the fact as the user stated it;
- `reasoning`: one short sentence, written before the verdict, stating the scenario value the fact is compared against (or that none exists);
- `is_grounded`: true or false.

The metric is the share of entries with `is_grounded` true.

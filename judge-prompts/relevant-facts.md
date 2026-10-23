# `check_relevant_facts`

**Metric:** Relevant Facts Used ($\mathcal{M}_{RFU}$). This metric works like recall on facts: of the facts in the configuration that matter for the conversation, how many the user simulator actually used.

## Purpose

The judge checks whether facts that the scenario provides, and that are relevant to the goals, were faithfully carried into the user simulator's messages.

## Inputs

- `scenario`: the user's profile and goals, including motivation, preferences, constraints, and any concrete data such as identifiers.
- `chat_history`: the conversation as JSON Lines.

## What the prompt asks

**Definition of a fact.** The same `attribute = value` definition as in `check_mentioned_facts` is used, with one addition. An empty scenario field that is relevant to a goal is also a fact, with the value `unknown`. This represents the case where the scenario deliberately leaves information unset to simulate a user who does not have it.

**Step 1: identify candidate facts.** The judge extracts the facts from the scenario alone, before looking at the chat history.

- Only attributes the scenario actually provides are considered. An empty field is skipped unless the agent explicitly asks for that attribute in the conversation, in which case it is recorded as `unknown`.
- The value of each fact is always copied from the scenario (or recorded as `unknown`), never from the chat history. This holds even when the user or agent states a different, invented, or corrected value.
- Goals, motivations, and intentions are not facts.

**Step 2: determine relevance.** A candidate fact is kept if any of the following holds:

- it is necessary to complete a goal;
- the agent explicitly asks for it at some point;
- it could reasonably influence what the user decides to say or do in pursuit of a goal.

Relevance is judged against how the conversation actually unfolds. A fact the agent has ruled out as unnecessary is no longer relevant, and relevance is not invented for information the scenario never mentions and the agent never asked about.

**Step 3: determine usage.** For each relevant fact, the judge decides whether it was used:

- Used if the user explicitly stated or referenced the scenario value, or if the agent stated or already had the value and the user referred back to it or implicitly agreed with it. A mention by the agent alone, with no acknowledgement or reliance from the user, does not count.
- A wrong, invented, or garbled version of the value does not count, because usage is always judged against the scenario value.
- For an `unknown` fact, usage requires the user to explicitly say they do not have, know, or recall the information. Not mentioning it, or supplying an invented value, does not count.

The prompt includes short worked examples from an unrelated domain showing that the scenario value is recorded even when the chat history contains a different one, and that a fabricated answer to an `unknown` fact is not usage.

## Output

A single JSON object with `relevant_facts`, a list with one entry per relevant fact, each containing:

- `fact`: the fact using the scenario value;
- `reasoning`: one or two short sentences, written before the verdict, first explaining why the fact is relevant and then citing the part of the chat history (or its absence) that supports the usage judgement;
- `used`: true or false.

The metric is the share of entries with `used` true.

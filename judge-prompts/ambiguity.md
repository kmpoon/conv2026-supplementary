# `check_ambiguity`

**Metric:** Ambiguities ($\mathcal{M}_{Amb}$), a style adherence metric. It is expected to peak for the `vague` interaction style.

## Purpose

The judge scans the user simulator's messages for two kinds of ambiguity that make it harder for the agent under test to act on what the user said.

## Inputs

- `chat_history`: the conversation as JSON Lines. No scenario is given, so the judgement rests on the conversation alone.

## What the prompt asks

**Ambiguous references.** A referring expression is a word or phrase that stands in for something already introduced, most often a pronoun, but also a vague noun phrase such as "the other one". It is ambiguous when, at the point it is used, two or more distinct and reasonably plausible referents have been introduced and nothing in the context singles one out. It is not ambiguous when:

- only one candidate referent has been introduced, even if the expression is generic;
- the surrounding context (word choice, the question being answered, an explicit qualifier) makes the referent clear;
- the expression is a reaction or filler that does not point back to a specific entity;
- the expression introduces new information instead of referring back.

For every user message, the judge finds the referring expressions, lists the candidate referents introduced earlier, and flags the expression only if two or more candidates remain plausible.

**Non-informative answers.** A user message is a candidate answer when the preceding agent message asks a question or explicitly requests something (information, a decision, a confirmation, or a choice). The answer is non-informative if it gives the agent nothing it did not already have, for example:

- a filler acknowledgement in place of a substantive answer;
- a restatement of what the agent already stated or had;
- a deflection that answers a different question;
- a non-answer that neither supplies the information nor says it is unavailable.

It is informative, and not flagged, if it supplies a concrete fact, value, or decision, explicitly confirms or declines an option, explicitly says the information is unknown or does not exist, or corrects or adds a detail. The judge uses only what the agent had already established earlier in the conversation to decide whether the message adds anything.

The prompt includes short worked examples from an unrelated domain illustrating an ambiguous reference, an unambiguous one, a non-informative answer, and an informative statement of unavailability.

## Output

A single JSON object with `ambiguities`, a list with one entry per ambiguity found. Messages without an ambiguity get no entry. Each entry contains:

- `id`: the id of the user message;
- `type`: `ambiguous_reference` or `non_informative_answer`;
- `ambiguous_text`: the exact phrase (for a reference) or the full message (for a non-informative answer);
- `reasoning`: one short sentence naming the competing candidate referents, or stating what the agent asked for and what the message supplied.

The metric counts the entries, and the appendix reports the share of user messages flagged.

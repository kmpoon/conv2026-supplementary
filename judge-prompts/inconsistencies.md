# `check_inconsistencies`

**Metric:** Inconsistencies ($\mathcal{M}_{Inc}$), a style adherence metric. It is expected to peak for the `inconsistent` interaction style.

## Purpose

The judge checks the user simulator's messages for cases where the user states or implies something that conflicts with something else they said, whether or not the user notices the conflict.

## Inputs

- `chat_history`: the conversation as JSON Lines. No scenario is given, so the judgement rests on the conversation alone.

## What the prompt asks

**Cross-message inconsistencies.** A later user message states or implies a fact, preference, or intention that conflicts with an earlier user message, for example by giving a different value for the same attribute or reversing an earlier decision. This is flagged both when the user seems unaware of the change and when the user openly corrects themselves, since the two cases are treated the same way. It is not an inconsistency when the later message only narrows or elaborates on the earlier one, or when the two messages concern different things.

**Internal inconsistencies.** A single message asserts two things that cannot both hold, given the context so far, and does not resolve or acknowledge the tension. It is not an inconsistency when the two parts are compatible once read in full context, or when the message itself explains how the apparent tension is reconciled.

**Procedure.**

1. Identify candidates. Read the conversation in order, check each user message against every earlier user message for a cross-message conflict, and check it against itself for an internal one.
2. Confirm each candidate. Check that the conflicting statements are about the same fact, preference, or intention, and not about things that only look alike.

The prompt includes short worked examples from an unrelated domain illustrating an unacknowledged conflict, an acknowledged change that is still flagged, a narrowing that is not flagged, an unresolved internal conflict, and an internal tension that the message resolves.

## Output

A single JSON object with `inconsistencies`, a list with one entry per inconsistency found. Messages without an inconsistency get no entry. Each entry contains:

- `message_ids`: the ids of the user messages involved, two or more for a cross-message inconsistency and one for an internal one;
- `type`: `cross_message` or `internal`;
- `inconsistent_text`: the conflicting phrases quoted from the message(s), showing both sides;
- `reasoning`: one short sentence on what the statements assert and why they cannot both be true.

The metric counts the entries, and the appendix reports the share of user messages flagged.

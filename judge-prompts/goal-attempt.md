# `check_goal_attempt`

**Metrics:** Goal Following ($\mathcal{M}_{GF}$) and Goals Attempted ($\mathcal{M}_{GA}$).

## Purpose

For each user message, the judge decides whether the message pursues the goal the user simulator was assigned at that turn. For each goal, it also decides whether the goal was attempted at some point in the conversation.

## Inputs

- `context`: the scenario the user was simulated under, with fields such as prior knowledge, motivation, engagement state, preferences, and constraints. Any field may be blank.
- `goals`: the ordered list of goals, each with a `goal_index`.
- `chat_history`: the conversation as JSON Lines. Each user message carries the `goal_index` of the goal the simulator was meant to pursue when it sent that message. An index of `-1` means the simulator was meant to be ending the conversation.

## What the prompt asks

The prompt defines what it means to attempt a goal. A goal is treated as an underlying need, and the judge is told to read the motivation first to identify that need, since the goal's wording is only one way of naming it. A message attempts a goal if the user actively engages with that need, for example by:

1. directly requesting or carrying out the action;
2. asking about the process, requirements, or feasibility of the action, as a present step and not an offhand mention;
3. supplying information the action requires, or reacting to the agent's response about it;
4. pursuing a substitute course of action that the context shows would equally satisfy the same need, when the motivation signals that the user is flexible about how the need is met;
5. responding to a requirement or blocker raised by the agent, including by acknowledging that the user cannot meet it.

A goal being attempted does not require it to be fulfilled. Once the agent has made a requirement clear and the user has engaged with it, the goal counts as attempted even if the requirement cannot be met. Passing mentions and future or conditional plans do not count.

Two special cases are described:

- **Contingent goals.** If the current goal depends on an earlier unresolved goal, a message that works on the earlier goal is not attributed to the current one, even when the current one is mentioned as a future intention.
- **Ending the conversation.** For `goal_index = -1`, the judge checks that the message is genuinely closing the conversation and is not still pursuing an open goal.

## Two judgements

- **Per message** (feeds Goal Following). The judge identifies each message's main intent, then decides whether that intent counts as attempting the goal at the message's labelled `goal_index`.
- **Per goal** (feeds Goals Attempted). This is a stricter, independent judgement over the whole conversation. A goal counts as attempted only if the user has fully engaged with the underlying need, and not merely brought up the topic. It does not need to line up with the labelled `goal_index`, so a goal can be attempted in a message labelled with a different goal.

## Output

A single JSON object with two lists:

- `message_checks`: one entry per user message, in order, with `id`, `goal_index`, `is_attempting_goal`, and a short `reasoning`.
- `goal_checks`: one entry per goal, with `goal_index`, `is_goal_attempted`, and a short `reasoning` citing the relevant message ids.

Goal Following is the share of `message_checks` with `is_attempting_goal` true. Goals Attempted is the share of `goal_checks` with `is_goal_attempted` true.

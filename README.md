# conv2026-supplementary

Supplementary materials for the paper:

**Configuration Fidelity for LLM-Based User Simulators in Automated Agent Testing**, submitted to [CONVERSATIONS 2026](https://2026.conversations.ws).

## Status

The full materials in this repository are **pending company clearance** and have not been released yet. In the meantime, this repository provides [descriptions of the LLM judge prompts](judge-prompts/), paraphrased from the prompts themselves. We will add the full materials once clearance is granted. Which materials are released, and in what form, depends on that decision.

## LLM judge prompts: descriptions

The descriptions in [`judge-prompts/`](judge-prompts/) cover each prompt's inputs, the judgement it asks for, the main decision rules, and the output format. They are intended to let readers understand and reproduce the approach, not to replace the prompts themselves.

There are five judge prompts. Together they compute the six configuration fidelity metrics in the paper. The judge was GPT-4.1 with a temperature of 0 (see the case study section of the paper).

| Prompt | Metric(s) | Aspect | Description |
|---|---|---|---|
| `check_goal_attempt` | Goal Following, Goals Attempted | Content faithfulness | [goal-attempt.md](judge-prompts/goal-attempt.md) |
| `check_mentioned_facts` | Mentioned Facts Grounded | Content faithfulness | [mentioned-facts.md](judge-prompts/mentioned-facts.md) |
| `check_relevant_facts` | Relevant Facts Used | Content faithfulness | [relevant-facts.md](judge-prompts/relevant-facts.md) |
| `check_ambiguity` | Ambiguities | Style adherence | [ambiguity.md](judge-prompts/ambiguity.md) |
| `check_inconsistencies` | Inconsistencies | Style adherence | [inconsistencies.md](judge-prompts/inconsistencies.md) |

### Common design

- **Templated input.** Each prompt is a template with placeholders that are filled with the run's data: the scenario (or context and goals), and the chat history as JSON Lines, one message per line with an `id` and a `type` (`human` for the user simulator, otherwise the agent under test).
- **Structure.** Every prompt presents the inputs first, then a statement of purpose, then the definitions and decision rules, then worked examples (where present), and finally the output specification.
- **Examples from a different domain.** Where a prompt includes worked examples, they come from an unrelated domain (such as dentist appointments, hotel rooms, or flights), and the prompt tells the judge to apply the same logic regardless of subject matter. This avoids leaking the test domain (order status and returns) into the judge's behaviour.
- **Only user messages are judged.** The judge evaluates only the messages from the user simulator. Agent messages are used as context.
- **Decompose, then decide.** The prompts that extract items (facts, ambiguities, inconsistencies) lay out explicit steps, so the judge enumerates candidates before deciding on each one.
- **Structured output.** Each prompt requires a single JSON object and no other text. Every verdict carries a short `reasoning` field. For the fact prompts the reasoning is requested before the verdict field, so the judge states its evidence first.
- **Counts become metrics.** The metrics are computed from the returned verdicts. For example, Relevant Facts Used is the share of returned `relevant_facts` entries with `used` set to true.
- **Iterative alignment.** The prompts were iteratively aligned against manual judgements from one author on the 8 scenarios, as described in the paper. Appendices on judge reliability report the agreement with human annotators.

## Citation

If you use these materials, please cite the paper. A full citation will be added here once the paper is published.

## Contact

Questions can be raised through [GitHub Issues](../../issues).

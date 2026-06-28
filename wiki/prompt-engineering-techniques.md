# Prompt Engineering Techniques
> Prompt engineering is the iterative process of improving prompts to get more reliable, higher-quality outputs — starting with a basic prompt, evaluating its performance, then systematically applying techniques to raise the score.

## Prompt Engineering ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/15))

Prompt engineering builds directly on prompt evaluation — you use your eval score as a baseline, then apply engineering techniques to move it up.

- **Iterative refinement loop:** write prompt → run eval → review scores and reasoning → apply a technique → re-run eval → compare
- The score alone doesn't matter — what matters is whether a specific change makes it go up or down
- The `prompting/` folder contains `prompting.ipynb` with the full `PromptEvaluator` class for running this loop
- `PromptEvaluator` handles: dataset generation, concurrent test case generation, model-based grading, and HTML report output
- `parse_json()` helper added to strip markdown fences from Claude 4 responses before parsing

> Learning: Prompt engineering without evaluation is guesswork. The eval score gives you a number to move — that's what makes improvement measurable.

## Being Clear and Direct ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/16))

The first line of a prompt sets the tone for everything that follows — being clear and direct here has the highest leverage.

- **Clear:** use simple language, state exactly what you want, no ambiguity
- **Direct:** use action verbs (`Write`, `Create`, `Generate`, `Identify`), not questions
- Lead with: what to do → what to produce → key constraints

Applied to the meal plan prompt: `"What should this person eat?"` → `"Generate a one-day meal plan for an athlete that meets their dietary restrictions"` — eval score jumped from **2.32 → 3.92**.

> Learning: One well-chosen action verb at the start of a prompt can double your eval score before touching anything else.

## Being Specific ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/18))

Specificity gives Claude a clear target — two types of guidelines you can add to any prompt.

- **Output quality guidelines** — control length, format, elements to include, tone (use in almost every prompt)
- **Process steps** — guide Claude to think step-by-step before answering; use for complex reasoning or decision-making tasks
- Combining both gives consistency in results AND confidence Claude considered all angles

Applied to the meal plan prompt: adding 6 specific output guidelines raised eval score from **3.92 → 7.86**.

> Learning: Specificity is the single highest-leverage technique. Output guidelines alone can double your score — add them to every prompt by default.

## Structure with XML Tags ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/19))

XML tags add structure to complex prompts — creating clear boundaries between instructions, data, and different content types.

- Wrap data in descriptive tags: `<sales_records>`, `<athlete_information>`, `<my_code>`, `<docs>`
- Use specific names over generic ones (`<sales_records>` not `<data>`)
- Most valuable when: mixing content types, large data blocks, multiple interpolated variables
- Applied to the meal plan prompt: athlete inputs wrapped in `<athlete_information>` tags for clarity

> Learning: XML tags don't always show dramatic score gains on simple prompts — their power scales with prompt complexity and data volume.

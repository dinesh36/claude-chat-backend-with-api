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

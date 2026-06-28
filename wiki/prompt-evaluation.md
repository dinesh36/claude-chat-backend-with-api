# Prompt Evaluation
> This doc cover building a complete prompt evaluation pipeline — generating test datasets with Claude, running evals programmatically, and grading outputs using both model-based and code-based graders to get a quantitative score for prompt quality.

## Generating Test Datasets ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/11))

Used Claude itself to automatically create a dataset of test cases for prompt evaluation.

- The dataset is a list of JSON objects, each with a `task` field describing an AWS coding task
- Tasks cover three types: Python functions, JSON configs, and regular expressions
- Used the system prompt workaround to get clean JSON output (no markdown or explanations)
- Pretty-printed with `json.dumps(dataset, indent=2)` for readable notebook output
- Saved to `dataset.json` so it can be reused without regenerating every time

> Learning: Generate the dataset once and save it to a file. Re-running Claude each time wastes tokens and makes your baseline inconsistent.

## Running the Eval ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/12))

Built the core evaluation pipeline using three simple functions that work together:

- `run_prompt(test_case)` — takes one test case, builds a prompt from it, sends it to Claude, and returns the response
- `run_test_case(test_case)` — calls `run_prompt`, then returns a result object with `output`, `test_case`, and `score`
- `run_eval(dataset)` — loops through all test cases, runs each one, and returns the full list of results

The score is hardcoded to `10` for now — real grading logic will replace this in the next step.

> Learning: Build the pipeline loop first, then worry about grading. Getting the plumbing right before adding complexity makes it easier to debug.

## Model Based Grading ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/13))

Replaced the hardcoded score placeholder with a real model grader using a second Claude call.

- `grade_by_model()` sends the original task and Claude's output to Claude for evaluation
- Returns a JSON object with `strengths`, `weaknesses`, `reasoning`, and a `score` (1–10)
- Applied the Claude 4 fix — system prompt instead of assistant prefill for raw JSON output
- `run_eval()` now calculates and prints an average score across all test cases
- Always ask for reasoning alongside the score — without it, models default to middling scores around 6

> Learning: The average score is your baseline. Change the prompt, re-run, compare — if the number goes up, the change helped.

## Code Based Grading ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/14))

Added syntax validation alongside the model grader to catch format and parse errors programmatically.

- Three validator functions: `validate_json` (uses `json.loads`), `validate_python` (uses `ast.parse`), `validate_regex` (uses `re.compile`)
- Each returns 10 (valid syntax) or 0 (invalid) — no partial credit
- Dataset updated to include a `"format"` field (`"python"`, `"json"`, or `"regexp"`) so the right validator is picked per task
- `run_test_case()` now averages model score and syntax score: `score = (model_score + syntax_score) / 2`
- The course also suggests `add_assistant_message(messages, "```code")` to enforce raw output — replaced with system prompt for Claude 4 compatibility

> Learning: Combining a code grader (did it parse?) with a model grader (did it solve the task?) gives a more complete picture than either alone.

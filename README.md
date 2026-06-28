# Claude API Learning Journey

A step-by-step learning repo following the [Building with the Claude API](https://anthropic.skilljar.com/claude-with-the-anthropic-api) course — each step documented with a PR link and a brief summary of what was learned.

## Learning Steps

### Step 1 — First API Request with Claude ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/2))

Set up the project and made the first working API call to Claude:
- Installed `anthropic` and `python-dotenv`; secured `ANTHROPIC_API_KEY` in `.env` (gitignored)
- Initialised the Anthropic client and called `client.messages.create()` with `model`, `max_tokens`, and `messages`
- Sent the prompt *"What is quantum computing? Answer in one sentence"* and extracted the response via `message.content[0].text`
- `chat.ipynb` contains the full working notebook

> Learning: API keys must never be committed — always load them from `.env`. The `max_tokens` param is a safety ceiling, not a target length.

### Step 2 — Multi-Turn Conversations ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/3))

Refactored the notebook to support multi-turn conversations:
- Claude is **stateless** — it has no memory of previous messages between requests
- To maintain context, you must manually keep a `messages` list and send the full history with every API call
- Added `add_user_message()`, `add_assistant_message()`, and `chat()` helper functions to manage conversation state cleanly

> Learning: Always append Claude's response back into the message list as an `assistant` message before sending the next user message — otherwise Claude loses all context.

### Step 3 — Chat Exercise: Continuous Chat Loop ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/4))

Built a live terminal chat session using a `while True` loop with the helper functions from Step 2:
- Each user input is added to `messages`, Claude is called with the full history, and its reply is appended back before the next turn
- Verified context is preserved across turns (e.g. *"add 4 in that"* correctly referenced the previous answer)
- Discovered that submitting empty input raises `BadRequestError: user messages must have non-empty content` — need to guard with `if not user_input.strip(): continue`

> Learning: The API rejects empty user messages — always validate input before sending.

### Step 4 — System Prompts ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/5))

Learned how to use system prompts to shape Claude's role and behaviour:
- Pass a plain string via the `system` parameter to define Claude's persona and constraints
- The same question gets a completely different response with vs. without a system prompt (e.g. math tutor gives hints instead of direct answers)
- Updated `chat()` to accept an optional `system` arg — conditionally included since the API rejects `system=None`

> Learning: System prompts are the primary tool for building specialised AI assistants — they define the role, tone, and constraints before any user message.

### Step 5 — System Prompts Exercise ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/6))

Applied a one-sentence system prompt to control Claude's coding style:
- **Without system prompt:** Claude returned a verbose multi-function response with docstrings, comparison tables, and full examples
- **With** `"You are a Python engineer who writes very concise code"`: Claude returned a single 1-line function — `return len(s) != len(set(s))`

> Learning: A one-sentence system prompt is often enough — it controls the *style and scope* of the response, not just the role.

### Step 6 — Temperature ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/7))

Added `temperature` (0.0–1.0) as an optional param to `chat()` and verified its effect with a movie idea generation demo:
- **temperature=0.0** — near-identical "retired safecracker" premise across 3 runs
- **temperature=1.0** — completely different ideas each time (lighthouse keeper, muscle memory heist, granddaughter rescue)

> Learning: Temperature doesn't guarantee different outputs — it changes the *probability* of getting them. Match it to your use case: low for factual/coding tasks, high for creative work.

### Step 7 — Response Streaming ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/8))

Implemented all three streaming patterns to deliver Claude's output word by word instead of waiting for the full response:
- `stream=True` on `messages.create()` — exposes raw events (`ContentBlockDelta` carries text chunks)
- `client.messages.stream()` with `text_stream` — SDK-simplified text-only iteration, no event parsing
- `get_final_message()` — assembled `Message` object after streaming for storage/logic

> Learning: Use `text_stream` for display, `get_final_message()` for storage — best of both worlds in a single request.

### Step 8 — Structured Data ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/9))

Forced Claude to return raw structured output (JSON) without markdown wrappers or explanatory prose:
- **Course technique:** assistant message prefill + stop sequences — makes Claude think it already started a code block, then stops before it closes
- **Claude 4 limitation:** `claude-sonnet-4-6` rejects conversations ending with an assistant message (`invalid_request_error: assistant message prefill not supported`)
- **Workaround:** system prompt `"Output raw JSON only. No markdown, no explanation, no code fences."` — works on all models
- Also added `stop_sequences` param to `chat()` for completeness

> Learning: Claude 4 models don't support assistant prefill at all — use a system prompt to control output format instead.

### Step 9 — Structured Data Exercise ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/10))

Practised getting clean structured output from Claude — this time generating raw AWS CLI bash commands.

- The course technique uses assistant prefill (` ```bash `) + stop sequence (` ``` `) to strip markdown wrappers
- This fails on Claude 4 with the same prefill error as Step 8
- Fix: use a system prompt instead — `"Output raw bash commands only, one per line, no numbering, no markdown, no explanation."`

> Learning: Whenever you see `add_assistant_message` being used to control output format, replace it with a system prompt — that's the Claude 4 compatible way.

### Step 10 — Prompt Evaluation: Generating Test Datasets ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/11))

Used Claude itself to automatically create a dataset of test cases for prompt evaluation.

- The dataset is a list of JSON objects, each with a `task` field describing an AWS coding task
- Tasks cover three types: Python functions, JSON configs, and regular expressions
- Used the system prompt workaround to get clean JSON output (no markdown or explanations)
- Pretty-printed with `json.dumps(dataset, indent=2)` for readable notebook output
- Saved to `dataset.json` so it can be reused without regenerating every time

> Learning: Generate the dataset once and save it to a file. Re-running Claude each time wastes tokens and makes your baseline inconsistent.

### Step 11 — Prompt Evaluation: Running the Eval ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/12))

Built the core evaluation pipeline using three simple functions that work together:

- `run_prompt(test_case)` — takes one test case, builds a prompt from it, sends it to Claude, and returns the response
- `run_test_case(test_case)` — calls `run_prompt`, then returns a result object with `output`, `test_case`, and `score`
- `run_eval(dataset)` — loops through all test cases, runs each one, and returns the full list of results

The score is hardcoded to `10` for now — real grading logic will replace this in the next step.

> Learning: Build the pipeline loop first, then worry about grading. Getting the plumbing right before adding complexity makes it easier to debug.

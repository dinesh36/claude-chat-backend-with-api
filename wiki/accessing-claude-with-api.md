# Accessing Claude with API
> Here we cover learning from making the first request and maintaining conversation state, to shaping Claude's behaviour with system prompts, controlling output randomness with temperature, streaming responses, and extracting clean structured data.

## First API Request with Claude ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/2))

Set up the project and made the first working API call to Claude:
- Installed `anthropic` and `python-dotenv`; secured `ANTHROPIC_API_KEY` in `.env` (gitignored)
- Initialised the Anthropic client and called `client.messages.create()` with `model`, `max_tokens`, and `messages`
- Sent the prompt *"What is quantum computing? Answer in one sentence"* and extracted the response via `message.content[0].text`
- `chat.ipynb` contains the full working notebook

> Learning: API keys must never be committed — always load them from `.env`. The `max_tokens` param is a safety ceiling, not a target length.

## Multi-Turn Conversations ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/3))

Refactored the notebook to support multi-turn conversations:
- Claude is **stateless** — it has no memory of previous messages between requests
- To maintain context, you must manually keep a `messages` list and send the full history with every API call
- Added `add_user_message()`, `add_assistant_message()`, and `chat()` helper functions to manage conversation state cleanly

> Learning: Always append Claude's response back into the message list as an `assistant` message before sending the next user message — otherwise Claude loses all context.

## Chat Exercise: Continuous Chat Loop ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/4))

Built a live terminal chat session using a `while True` loop with the helper functions from the previous step:
- Each user input is added to `messages`, Claude is called with the full history, and its reply is appended back before the next turn
- Verified context is preserved across turns (e.g. *"add 4 in that"* correctly referenced the previous answer)
- Discovered that submitting empty input raises `BadRequestError: user messages must have non-empty content` — need to guard with `if not user_input.strip(): continue`

> Learning: The API rejects empty user messages — always validate input before sending.

## System Prompts ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/5))

Learned how to use system prompts to shape Claude's role and behaviour:
- Pass a plain string via the `system` parameter to define Claude's persona and constraints
- The same question gets a completely different response with vs. without a system prompt (e.g. math tutor gives hints instead of direct answers)
- Updated `chat()` to accept an optional `system` arg — conditionally included since the API rejects `system=None`

> Learning: System prompts are the primary tool for building specialised AI assistants — they define the role, tone, and constraints before any user message.

## System Prompts Exercise ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/6))

Applied a one-sentence system prompt to control Claude's coding style:
- **Without system prompt:** Claude returned a verbose multi-function response with docstrings, comparison tables, and full examples
- **With** `"You are a Python engineer who writes very concise code"`: Claude returned a single 1-line function — `return len(s) != len(set(s))`

> Learning: A one-sentence system prompt is often enough — it controls the *style and scope* of the response, not just the role.

## Temperature ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/7))

Added `temperature` (0.0–1.0) as an optional param to `chat()` and verified its effect with a movie idea generation demo:
- **temperature=0.0** — near-identical "retired safecracker" premise across 3 runs
- **temperature=1.0** — completely different ideas each time (lighthouse keeper, muscle memory heist, granddaughter rescue)

> Learning: Temperature doesn't guarantee different outputs — it changes the *probability* of getting them. Match it to your use case: low for factual/coding tasks, high for creative work.

## Response Streaming ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/8))

Implemented all three streaming patterns to deliver Claude's output word by word instead of waiting for the full response:
- `stream=True` on `messages.create()` — exposes raw events (`ContentBlockDelta` carries text chunks)
- `client.messages.stream()` with `text_stream` — SDK-simplified text-only iteration, no event parsing
- `get_final_message()` — assembled `Message` object after streaming for storage/logic

> Learning: Use `text_stream` for display, `get_final_message()` for storage — best of both worlds in a single request.

## Structured Data ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/9))

Forced Claude to return raw structured output (JSON) without markdown wrappers or explanatory prose:
- **Course technique:** assistant message prefill + stop sequences — makes Claude think it already started a code block, then stops before it closes
- **Claude 4 limitation:** `claude-sonnet-4-6` rejects conversations ending with an assistant message (`invalid_request_error: assistant message prefill not supported`)
- **Workaround:** system prompt `"Output raw JSON only. No markdown, no explanation, no code fences."` — works on all models
- Also added `stop_sequences` param to `chat()` for completeness

> Learning: Claude 4 models don't support assistant prefill at all — use a system prompt to control output format instead.

## Structured Data Exercise ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/10))

Practised getting clean structured output from Claude — this time generating raw AWS CLI bash commands.

- The course technique uses assistant prefill (` ```bash `) + stop sequence (` ``` `) to strip markdown wrappers
- This fails on Claude 4 with the same prefill error as the previous step
- Fix: use a system prompt instead — `"Output raw bash commands only, one per line, no numbering, no markdown, no explanation."`

> Learning: Whenever you see `add_assistant_message` being used to control output format, replace it with a system prompt — that's the Claude 4 compatible way.

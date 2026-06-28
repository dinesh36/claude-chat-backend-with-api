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

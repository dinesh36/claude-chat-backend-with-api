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

# Claude API Learning Journey

A step-by-step learning repo following the [Building with the Claude API](https://anthropic.skilljar.com/claude-with-the-anthropic-api) course — each step documented with a PR link and a brief summary of what was learned.

## Learning Steps

### Step 1 — Project Setup & Security ([PR #1](https://github.com/dinesh36/claude-chat-backend-with-api/pull/1))

Added `.gitignore` to protect sensitive files from being committed:
- `.env` / `.env.*` — keeps the `ANTHROPIC_API_KEY` out of version control
- `chat.ipynb` — excludes local experiment notebooks from the repo

> Learning: Always set up `.gitignore` before committing anything to avoid accidentally leaking API keys.

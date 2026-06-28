# Tool Use with Claude
> Extend Claude beyond its training data by giving it Python functions it can call to access real-time information, perform date calculations, and trigger actions in external systems.

## Tool Functions ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/22))

Tool functions are plain Python functions Claude calls automatically when it needs data beyond its training — the core building block of tool use.

- Write regular Python functions; Claude decides when to call them based on user requests
- **Validate inputs** and raise descriptive errors — Claude reads errors and may self-correct and retry
- Use descriptive function and parameter names so Claude understands the tool's purpose
- First tool built: `get_current_datetime(date_format)` — returns current time formatted via `strftime`
- Creating the function is step one; next comes the JSON schema that describes it to Claude

> Learning: Claude learns from error messages — a clear `ValueError` lets it retry with corrected arguments rather than failing silently.

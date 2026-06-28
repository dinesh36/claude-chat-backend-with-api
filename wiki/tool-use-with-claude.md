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

## Tool Schemas ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/23))

A JSON schema is the documentation Claude reads to understand when and how to call your tool — every tool spec needs three fields: `name`, `description`, and `input_schema`.

- `description` is the most important field — Claude uses it to decide whether to call the tool; aim for 3–4 sentences covering what it does, when to use it, and what it returns
- `input_schema` follows standard JSON Schema spec; describe each argument's type, purpose, and default
- Name schemas `function_name_schema` to keep them easy to match with their function
- Let Claude generate schemas for you — paste your function + Anthropic docs and ask for a valid tool schema
- Use `ToolParam` from `anthropic.types` for type safety when passing schemas to the API

> Learning: The description field is what Claude actually reads to decide when to call a tool — a vague description leads to missed or incorrect tool calls.

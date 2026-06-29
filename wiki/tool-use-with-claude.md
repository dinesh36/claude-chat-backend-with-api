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

## Handling Message Blocks ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/24))

When Claude uses a tool, it returns a multi-block `response.content` list — not a simple text string — containing a TextBlock and a ToolUseBlock.

- Pass tool schemas via the `tools` parameter in `client.messages.create()`
- **TextBlock** — human-readable explanation of what Claude is doing
- **ToolUseBlock** — `id`, `name` (function to call), `input` (params dict), `type: "tool_use"`
- Always append `response.content` (not just the text) to conversation history — dropping either block breaks context
- Update any `add_assistant_message()` helpers to accept multi-block content, not just strings
- Full flow: send message + schema → receive TextBlock + ToolUseBlock → execute function → send result back → receive final response

> Learning: Appending `response.content` directly (not extracting just the text) is the correct way to preserve multi-block conversation history.

## Sending Tool Results ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/25))

After Claude returns a ToolUseBlock, execute the function and send back a `tool_result` user message to complete the round-trip.

- Unpack Claude's requested args with `**response.content[1].input` and call your function
- Wrap the result in a `tool_result` block with `tool_use_id` (must match ToolUseBlock `id`), `content` (string), and `is_error`
- Claude can request multiple tool calls in one response — each has a unique ID, match them when sending results back
- Always include the tool schema in the follow-up API call — Claude needs it to understand tool references already in the history
- Full message history: user message → assistant (TextBlock + ToolUseBlock) → user (tool_result) → final assistant response

> Learning: Always pass `tools=[...]` on the follow-up call even when you don't expect another tool use — without it Claude can't parse its own prior tool references in the history.

## Multi-Turn Conversations with Tools ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/26))

When a question requires multiple tools in sequence, a `while True` conversation loop keeps calling Claude and executing tools until no more ToolUseBlocks are returned.

- Loop pattern: `chat()` → append assistant message → check for tool use → `run_tools()` → append tool results → repeat
- Refactor `add_user_message` / `add_assistant_message` to accept strings, block lists, or full `Message` objects (`isinstance(message, Message)`)
- Update `chat()` to accept optional `tools` list and return the full message object (not just text)
- Add `text_from_message()` helper: joins all TextBlocks for display — `[block.text for block in message.content if block.type == "text"]`
- Break condition: response contains no ToolUseBlock → Claude has all the info it needs

> Learning: The conversation loop is the core of any multi-tool agent — keep it running until Claude stops asking for tools, then extract the final text.

## Implementing Multiple Turns ([Changes](https://github.com/dinesh36/claude-chat-backend-with-api/pull/27))

The complete `run_conversation()` loop — uses `stop_reason == "tool_use"` to detect whether to continue, executes all tool blocks, and sends results back until Claude has a final answer.

- Break condition: `response.stop_reason != "tool_use"` — clean signal Claude is done requesting tools
- `run_tools()` filters `message.content` for all `tool_use` blocks and processes each with matching `tool_use_id`
- Error handling: always send a `tool_result` block even on failure (`is_error: True`) so Claude can respond gracefully
- `run_tool()` routing function maps tool names to implementations — add new tools here without touching the loop
- Serialize tool output with `json.dumps()` before putting it in the `content` field

> Learning: Never skip sending a tool_result block on error — Claude expects a result for every tool it requested, and omitting one breaks the conversation.

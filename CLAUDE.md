# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A step-by-step learning repo following the [Building with the Claude API](https://anthropic.skilljar.com/claude-with-the-anthropic-api) course. Each step is documented in a wiki and implemented in Jupyter notebooks.

## Setup

```bash
pip install anthropic python-dotenv
```

Place `ANTHROPIC_API_KEY=<your-key>` in `.env` (already gitignored). The notebooks load it with `load_dotenv()`.

## Running Notebooks

```bash
jupyter notebook
```

- `chat.ipynb` — core API patterns (steps 1–13): first request, multi-turn, system prompts, temperature, streaming, structured data, prompt evaluation
- `prompting/prompting.ipynb` — prompt engineering loop using the `PromptEvaluator` class (steps 14+)

Run cells top-to-bottom within each notebook; state is shared within a session.

## Architecture

### Two notebooks, one progression

`chat.ipynb` establishes all foundational helpers (`add_user_message`, `add_assistant_message`, `chat()`) and builds a simple eval pipeline (`run_prompt` → `run_test_case` → `run_eval`). `prompting/prompting.ipynb` imports those same patterns but replaces the ad-hoc eval with the full `PromptEvaluator` class.

### PromptEvaluator (`prompting/prompting.ipynb`)

The central class for the prompt engineering workflow:
- `generate_dataset()` — uses Claude to generate diverse test cases concurrently (`ThreadPoolExecutor`), saves to `dataset.json`
- `run_evaluation()` — runs each test case through `run_prompt_function`, grades with `grade_output()`, writes `output.json` and `output.html`
- `grade_output()` — second Claude call that returns structured JSON (`strengths`, `weaknesses`, `reasoning`, `score`)
- `render()` — simple `{placeholder}` template engine used to build prompts from test case inputs

The eval loop is: write prompt → run `run_evaluation()` → check average score → tweak prompt → repeat.

### Dataset files

`dataset.json` (both root and `prompting/`) stores generated test cases so they can be reused without re-calling Claude. `prompting/output.json` and `prompting/output.html` are the last eval run results.

## Claude 4 Compatibility

**Assistant message prefill is not supported on Claude 4 models.** The course material uses `add_assistant_message(messages, "```json")` + stop sequences to force structured output — this raises `invalid_request_error` on `claude-sonnet-4-6` and `claude-haiku-4-5`.

The fix used throughout this repo: pass a system prompt instead.
```python
# Instead of prefill + stop_sequences:
chat(messages, system="Output raw JSON only. No markdown, no explanation, no code fences.")
```

A `parse_json()` helper in `prompting/prompting.ipynb` also strips any stray markdown fences before `json.loads()` as a secondary safeguard.

## Models

- `chat.ipynb` uses `claude-sonnet-4-6`
- `prompting/prompting.ipynb` uses `claude-haiku-4-5` (cheaper for the high-volume eval loop)

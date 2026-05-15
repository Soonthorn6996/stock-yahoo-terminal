# CLAUDE.md — Claude-Specific Guide for Stock Yahoo

This file extends [AGENTS.md](./AGENTS.md) with instructions specific to Claude Code and the Anthropic API integration in this project.

---

## Claude Code Usage

This repo is designed to be worked on with **Claude Code** (`claude` CLI). Before starting a session:

```bash
# Start from repo root
cd stock-yahoo

# Confirm backend is running
cd apps/api && uvicorn app.main:app --reload --port 8000

# Confirm frontend is running
cd apps/web && pnpm dev
```

---

## Anthropic API Integration

The Anthropic API is used in `apps/api/app/services/ai_service.py` for:

1. **Stock summary** — Summarizes a stock's key metrics and recent performance into a human-readable paragraph
2. **AI analysis** — Provides a brief AI-generated outlook given stock data (price, PE ratio, 52-week range, etc.)

### Configuration

```env
# apps/api/.env
ANTHROPIC_API_KEY="sk-ant-..."
ANTHROPIC_MODEL="claude-sonnet-4-20250514"
```

### Model Usage Pattern

```python
# apps/api/app/services/ai_service.py
import anthropic
import os

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env

async def summarize_stock(symbol: str, stock_data: dict) -> str:
    prompt = f"""
    Given the following stock data for {symbol}, write a 2-3 sentence summary
    for a retail investor. Be factual and concise.

    Data: {stock_data}
    """

    message = await asyncio.to_thread(
        client.messages.create,
        model=os.getenv("ANTHROPIC_MODEL", "claude-sonnet-4-20250514"),
        max_tokens=300,
        messages=[{"role": "user", "content": prompt}],
    )

    return message.content[0].text.strip()
```

---

## Claude Code Behavior Guidelines

### Preferred patterns
- Use `pytest tests/test_<module>.py` to run specific test files
- Check types: `mypy app/` before finishing backend changes
- Check frontend types: `pnpm tsc --noEmit` in `apps/web/`
- Always run `pnpm lint` after frontend edits

### Do not
- Do not call `client.messages.create` outside of `ai_service.py`
- Do not hardcode the model string — always use `os.getenv("ANTHROPIC_MODEL", "claude-sonnet-4-20250514")`
- Do not call Anthropic API from `apps/web` — proxy through FastAPI only

---

## Common Claude Code Tasks

### Add a new AI-powered feature
1. Add function to `apps/api/app/services/ai_service.py`
2. Add endpoint in `apps/api/app/routers/ai.py` (create if absent)
3. Add `Depends(get_current_user)` auth guard
4. Add new env vars to `apps/api/.env.example`
5. Add TypeScript client call in `apps/web/src/lib/api.ts`

### Debug Anthropic API issues
```bash
# Verify key is loaded
cd apps/api
python -c "import os; from dotenv import load_dotenv; load_dotenv(); print(os.getenv('ANTHROPIC_API_KEY', '')[:10])"

# Run AI service tests
pytest tests/test_ai_service.py -v
```

### Debug Supabase Auth issues
```bash
# Test JWT verification locally
cd apps/api
pytest tests/test_auth.py -v
```

---

## MCP Server (Claude Desktop)

To connect Claude Desktop to this project via MCP:

```json
{
  "mcpServers": {
    "stock-yahoo-docs": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-filesystem", "/path/to/stock-yahoo"],
      "env": {}
    }
  }
}
```

---

## Related Docs

- [AGENTS.md](./AGENTS.md) — general AI agent guide (read first)
- [API.md](./API.md) — endpoint reference including `/api/ai/*` routes
- [SECURITY.md](./SECURITY.md) — API key handling and secrets management

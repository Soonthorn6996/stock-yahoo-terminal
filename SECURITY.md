# SECURITY.md — Security Guide for Stock Yahoo

## Authentication & Session Management

Stock Yahoo delegates all authentication to **Supabase Auth**. Users log in via the Next.js frontend using the Supabase JS SDK, which returns a JWT access token and a refresh token managed by Supabase.

| Token | Storage | Lifetime | Purpose |
|-------|---------|----------|---------|
| Access token | Supabase session (memory / cookie) | 1 hour (Supabase default) | Authenticates API requests |
| Refresh token | Supabase session | 7 days | Issues new access tokens |

The FastAPI backend **verifies** the JWT on every protected request using the `get_current_user` dependency. It does not issue its own tokens.

---

## Authorization

All protected FastAPI routes use the `get_current_user` dependency:

```python
from app.middleware.auth import get_current_user
from fastapi import Depends

@router.get("/stocks/{symbol}")
async def get_stock(symbol: str, user_id: str = Depends(get_current_user)):
    ...
```

Unauthenticated requests receive `401 Unauthorized`.

### Supabase Row Level Security (RLS)

Database-level authorization is enforced via Supabase RLS policies. Users can only read and write their own rows in `watchlist` and `search_history`. See [DATABASE.md](./DATABASE.md) for policy definitions.

---

## Secrets Management

### Development

Secrets live in `.env` / `.env.local` files (gitignored). Copy from `.env.example`:

```bash
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.local.example apps/web/.env.local
```

**Never commit `.env` or `.env.local` files.**

### Production (Vercel + CI)

| Variable | Where | Description |
|----------|-------|-------------|
| `NEXT_PUBLIC_SUPABASE_URL` | Vercel env | Supabase project URL (public) |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Vercel env | Supabase anon key (public, safe for frontend) |
| `SUPABASE_URL` | API host env | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | API host env | **Server-side only** — full DB access, never expose |
| `ANTHROPIC_API_KEY` | API host env | Anthropic Claude API key |

The `SUPABASE_SERVICE_ROLE_KEY` bypasses RLS — it must **never** be sent to or used in the frontend.

---

## Input Validation

All FastAPI request bodies are validated with **Pydantic v2** schemas. Invalid input returns `422` before reaching service or database code.

```python
# apps/api/app/models/schemas.py
from pydantic import BaseModel, field_validator
import re

class WatchlistAddRequest(BaseModel):
    symbol: str

    @field_validator("symbol")
    @classmethod
    def validate_symbol(cls, v: str) -> str:
        v = v.upper().strip()
        if not re.match(r'^[A-Z0-9.\-]{1,10}$', v):
            raise ValueError("Invalid ticker symbol format")
        return v
```

---

## CORS Configuration

CORS is configured in `apps/api/app/main.py`:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[os.getenv("ALLOWED_ORIGINS", "http://localhost:3000")],
    allow_credentials=True,
    allow_methods=["GET", "POST", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
```

In production, `ALLOWED_ORIGINS` is set to `https://stock-yahoo.vercel.app` (or your custom domain).

---

## Rate Limiting

Auth and AI endpoints are rate-limited using `slowapi` to mitigate abuse:

- Default endpoints: 60 requests/min per user
- AI endpoints: 10 requests/min per user

---

## Data Sensitivity

Stock Yahoo does not process payment data or health records. PII stored:
- User email addresses (managed by Supabase Auth)
- Stock watchlist and search history

No third-party analytics that receive PII.

---

## Yahoo Finance Proxy

The frontend **never** calls Yahoo Finance directly. All Yahoo Finance requests are proxied through FastAPI. This prevents API key exposure and allows server-side caching and rate limiting.

---

## Reporting Vulnerabilities

Report security issues privately to `security@your-org.com`. Do not open a public GitHub issue for security vulnerabilities.

Response SLA: acknowledgement within 48 hours.

---

## Security Checklist for New Features

- [ ] All new routes protected with `Depends(get_current_user)` unless explicitly public
- [ ] Request bodies validated with Pydantic
- [ ] No user-controlled values interpolated into SQL (use Supabase parameterized queries)
- [ ] New env vars added to `.env.example` (value redacted)
- [ ] `SUPABASE_SERVICE_ROLE_KEY` not used or referenced in `apps/web/`
- [ ] New AI endpoints added to rate-limit config

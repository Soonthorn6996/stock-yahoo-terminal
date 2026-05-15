# API.md — REST API Reference for Stock Yahoo

## Base URL

| Environment | URL |
|-------------|-----|
| Development | `http://localhost:8000` |
| Production  | `https://api.stock-yahoo.app` (replace with actual) |

All endpoints are prefixed with `/api`. Interactive docs available at `/docs` (Swagger UI) and `/redoc`.

---

## Authentication

Stock Yahoo uses **Supabase Auth JWTs**. After login via the frontend, include the Bearer token in every protected request:

```
Authorization: Bearer <supabase_access_token>
```

The token is automatically injected by the Axios client in `apps/web/src/lib/api.ts`.

---

## Endpoints

### Health

```
GET /api/health
```

**Response 200:**
```json
{ "status": "ok" }
```

---

### Auth (handled by Supabase on frontend)

Auth (login, register, logout, password reset) is handled directly by the **Supabase Auth SDK** on the frontend — no custom auth endpoints in FastAPI. The FastAPI backend only *verifies* tokens.

---

### Stocks

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/stocks/{symbol}` | ✅ | Get stock detail by ticker symbol |
| GET | `/api/stocks/{symbol}/history` | ✅ | Get historical price data |

---

#### Get Stock Detail

```
GET /api/stocks/{symbol}
```

Fetches real-time stock data from Yahoo Finance for the given ticker symbol.

**Path params:**
- `symbol` — Ticker symbol, e.g. `AAPL`, `MSFT`, `PTT.BK`

**Response 200:**
```json
{
  "symbol": "AAPL",
  "name": "Apple Inc.",
  "price": 189.30,
  "change": 1.25,
  "change_percent": 0.66,
  "volume": 52341000,
  "market_cap": 2930000000000,
  "pe_ratio": 29.8,
  "week_52_high": 199.62,
  "week_52_low": 164.08,
  "currency": "USD",
  "exchange": "NASDAQ",
  "fetched_at": "2026-05-15T10:30:00Z"
}
```

**Errors:** `401` unauthorized, `404` symbol not found, `502` Yahoo Finance unavailable

---

#### Get Historical Price

```
GET /api/stocks/{symbol}/history?period=1mo&interval=1d
```

**Query params:**

| Param | Default | Options | Description |
|-------|---------|---------|-------------|
| `period` | `1mo` | `1d`, `5d`, `1mo`, `3mo`, `6mo`, `1y`, `5y` | Time period |
| `interval` | `1d` | `1m`, `5m`, `1h`, `1d`, `1wk`, `1mo` | Data interval |

**Response 200:**
```json
{
  "symbol": "AAPL",
  "period": "1mo",
  "interval": "1d",
  "data": [
    { "date": "2026-04-15", "open": 172.10, "high": 174.30, "low": 171.50, "close": 173.80, "volume": 48200000 }
  ]
}
```

---

### Watchlist

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/watchlist` | ✅ | Get current user's watchlist |
| POST | `/api/watchlist` | ✅ | Add symbol to watchlist |
| DELETE | `/api/watchlist/{symbol}` | ✅ | Remove symbol from watchlist |

**Add to watchlist body:**
```json
{ "symbol": "AAPL" }
```

---

### AI Features

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/ai/summarize` | ✅ | Generate AI summary of a stock |

**Request body:**
```json
{
  "symbol": "AAPL",
  "stock_data": {
    "price": 189.30,
    "pe_ratio": 29.8,
    "week_52_high": 199.62,
    "week_52_low": 164.08
  }
}
```

**Response 200:**
```json
{
  "symbol": "AAPL",
  "summary": "Apple is currently trading at $189.30, near the middle of its 52-week range..."
}
```

---

## Error Format

FastAPI returns errors in this shape:

```json
{
  "detail": "Symbol ZZZZ not found on Yahoo Finance"
}
```

For validation errors (422):
```json
{
  "detail": [
    {
      "loc": ["body", "symbol"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

### Common HTTP Status Codes

| HTTP | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 401 | Missing or invalid JWT |
| 403 | Forbidden |
| 404 | Resource not found |
| 422 | Validation error (Pydantic) |
| 502 | Yahoo Finance upstream error |
| 500 | Unexpected server error |

---

## Rate Limiting

- **Default:** 60 requests / minute per user (enforced via `slowapi`)
- **AI endpoints:** 10 requests / minute per user

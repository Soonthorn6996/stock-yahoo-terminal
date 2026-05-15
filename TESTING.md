# TESTING.md — Testing Guide for Stock Yahoo

## Test Stack

| Layer | Tool | Location |
|-------|------|----------|
| Unit / integration (API) | Pytest | `apps/api/tests/` |
| Unit (Frontend) | Jest + React Testing Library | `apps/web/src/**/*.test.tsx` |
| E2E | Playwright | `e2e/**/*.spec.ts` |
| API contract | HTTPX + Pytest | `apps/api/tests/test_routers/` |

---

## Running Tests

```bash
# Backend — all tests
cd apps/api
pytest

# Backend — with coverage
pytest --cov=app --cov-report=html

# Backend — single file
pytest tests/test_services/test_yahoo_service.py -v

# Frontend — Jest unit tests
cd apps/web && pnpm test

# E2E (requires dev servers on ports 3000 + 8000)
pnpm test:e2e
```

---

## Test Structure

### Backend (Pytest)

```
apps/api/
├── app/
│   ├── services/
│   │   ├── yahoo_service.py
│   │   └── ai_service.py
│   └── routers/
│       └── stocks.py
└── tests/
    ├── conftest.py                  ← shared fixtures (app client, mock user)
    ├── test_services/
    │   ├── test_yahoo_service.py
    │   └── test_ai_service.py
    └── test_routers/
        ├── test_stocks.py
        └── test_watchlist.py
```

### Frontend (Jest)

```
apps/web/src/
├── components/
│   ├── StockCard.tsx
│   └── StockCard.test.tsx          ← co-located with component
└── hooks/
    ├── useWatchlist.ts
    └── useWatchlist.test.ts
```

### E2E (Playwright)

```
e2e/
├── auth.spec.ts                    ← login, logout, protected route redirect
├── stocks.spec.ts                  ← search and view stock detail
└── watchlist.spec.ts               ← add/remove watchlist items
```

---

## Writing Backend Tests

### Service test (with mock)

```python
# tests/test_services/test_yahoo_service.py
import pytest
from unittest.mock import patch, MagicMock
from app.services.yahoo_service import get_stock_detail

@pytest.mark.asyncio
async def test_get_stock_detail_returns_price():
    mock_ticker = MagicMock()
    mock_ticker.info = {
        "shortName": "Apple Inc.",
        "currentPrice": 189.30,
        "regularMarketChange": 1.25,
        "regularMarketChangePercent": 0.66,
        "volume": 52341000,
        "trailingPE": 29.8,
        "fiftyTwoWeekHigh": 199.62,
        "fiftyTwoWeekLow": 164.08,
        "currency": "USD",
        "exchange": "NMS",
    }

    with patch("app.services.yahoo_service.yf.Ticker", return_value=mock_ticker):
        result = await get_stock_detail("AAPL")

    assert result["symbol"] == "AAPL"
    assert result["price"] == 189.30
    assert result["name"] == "Apple Inc."


@pytest.mark.asyncio
async def test_get_stock_detail_raises_404_for_unknown_symbol():
    mock_ticker = MagicMock()
    mock_ticker.info = {}  # yfinance returns empty dict for bad symbols

    with patch("app.services.yahoo_service.yf.Ticker", return_value=mock_ticker):
        with pytest.raises(Exception):  # should raise HTTPException 404
            await get_stock_detail("ZZZZZZ")
```

### Router integration test (HTTPX)

```python
# tests/test_routers/test_stocks.py
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.fixture
def auth_headers():
    # Use a real or mock Supabase JWT for testing
    return {"Authorization": "Bearer test-token"}

@pytest.mark.asyncio
async def test_get_stock_requires_auth():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/api/stocks/AAPL")
    assert response.status_code == 401

@pytest.mark.asyncio
async def test_get_stock_returns_detail(auth_headers, mocker):
    mocker.patch("app.middleware.auth.get_current_user", return_value="user-uuid")
    mocker.patch("app.services.yahoo_service.get_stock_detail", return_value={
        "symbol": "AAPL", "price": 189.30, "name": "Apple Inc."
    })

    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/api/stocks/AAPL", headers=auth_headers)

    assert response.status_code == 200
    assert response.json()["symbol"] == "AAPL"
```

---

## Writing E2E Tests

```typescript
// e2e/stocks.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Stock detail", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/login");
    await page.fill('[name="email"]', "test@example.com");
    await page.fill('[name="password"]', "password123");
    await page.click('button[type="submit"]');
    await page.waitForURL("/dashboard");
  });

  test("can search and view stock detail for AAPL", async ({ page }) => {
    await page.fill('[placeholder="Search symbol..."]', "AAPL");
    await page.press('[placeholder="Search symbol..."]', "Enter");
    await page.waitForURL("/stocks/AAPL");

    await expect(page.locator("text=Apple Inc.")).toBeVisible();
    await expect(page.locator("[data-testid=stock-price]")).toBeVisible();
  });
});
```

---

## Test Environment

Backend tests use a **test Supabase project** or mock the Supabase client. Configure in `apps/api/.env.test`:

```env
SUPABASE_URL="https://test-project.supabase.co"
SUPABASE_SERVICE_ROLE_KEY="test-service-role-key"
ANTHROPIC_API_KEY="test-key"
```

The `conftest.py` loads `.env.test` automatically via `pytest-dotenv`.

---

## Coverage Requirements

CI enforces minimum coverage thresholds (configured in `pyproject.toml`):

| Metric | Threshold |
|--------|-----------|
| Statements | 70% |
| Branches | 60% |

Run `pytest --cov=app --cov-report=term-missing` to see the current report.

# Stock Yahoo Terminal

Real-time stock market terminal with Yahoo Finance API + Supabase authentication.

![Dark terminal UI](https://img.shields.io/badge/UI-Dark%20Terminal-00e5a0?style=flat-square)
![Supabase](https://img.shields.io/badge/Auth-Supabase-3ECF8E?style=flat-square&logo=supabase)
![Yahoo Finance](https://img.shields.io/badge/Data-Yahoo%20Finance-6001D2?style=flat-square)

## Features

- **Login** — Supabase authentication (email + password)
- **Dashboard** — Watchlist, portfolio chart, top gainers/losers, market ribbon
- **Stock Detail** — Real-time price chart (1D / 5D / 1M / 3M / 6M / 1Y / 5Y), AI summary, stats
- **Search** — `Ctrl+K` quick search across all tickers
- **Live Data** — Yahoo Finance API via CORS proxy, auto-refresh every 60s
- **Thai Stocks** — PTT.BK, CPALL.BK, AOT.BK included

## Quick Start

Open `Stock Yahoo Terminal.html` directly in a browser, or use a local server:

```bash
python -m http.server 5500
# then open http://localhost:5500/Stock%20Yahoo%20Terminal.html
```

## Demo Accounts

| Email | Password | Role |
|-------|----------|------|
| trader@demo.com | Demo1234! | Trader |
| admin@demo.com | Admin1234! | Admin |
| guest@demo.com | Guest1234! | Guest |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 (CDN + Babel) |
| Auth | Supabase JS v2 |
| Market Data | Yahoo Finance API |
| Styling | CSS (Dark terminal theme) |
| Charts | SVG (custom) |

## Configuration

Supabase credentials are set in `Stock Yahoo Terminal.html`:

```js
const CONFIG = {
  SUPABASE_URL:      'https://your-project.supabase.co',
  SUPABASE_ANON_KEY: 'your-anon-key',
};
```

## License

MIT

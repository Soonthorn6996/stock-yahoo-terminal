# ARCHITECTURE.md — Stock Yahoo

## System Summary

Stock Yahoo is a full-stack web application that allows authenticated users to look up real-time stock prices and details sourced from Yahoo Finance. The frontend is a **Next.js 14 App Router SPA**, the backend is a **FastAPI (Python)** REST API, authentication is handled by **Supabase Auth**, and data is persisted in **Supabase PostgreSQL**. AI-powered features are provided via the **Anthropic API**.

---

## High-Level Diagram

```
┌────────────────────────────────────────────────────────┐
│                    Client Layer                        │
│         Next.js 14 App Router (TypeScript)             │
│         Hosted: Vercel                                 │
└──────────────────────┬─────────────────────────────────┘
                       │ HTTPS / REST
┌──────────────────────▼─────────────────────────────────┐
│                   API Layer                            │
│       FastAPI (Python 3.12, Uvicorn)                   │
│       Auth: Supabase JWT verification                  │
│       Port: 8000 (dev) / Dockerized or cloud (prod)    │
└────────┬──────────────────────────┬────────────────────┘
         │ Supabase Client           │ httpx
┌────────▼──────────────┐  ┌────────▼───────────────────┐
│   Supabase (Cloud)    │  │   Yahoo Finance API        │
│   PostgreSQL + Auth   │  │   (yfinance / rapid API)   │
└───────────────────────┘  └────────────────────────────┘
                  │
         ┌────────▼───────────┐
         │  Anthropic API     │
         │  (AI features)     │
         └────────────────────┘
```

---

## Repository Structure

```
stock-yahoo/
├── apps/
│   ├── web/                        # Next.js 14 frontend
│   │   ├── src/
│   │   │   ├── app/                # App Router pages & layouts
│   │   │   │   ├── (auth)/         # Login / Register pages
│   │   │   │   ├── dashboard/      # Protected dashboard
│   │   │   │   └── stocks/[symbol] # Stock detail page
│   │   │   ├── components/         # Reusable UI components
│   │   │   ├── hooks/              # Custom React hooks
│   │   │   ├── lib/                # Supabase client, API client
│   │   │   └── types/              # TypeScript interfaces
│   │   ├── .env.local
│   │   └── package.json
│   └── api/                        # FastAPI backend
│       ├── app/
│       │   ├── routers/            # Route handlers (stocks, ai)
│       │   ├── services/           # Business logic (yahoo, ai)
│       │   ├── middleware/         # Auth verification
│       │   ├── models/             # Pydantic schemas
│       │   └── core/               # Config, dependencies
│       ├── .env
│       ├── requirements.txt
│       └── Dockerfile
├── .github/
│   └── workflows/
│       └── ci.yml                  # GitHub Actions CI/CD
└── README.md
```

---

## Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Frontend | Next.js 14 App Router | SSR for SEO, built-in routing, easy Vercel deploy |
| Backend | FastAPI (Python) | Fast async I/O, great for proxying Yahoo API, yfinance library support |
| Auth | Supabase Auth | Built-in JWT, social login, no custom auth server needed |
| Database | Supabase PostgreSQL | Managed cloud DB with RLS, pairs naturally with Supabase Auth |
| AI | Anthropic API | Claude for stock summarization and AI analysis features |
| Deploy | Vercel + GitHub Actions | Zero-config Next.js deploy; Actions for API CI/CD |

---

## Key Data Flows

### 1. User Authentication
```
User visits /login
  → Next.js calls Supabase Auth (client-side SDK)
  → Supabase returns JWT access token + refresh token
  → Token stored in Supabase session (httpOnly cookie)
  → User redirected to /dashboard
```

### 2. Fetch Stock Detail
```
GET /api/stocks/{symbol}  (e.g. AAPL)
  → Auth middleware: verify Supabase JWT
  → StockService.get_detail(symbol)
      → fetch from Yahoo Finance (yfinance / yahoo-fin)
  → Return stock data JSON to Next.js
  → Next.js renders stock detail page
```

### 3. AI Stock Summary
```
POST /api/ai/summarize
  → Auth middleware: verify JWT
  → Send stock data to Anthropic API (Claude)
  → Return AI-generated summary text
```

---

## External Dependencies

| Service | Purpose | Required? |
|---------|---------|-----------|
| Supabase | Auth + PostgreSQL database | Required |
| Yahoo Finance (yfinance) | Real-time stock price data | Required |
| Anthropic API | AI stock analysis & summaries | Required |
| Vercel | Frontend hosting | Production |
| GitHub Actions | CI/CD pipeline | Production |

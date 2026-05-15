# Stock Yahoo

> Web app ดูราคาหุ้น real-time จาก Yahoo Finance — ต้อง Login ก่อนเข้าใช้งาน พร้อม AI วิเคราะห์หุ้น

[![CI](https://github.com/your-org/stock-yahoo/actions/workflows/ci.yml/badge.svg)](https://github.com/your-org/stock-yahoo/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## What Is This?

Stock Yahoo ให้ผู้ใช้ที่ Login แล้วสามารถค้นหาและดูราคาหุ้นแบบ real-time โดย proxy ข้อมูลมาจาก Yahoo Finance พร้อม AI สรุปข้อมูลหุ้นผ่าน Anthropic API

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14 (App Router), TypeScript |
| Backend | FastAPI (Python 3.12), Uvicorn |
| Database | Supabase (PostgreSQL on cloud) |
| Auth | Supabase Auth (JWT) |
| Stock Data | Yahoo Finance (yfinance) |
| AI | Anthropic Claude API |
| Testing | Pytest (API), Jest / Playwright (Web) |
| Infra | Vercel (web), GitHub Actions (CI/CD) |

---

## Prerequisites

- Node.js ≥ 20.x
- Python ≥ 3.12
- pnpm ≥ 9.x (`npm install -g pnpm`)
- Supabase account + project

---

## Quick Start

```bash
# 1. Clone
git clone https://github.com/your-org/stock-yahoo.git
cd stock-yahoo

# 2. Setup frontend
cd apps/web
cp .env.local.example .env.local
pnpm install

# 3. Setup backend
cd ../api
cp .env.example .env
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 4. Start dev servers
# Terminal 1 — API
cd apps/api && uvicorn app.main:app --reload --port 8000

# Terminal 2 — Web
cd apps/web && pnpm dev
```

Web: `http://localhost:3000`  
API: `http://localhost:8000`  
API Docs: `http://localhost:8000/docs`

---

## Environment Variables

### `apps/web/.env.local`

```env
NEXT_PUBLIC_SUPABASE_URL="https://your-project.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="your-anon-key"
NEXT_PUBLIC_API_URL="http://localhost:8000"
```

### `apps/api/.env`

```env
# Supabase
SUPABASE_URL="https://your-project.supabase.co"
SUPABASE_SERVICE_ROLE_KEY="your-service-role-key"

# Anthropic
ANTHROPIC_API_KEY="sk-ant-..."
ANTHROPIC_MODEL="claude-sonnet-4-20250514"

# App
PORT=8000
ALLOWED_ORIGINS="http://localhost:3000"
```

---

## Common Commands

```bash
# Frontend
pnpm dev                   # Start Next.js dev server
pnpm build                 # Build for production
pnpm test                  # Run Jest unit tests
pnpm test:e2e              # Run Playwright E2E

# Backend
uvicorn app.main:app --reload     # Start FastAPI dev server
pytest                            # Run all tests
pytest --cov=app                  # Coverage report
```

---

## Project Structure

See [ARCHITECTURE.md](./ARCHITECTURE.md) for full folder layout and data flows.

---

## Contributing

1. Branch from `main`: `git checkout -b feat/your-feature`
2. Write tests for new logic
3. Open a PR — CI must pass before merge

---

## License

MIT © Your Org

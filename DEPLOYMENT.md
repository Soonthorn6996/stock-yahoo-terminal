# DEPLOYMENT.md — Deployment Guide for Stock Yahoo

## Infrastructure Overview

| Component | Platform | Notes |
|-----------|---------|-------|
| Frontend (`apps/web`) | Vercel | Next.js auto-deploy on merge to `main` |
| Backend (`apps/api`) | Docker + cloud host (Railway / Fly.io / GCP) | FastAPI, Dockerized |
| Database + Auth | Supabase Cloud | Managed PostgreSQL + Auth |
| CI/CD | GitHub Actions | Runs on PR and merge to `main` |

---

## Environments

| Environment | Branch | API URL | Web URL |
|-------------|--------|---------|---------|
| Development | local | `http://localhost:8000` | `http://localhost:3000` |
| Staging | `staging` | `https://api-staging.stock-yahoo.app` | `https://staging.stock-yahoo.vercel.app` |
| Production | `main` | `https://api.stock-yahoo.app` | `https://stock-yahoo.app` |

---

## Local Development

```bash
# Terminal 1 — FastAPI backend
cd apps/api
source .venv/bin/activate
uvicorn app.main:app --reload --port 8000

# Terminal 2 — Next.js frontend
cd apps/web
pnpm dev
```

API docs: `http://localhost:8000/docs`

---

## Docker (Backend)

The FastAPI backend is containerized. `Dockerfile` at `apps/api/Dockerfile`:

```bash
# Build image
docker build -t stock-yahoo-api -f apps/api/Dockerfile .

# Run container
docker run \
  -e SUPABASE_URL="https://..." \
  -e SUPABASE_SERVICE_ROLE_KEY="..." \
  -e ANTHROPIC_API_KEY="sk-ant-..." \
  -e ALLOWED_ORIGINS="https://stock-yahoo.app" \
  -p 8000:8000 \
  stock-yahoo-api
```

---

## CI/CD Pipeline (GitHub Actions)

Pipeline: `.github/workflows/ci.yml`

### On every pull request

```
1. Install Python deps + run pytest (backend)
2. Install Node deps + run tsc --noEmit (frontend)
3. Run pnpm lint (frontend)
4. Run Jest unit tests (frontend)
5. Build Next.js (frontend)
```

### On merge to `main`

```
1. All PR checks above
2. Build Docker image → push to GitHub Container Registry (GHCR)
3. Deploy API image to production host
4. Vercel auto-deploys frontend from main branch
```

### On merge to `staging`

Same as `main` but deploys to staging environment only.

---

## Vercel Configuration (Frontend)

`vercel.json` at `apps/web/`:

```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }],
  "env": {
    "NEXT_PUBLIC_API_URL": "https://api.stock-yahoo.app"
  }
}
```

**Required Vercel environment variables** (set in Vercel dashboard):

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
NEXT_PUBLIC_API_URL
```

---

## Backend Deployment (Railway / Fly.io)

**Required environment variables** on the API host:

```
SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
ANTHROPIC_API_KEY
ANTHROPIC_MODEL              # claude-sonnet-4-20250514
ALLOWED_ORIGINS              # https://stock-yahoo.app
PORT                         # 8000
```

Health check endpoint: `GET /api/health` — returns `200 { "status": "ok" }`.

---

## Supabase Migrations in CI

Apply schema migrations as part of deployment:

```bash
# Requires Supabase CLI + SUPABASE_ACCESS_TOKEN in CI env
supabase db push --project-ref your-project-ref
```

Always test migration against staging Supabase project before applying to production.

---

## Rolling Back

### Frontend (Vercel)
Vercel Dashboard → Deployments → select previous deployment → **Promote to Production**

### Backend
Re-deploy the previous Docker image tag from GHCR via the hosting dashboard or CLI.

### Database
Supabase does not auto-generate down migrations. To roll back:
1. Write a manual reversal SQL file in `apps/api/supabase/migrations/`
2. Apply via `supabase db push`

Always test rollback steps in staging first.

---

## Monitoring

| Tool | Purpose | Access |
|------|---------|--------|
| Vercel Analytics | Core Web Vitals, traffic | Vercel dashboard |
| Supabase Dashboard | DB queries, auth logs, API usage | supabase.com |
| FastAPI `/docs` | Live API introspection | `/docs` on API host |
| GitHub Actions | CI build status | GitHub repository |

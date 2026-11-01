# Barcode Intelligence (India)

Search a barcode or product name and get the product identity, ingredients and nutrition (food), GST 2.0 and HSN guidance, images with downloads, Indian shop listings and a full source list. Every field shows where it came from. Missing data shows "Information not available". Unclear GST/HSN shows "Requires verification". Nothing is invented and there is no stored product dataset.

```
backend/      FastAPI API, providers, GST table, tests
frontend/     React + Vite + TypeScript + Tailwind
standalone/   index.html: the same idea as one file, no server needed
Dockerfile    builds frontend and serves it with the API (one URL)
render.yaml   one-click deploy on Render
```

## 1. Run on your computer
```bash
cp .env.example .env          # fill in keys, see below
docker compose up --build     # open http://localhost:8000
```
Without Docker: `cd backend && pip install -r requirements.txt && uvicorn app.main:app --reload` and, in another terminal, `cd frontend && npm install && npm run dev` (open http://localhost:5173). Tests: `cd backend && pytest`.

## 2. Put it on GitHub
Create an empty repo on github.com (no README), then:
```bash
git init && git add . && git commit -m "Barcode Intelligence"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git push -u origin main
```
`.env` is git-ignored, so your keys are not uploaded.

## 3. Get a share link
**A. Full app with backend (recommended), on Render.** Render dashboard > New > Blueprint > choose your repo. It reads `render.yaml`, builds the Dockerfile and creates a Postgres database. Open the service > Environment and add `GOOGLE_API_KEY` and `GOOGLE_CSE_ID` (or `SERPAPI_KEY`). Your link is `https://<service-name>.onrender.com`. Free instances sleep when idle, so the first visit can take about a minute.

**B. Standalone page on GitHub Pages (no server).** Repo Settings > Pages > Source: GitHub Actions, then push to `main`. The page is published at `https://YOUR-USERNAME.github.io/YOUR-REPO/`. Visitors paste their own Google or Anthropic keys in the page (stored in their browser only), because a static page cannot hold a secret.

Shareable result links look like `<your link>/?q=BARCODE`.

## 4. Keys
| Key | Where to get it | Used for |
|---|---|---|
| `GOOGLE_API_KEY` | console.cloud.google.com, enable "Custom Search API" | live Google India search |
| `GOOGLE_CSE_ID` | programmablesearchengine.google.com, search the entire web, turn on Image search | which sites to search |
| `SERPAPI_KEY` (optional) | serpapi.com | alternative to Google |
| `UPCITEMDB_KEY` (optional) | upcitemdb.com | higher UPC lookup limit |

Open Food Facts and UPCitemdb (trial) need no key. Without a Google or SerpAPI key, web search, retailer listings and web images are reported as "not configured". Google's free tier is about 100 queries a day and one barcode search uses about 7, so set `RATE_LIMIT_PER_MIN` and `CACHE_TTL_HOURS`.

## API
Interactive docs at `/docs`. `POST /api/search/barcode` | `GET /api/products/{id}` | `/api/search/history` | `/api/products/{id}/images` | `/images/{n}/download` | `/images/zip` | `/export?format=json|csv` | `/api/health`.

## GST and HSN
CBIC and the GST Portal have no public rate API. `backend/app/gst.py` holds a small curated table for GST 2.0 (effective 22 Sep 2025) and labels matches "indicative". Anything else is "requires verification". Extend the table from the official notifications for your catalogue and verify before invoicing.

## Not included yet
Alembic migrations (tables are auto-created), server-side PDF export (use Print > Save as PDF), manufacturer-site scraping for specifications, GS1 verification (needs a paid account), accounts. The in-memory rate limiter is per process; use Redis if you run several workers. Check image usage rights before reuse.

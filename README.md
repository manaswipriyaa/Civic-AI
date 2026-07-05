# CivicAI — Municipal Decision Intelligence Platform

A working full-stack starter: React + Tailwind frontend, FastAPI backend,
SQLite persistence, JWT auth, and real Gemini/Google Maps wiring that
**automatically falls back to mock data** if no API keys are set — so the
whole app runs end-to-end out of the box.

## What's included

- **AI Chat Assistant** — natural-language Q&A (traffic, hospitals, schemes)
- **Document Q&A (RAG)** — upload a text file, ask questions grounded in it, with an upload UI
- **Service Finder** — hospitals, police, fire, schools, bus stations — real Google Places search if `GOOGLE_MAPS_API_KEY` is set, mock data otherwise
- **Traffic Dashboard** — congestion by road (mock — see note below)
- **Complaint Analyzer** — free-text complaint → department + priority, persisted per user
- **City Data Dashboard** — complaint trends, AQI, water usage charts
- **Auth** — signup/login with JWT, passwords hashed with bcrypt
- **Persistent database** — SQLite via SQLAlchemy; users, complaints, and documents survive restarts

## Run the backend

```bash
cd backend
python -m venv venv && source venv/bin/activate   # optional but recommended
pip install -r requirements.txt
cp .env.example .env
uvicorn main:app --reload --port 8000
```

Backend runs at `http://localhost:8000`. Check `/api/health` — it reports
whether Gemini and Maps are running in mock mode. A `civicai.db` SQLite
file is created automatically on first run.

## Run the frontend

```bash
cd frontend
npm install
cp .env.example .env
npm run dev
```

Frontend runs at `http://localhost:5173`. Sign up for an account to report
issues and see your own complaint history.

## Adding real API keys

**Gemini** (chat, document Q&A, complaint classification):
1. Get a key at https://aistudio.google.com/apikey
2. Add it to `backend/.env` as `GEMINI_API_KEY`

**Google Maps Places** (real nearby-service search):
1. Enable "Places API (New)" at https://console.cloud.google.com/google/maps-apis
2. Add the key to `backend/.env` as `GOOGLE_MAPS_API_KEY`
3. Call `/api/services?lat=...&lng=...` with real coordinates (the frontend
   currently calls it without coordinates, so it'll still return mock data
   until you wire up browser geolocation — see "Next steps" below)

Restart the backend after adding keys — `/api/health` should show
`mock_mode: false` for whichever service you configured.

**JWT_SECRET**: change this in `backend/.env` before deploying anywhere
real — the default is only safe for local development.

## A note on "real" traffic data

The Traffic Dashboard is mock-only, on purpose. Live, traffic-aware data
(current congestion, ETAs) requires Google's billed Routes API or a
municipal traffic feed — a materially bigger integration than Places
search, usually with its own cost and rate-limit considerations. Rather
than fake it, `/api/traffic` stays clearly mock and `maps_client.py`
documents exactly what a real integration would need.

## Project structure

```
backend/
  main.py            FastAPI app, all API routes
  auth.py             Password hashing + JWT issuing/verification
  db.py               SQLAlchemy models (User, Complaint, Document) + session
  gemini_client.py    Gemini wrapper (chat, RAG answers, complaint classification)
  maps_client.py      Google Places wrapper for the Service Finder
  rag_store.py        In-memory document chunking + naive keyword retrieval
  mock_data.py        Mock services / traffic / analytics data
frontend/
  src/pages/          One file per feature page, including Login and Documents
  src/components/     Shared nav shell + ticker
  src/lib/api.js      Fetch wrapper — attaches JWT automatically where needed
  src/lib/AuthContext.jsx  Login state, persisted in localStorage
```

## Known gaps / next steps

- Service Finder frontend doesn't request browser geolocation yet — add a
  `navigator.geolocation.getCurrentPosition()` call in `Services.jsx` and
  pass the coords into `api.services(type, coords)` (the backend already
  accepts `lat`/`lng`)
- RAG uses keyword-overlap retrieval, not real embeddings — fine for a
  demo, swap in Vertex AI Matching Engine or pgvector for real semantic search
  over larger document sets
- No password reset / email verification flow
- SQLite is fine for a demo or small deployment; swap `DATABASE_URL` for
  Postgres for anything with concurrent writers
- Traffic is mock-only — see note above

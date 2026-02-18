# Expense Tracker

Minimal full-stack expense tracker built for the assignment requirements.

**Live Demo:** https://esepns.netlify.app

---

## Stack

* **SQLite** (file-based persistence)
* **Express + Node.js** (REST API)
* **React + Vite** (frontend)

---

## Project Structure

```
backend/
├── src/
│   ├── db/
│   │   ├── index.js
│   │   └── schema.sql
│   ├── services/expenseService.js
│   ├── routes/expenses.js
│   ├── middleware/validate.js
│   └── app.js
└── tests/expenses.test.js

frontend/
├── src/
│   ├── components/
│   │   ├── ExpenseForm.jsx
│   │   ├── ExpenseTable.jsx
│   │   ├── FilterBar.jsx
│   │   ├── MonthOverview.jsx          # Dashboard Level 1
│   │   ├── MonthCategoryView.jsx      # Dashboard Level 2
│   │   └── MonthCategoryExpenses.jsx  # Dashboard Level 3
│   ├── api/client.js
│   ├── utils/format.js
│   └── App.jsx
└── styles.css
```

---

## Why This Persistence Choice

SQLite is zero-config and file-based, so data survives server restarts. Schema constraints and unique indexes handle validation and idempotency at the database level.

---

## Features Implemented

### Core (Assignment Requirements)
* `POST /expenses` to create expenses with:
  * `amount`, `category`, `description`, `date`
* Retry-safe create using idempotency keys:
  * Accepts `X-Idempotency-Key` header
  * Duplicate key returns existing expense instead of creating a duplicate
* `GET /expenses` with:
  * `?category=Food` filter (exact match, case-insensitive)
  * `?sort=date_desc` sorting (newest first, default)
* Frontend UI:
  * Add expense form
  * Expense table with filters
  * Total amount (server-computed)
  * Loading and error states

### Extra Feature: 3-Level Dashboard Drill-Down
* **Level 1:** Month overview with bar chart + month cards
* **Level 2:** Category breakdown for selected month (donut chart + tiles)
* **Level 3:** Individual expenses for month+category (detailed table)
* Navigation: Click chart/cards to drill down; back buttons preserve context

---

## Money Handling

Amounts stored as **integer cents** (`₹150.75` → `15075`) to avoid floating-point errors. API responses return decimal strings (`"150.75"`).

---

## Idempotency Behavior

* Frontend generates UUID (`crypto.randomUUID()`) on form mount, not on click
* Backend inserts with `UNIQUE` constraint on `idempotency_key`
* Duplicate key → catches error, returns original expense with `200 OK`
* Client sees same response whether 1st or 10th retry

---

## Run Locally

### 1. Start Backend
```bash
cd backend
npm install express better-sqlite3 cors uuid
npm install -D vitest supertest
npm run dev
```
Backend runs at `http://localhost:3000`

### 2. Start Frontend
```bash
cd frontend
npm install recharts
npm run dev
```
Open `http://localhost:5173`

### Environment Variables
**Backend** (`backend/.env`):
```
PORT=3000
DB_PATH=./data/expenses.db
CORS_ORIGIN=http://localhost:5173
```

**Frontend** (`frontend/.env`):
```
VITE_API_URL=http://localhost:3000
```

---

## API Quick Examples

**Create expense:**
```bash
curl -X POST http://localhost:3000/expenses \
  -H "Content-Type: application/json" \
  -H "X-Idempotency-Key: $(uuidgen)" \
  -d '{"amount":"150.75","category":"Food","description":"Groceries","date":"2025-01-15"}'
```

**List expenses (filtered + sorted):**
```bash
curl "http://localhost:3000/expenses?category=Food&sort=date_desc"
```

**Dashboard month overview:**
```bash
curl http://localhost:3000/expenses/dashboard/months
```

---

## Deployment

### Backend: Render.com (Free Tier)
* **Root Directory:** `backend`
* **Start Command:** `npm start`
* **Env:** `DB_PATH=./data/expenses.db`, `CORS_ORIGIN=<frontend-url>`

### Frontend: Netlify or Vercel
* **Base Directory:** `frontend`
* **Build Command:** `npm run build`
* **Publish Directory:** `dist`
* **Env:** `VITE_API_URL=<backend-url>`

**Deploy order:** Backend first (get URL) → frontend (use that URL) → update backend CORS

---

## Trade-offs Due to Timebox

* Focused on clean assignment scope (no auth/multi-user)
* No pagination for expense lists
* Fixed category dropdown (no custom categories)

---

## Intentionally Not Done

* CSV export/import
* Budget goals or alerts
* Optimistic UI updates
* Frontend unit tests (backend tests exist)

---

## Tests

Run backend tests:
```bash
cd backend
npm test
```

Covers: idempotency, validation, filtering, sorting, dashboard endpoints.
<div align="center">

# 🧪 QueryLab

**Ask your database a question. Get a real answer.**

QueryLab turns plain-language questions into SQL, runs them against your database, and explains the result back in human language — so anyone can explore data without knowing a single `JOIN`.

[![Next.js](https://img.shields.io/badge/Next.js-15-000000?logo=next.js&logoColor=white)](#)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)](#)
[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)](#)
[![Node.js](https://img.shields.io/badge/Node.js-339933?logo=node.js&logoColor=white)](#)
[![Prisma](https://img.shields.io/badge/Prisma-2D3748?logo=prisma&logoColor=white)](#)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)](#)

</div>

---

## 💡 The idea

For most people, a relational database is a locked room. The data is all there, but the key is SQL — and learning it just to answer *"how many students enrolled in Computer Science last year?"* is a dealbreaker.

QueryLab hands the key to an LLM. You type the question in Portuguese or English, it writes the SQL, executes it safely, shows the results in a table or chart, and gives you a plain-language summary of what the answer means.

It was built around one hard reality: **LLMs hallucinate table and column names.** So QueryLab first *discovers your real schema*, embeds it, and only feeds the model the parts that matter — which keeps the generated SQL grounded in tables that actually exist.

## ✨ What it does

- 🗣️ **Natural language → SQL** — ask in Portuguese or English, get runnable SQL back.
- 🧠 **Multi-provider LLM routing** — swap between **Groq**, **Google Gemini**, **OpenRouter** and **Cloudflare Workers AI (SQLCoder)**; each provider sits behind one service interface.
- 🗺️ **Schema discovery + embeddings** — the app introspects the database, indexes the schema, and a *smart schema reducer* trims it down so even large schemas fit the model's context window.
- 🧯 **Anti-hallucination safeguards** — generated SQL is validated and common hallucinations are auto-corrected before anything runs.
- ↔️ **Reverse translation** — after executing, the app explains the result (and the SQL) back in natural language.
- 📊 **Results that make sense** — formatted tables, charts (**Recharts**), diagrams (**Mermaid** / **React Flow**) and data export.
- 💬 **Chat sessions over WebSocket** — every conversation is saved, with a full history sidebar you can revisit.
- 🧪 **Built-in evaluation suite** — a dedicated area and API to score generated SQL against expected results, so changes are measured, not guessed.
- 📚 **Schema dictionary** — a browsable reference of your tables and columns.
- 🔐 **Accounts & auth** — register, log in, and keep your sessions private (JWT + bcrypt).
- 🎨 **Three themes** — light, dark and high-contrast, for comfort and accessibility.
- 🗄️ **Pluggable databases** — adapters for **PostgreSQL**, **IBM DB2** (via a dedicated microservice) and a mock **JSON Server** for development.

## 🏗️ How it's put together

```
┌────────────────────────┐      WebSocket + REST      ┌───────────────────────────┐
│  Frontend — Next.js 15 │  ◀──────────────────────▶  │  Backend — Express + TS   │
│  React 19 · Tailwind   │                            │  JWT · Prisma · Socket.IO │
└────────────────────────┘                            └────────────┬──────────────┘
                                                                   │
                              ┌────────────────────────────────────┼─────────────────────────────┐
                              ▼                                    ▼                             ▼
                     LLM providers                    App DB (Postgres/Prisma)      Query DB adapters
              Groq · Gemini · OpenRouter                  users, sessions          PostgreSQL · DB2 · JSON
              Cloudflare AI (SQLCoder)                                                   │
                                                                                           ▼
                                                                                  your real data
```

The npm workspace keeps things tidy: `backend/` and `frontend/` are siblings, and the root scripts run both at once.

## 🧱 Tech stack

| Layer | Choice |
| --- | --- |
| Frontend | Next.js 15 (App Router) · React 19 · TypeScript · Tailwind CSS · Radix/shadcn · Zustand · Zod |
| Charts & diagrams | Recharts · Mermaid · React Flow (`@xyflow/react`) |
| Backend | Node.js · Express 5 · TypeScript · Socket.IO · Helmet · Morgan |
| Database | PostgreSQL via **Prisma** (app data) + pluggable query adapters |
| LLMs | Groq · Google Gemini · OpenRouter · Cloudflare Workers AI · Replicate |
| Auth | JWT + bcrypt |
| Deploy | Docker Compose · Render (`render.yaml`) |

## 🚀 Getting started

### Option A — Docker (everything at once)

```bash
docker compose up
```

This starts PostgreSQL, the backend API and the Next.js frontend. Then open **http://localhost:3000**.

### Option B — Local development

```bash
# 1. Install every workspace
npm install

# 2. Configure the backend  (copy backend/.env.example to backend/.env)
#    POSTGRES_URL=postgresql://user:pass@localhost:5432/querylab     # app database
#    JWT_SECRET=...  JWT_EXPIRES_IN=7d
#    GROQ_API_KEY=...  GEMINI_API_KEY=...  OPENROUTER_API_KEY=...    # pick your LLM provider(s)
#    QUERY_DB_TYPE=postgres   QUERY_DB_HOST/ PORT/ USER/ PASSWORD    # the database you'll query

# 3. Configure the frontend  (frontend/.env.local)
#    NEXT_PUBLIC_API_URL=http://localhost:5000/api
#    NEXT_PUBLIC_WEBSOCKET_URL=http://localhost:5000

# 4. Set up the database
npx prisma migrate deploy -w backend
npm run prisma:seed -w backend

# 5. Run backend + frontend together
npm run dev
```

Then open **http://localhost:3000**, create a session, and try:

> *"List all federal universities in the South region."*
> *"How many professors does USP have?"*
> *"Which Computer Science courses have the highest entry scores?"*

## 📁 Project structure

```
querylab/
├── frontend/            # Next.js app (chat, results, schema dictionary, evaluations)
├── backend/             # Express + TypeScript API
│   ├── src/routes/      # auth · chat · sessions · history · schema-discovery · evaluation · health
│   ├── src/services/    # LLM routing, SQL generation, schema reduction, execution, evaluation
│   ├── src/adapters/    # database adapters: postgres · db2-http · json-server
│   └── prisma/          # schema, migrations and seed
├── scripts/             # schema discovery helpers
├── docker-compose.yml   # postgres + backend + frontend
└── render.yaml          # one-click deploy blueprint
```

## 🔌 API highlights

| Area | Endpoints |
| --- | --- |
| Health | `GET /api/health` · `GET /api/health/database` |
| Chat | `POST /api/chat/message` · `GET /api/chat/sessions/:id` |
| Sessions | `GET/POST /api/sessions` · `PUT/DELETE /api/sessions/:id` |
| Schema | `/api/schema-discovery` |
| Evaluation | `/api/evaluation` · `/api/test-results` |
| History | `/api/history` |
| Auth | `/api/auth` |

## 👤 Author

**Diovani Facco** — [LinkedIn](https://www.linkedin.com/in/diovani-facco-3a247b11a/) · [diovanifacco@gmail.com](mailto:diovanifacco@gmail.com)

## 📄 License

MIT.

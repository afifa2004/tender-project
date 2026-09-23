# Sentinel Group Intelligence — Full Project

Two folders:
- `backend/` — Python API + AI (RAG). No SQL, no database server, no Docker.
  Data is stored in a plain JSON file. Search runs on a lightweight
  built-in index — no external database, no compiled dependencies that can
  fail to install.
- `frontend/` — the existing React dashboard, now wired to call the backend
  for the AI Assistant.

---

## The ONLY thing you have to fill in

**Nothing, actually — free by default.** The AI Assistant uses [Ollama](https://ollama.com)
by default: a free program that runs an AI model on your own computer.
No API key, no credit card, no bills, ever.

### One-time Ollama setup (~5 minutes)

1. Install Ollama: https://ollama.com/download (Windows/Mac/Linux, free).
2. Open a terminal and run:
   ```
   ollama pull llama3.2
   ```
   (downloads a ~2GB model, one-time only)
3. Leave Ollama running in the background (it starts automatically on
   most installs — check for its icon in your system tray, or run
   `ollama serve` if it's not already running).

That's it. `backend/.env` is already set to use it by default
(`LLM_PROVIDER=ollama`) — nothing to edit.

### Don't want to install anything locally? Use Groq instead (also free)

Open `backend/.env` and change:
```
LLM_PROVIDER=groq
GROQ_API_KEY=paste-a-free-key-from-console.groq.com/keys
```
Groq's free tier needs no card and is generous for personal/dev use.

### Want Claude specifically, and have credits?

```
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...
```

---

## How to run it (every time)

You need **two terminals open at the same time** — one for backend, one for frontend.

### Terminal 1 — Backend

**Mac/Linux:**
```bash
cd backend
./run.sh
```

**Windows:** double-click `backend/run.bat`, or in a terminal:
```
cd backend
run.bat
```

First run installs everything automatically (takes a couple of minutes — it's
downloading a small local AI model for search, ~90MB, one-time only).
You'll see:
```
Starting Sentinel backend on http://localhost:8000
```
Leave this terminal running.

### Terminal 2 — Frontend

```bash
cd frontend
npm install
npm run dev
```

Opens automatically at **http://localhost:3000**

---

## What you'll see

- The dashboard now loads **live from your backend** — companies, tenders,
  credentials all come from `GET /api/companies` and `GET /api/tenders`,
  not the old static mock file. If the backend isn't running, you'll see a
  clear "could not reach the backend" screen with a Retry button instead of
  a blank or broken page.
- **Adding a tender** (the "+ Add Tender" button) now really saves it to the
  backend (`backend/data/db.json`) — refresh the page and it's still there.
- **Updating a tender's status** on its detail page also saves to the backend.
- Click the **AI Assistant** (sparkle icon) and ask something like:
  - "Which company is best for a manned guarding tender?"
  - "What certifications does Sentinel Technologies hold?"
- These go to your real backend, get retrieved from the indexed company/
  tender data (RAG), and get answered by your chosen AI provider (Ollama by
  default).

First time you open the Assistant, the backend automatically:
1. Loads all company/tender data into `backend/data/db.json`.
2. Converts it into searchable chunks in `backend/data/chroma/`.

Both happen once, silently, on backend startup — nothing to run manually.

---

## Uploading tender documents (optional, for real RAG over your own files)

```bash
curl -X POST http://localhost:8000/api/documents/upload \
  -F "file=@/path/to/your-tender.pdf"
```

That file's content gets chunked and indexed immediately — ask the Assistant
about it right after.

---

## If something goes wrong

- **"Could not reach the backend"** in the Assistant → Terminal 1 isn't
  running, or crashed. Check it for errors.
- **Backend won't start / `python3` not found** → install Python 3.10+ from
  python.org first.
- **Frontend won't start / `npm` not found** → install Node.js 18+ from
  nodejs.org first.
- **Assistant replies with a ⚠️ "Can't reach Ollama"** → Ollama isn't running.
  Open the Ollama app, or run `ollama serve` in a terminal, and try again.
- **Assistant replies with a ⚠️ "Model isn't pulled yet"** → run
  `ollama pull llama3.2` and try again.

---

## What's actually happening under the hood (for reference, not required reading)

- `backend/storage.py` — a JSON file acting as your database (TinyDB). No SQL.
- `backend/vectorstore.py` — a lightweight built-in search index (no
  external database, no native code, works on any Python 3.10+).
- `backend/indexing.py` — turns company/tender/document data into searchable
  chunks.
- `backend/rag.py` — sends your question + the most relevant chunks to Claude,
  and Claude answers using only that retrieved context (won't invent facts).
- `backend/main.py` — the API itself (FastAPI), single file, all routes.

# JesusGPT — Deployment Guide

## Local Development

### Prerequisites
- Node.js 18+
- Groq API key — [console.groq.com](https://console.groq.com) (free)
- Gemini API key — [aistudio.google.com](https://aistudio.google.com) (free tier, fallback only)

### Setup

```bash
git clone https://github.com/HermannPR/lab6.git
cd lab6/jesusgpt

# Install root deps
npm install

# Install server deps
cd server && npm install && cd ..

# Install client deps
cd client && npm install && cd ..
```

### Environment variables

Create `server/.env`:
```
GROQ_API_KEY=gsk_...
GEMINI_API_KEY=AIza...
PORT=3001
```

> **Never commit `.env`.** It is in `.gitignore`.

### Run

```bash
npm run dev
```

- Server: `http://localhost:3001`
- Client: `http://localhost:3000`

---

## Deploy to Vercel

### 1. Push to GitHub

```bash
git push origin main
```

### 2. Import on Vercel

1. Go to [vercel.com](https://vercel.com) → Add New Project
2. Import `HermannPR/lab6` (or your fork)
3. Set **Root Directory** to `jesusgpt`
4. Framework: **Vite**

### 3. Environment Variables

In Vercel → Project → Settings → Environment Variables, add:

| Key | Value |
|-----|-------|
| `GROQ_API_KEY` | `gsk_...` |
| `GEMINI_API_KEY` | `AIza...` |

### 4. Redeploy

Vercel auto-deploys on push. After adding env vars, trigger a redeploy:
Deployments tab → three dots → Redeploy.

---

## Architecture

```
client/          React + Vite + Tailwind
  src/
    App.jsx              chat thread, conversation state
    components/
      QuestionInput.jsx  auto-resize textarea, mode pills, send btn
      ResponseDisplay.jsx  response + Bible Gateway verse links
      LanguageSwitcher.jsx  EN / ES / LA / GR
    hooks/
      useReflection.js   fetch + conversation history
    context/
      LanguageContext.jsx

server/          Express (Node ESM)
  server.js              entry, CORS, gospel data middleware
  routes/
    reflect.js           POST /api/reflect — validates, runs RAG, calls AI
  services/
    geminiClient.js      Groq primary, Gemini fallback, system prompts
    ragEngine.js         keyword search over gospels.json
    bibleLoader.js       loads gospels.json once at startup

api/             Vercel serverless entry (wraps server/)
vercel.json      routes /api/* → serverless, /* → client/dist
```

## Models

| Mode | Model | Provider |
|------|-------|----------|
| Direct (fallback) | `llama-3.3-70b-versatile` | Groq |
| Parable (fallback) | `llama-3.3-70b-versatile` | Groq |
| Fallback | `gemini-2.0-flash` | Google |

## Known limits

- Free Groq tier: 30 req/min, 500k tokens/day — plenty for personal use
- Gemini free tier: low RPD — treat as emergency fallback only
- RAG is keyword-based (not semantic embeddings) — planned upgrade

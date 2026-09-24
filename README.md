### Hi, I'm Finn 👋

I build full-stack products and automation tools — from an AI-driven interview prep platform to a decision-support system for equity trading. Below are write-ups of my active projects. Source for both is private; reach out if you'd like a walkthrough.

---

## 🎙️ AI Interview Prep

A locally-running platform that puts you through a live, voice-based mock technical interview with an AI interviewer, then gives you a structured breakdown of how you did.

**How it works:** pick a company (Google, Amazon, Jane Street at MVP), a role, and an interview round. A speech-to-text pipeline feeds your spoken answers to an orchestrator model that plays the interviewer in real time — asking follow-ups, giving hints in a company-specific style, and reacting to wrong answers the way that company's interviewers actually do. Coding questions get a live pseudocode editor synced to the same orchestrator. When the interview ends, a separate model scores four categories (out of 10.0, decimal precision) and produces a per-question breakdown with full audio replay and a flag timeline marking exactly where things went well or sideways.

**Architecture**

```
Next.js frontend  ⇄  WebSocket  ⇄  FastAPI backend
                                      ├─ Deepgram Nova-2      (speech-to-text, ~300ms)
                                      ├─ ElevenLabs Turbo v2   (text-to-speech)
                                      ├─ Claude Haiku          (interviewer / orchestrator)
                                      ├─ Claude Sonnet         (briefings + scoring)
                                      └─ SQLite                (sessions, question bank)
```

**Two-person build, split along the data boundary:**
- **My half:** the scraper/data pipeline — collecting real interview questions from public sources (GitHub prep lists, forums, blogs), deduplicating them (exact-hash + embedding similarity), and storing them against a shared schema.
- **My partner's half:** the AI enrichment, live interview orchestration, and scoring models that read from that store.

**Engineering decisions worth calling out** — every technology choice here was made deliberately and is documented: Deepgram over Whisper because batch transcription adds 1–3s of dead air after every answer and kills the conversational feel; ElevenLabs over two other TTS options after blind-listening to samples, because voice quality is part of the actual product experience; a randomized (not fixed) silence threshold before the AI prompts you, because a fixed timer is gameable and real interviewers don't behave like a timer.

**Stack:** Next.js 16 · TypeScript · Tailwind v4 · Clerk auth · FastAPI · Deepgram · ElevenLabs · Claude (Haiku + Sonnet) · SQLite

---

## 📈 Stock Model

A personal decision-support tool that automates the daily chart-watching a swing-trading strategy normally requires — so signals surface on their own instead of depending on manually checking charts every day.

**The core idea:** every signal requires two independent systems to agree before it fires.

```
   Technical pillar                 Fundamental / news pillar
   (is this the right               (does this company deserve
    moment, chart-wise?)              the trade at all?)
          │                                    │
          └──────────────►  final signal  ◄────┘
```

The technical pillar reads price action — momentum, volatility bands, moving averages, volume confirmation — to judge *timing*. The fundamental pillar runs each company's recent news, earnings, and financial health through Claude to judge *quality* — and can veto a technically perfect setup if the underlying business looks shaky. That veto tier (a chart that looks like a buy, but isn't one) is the main reason this exists as two systems instead of one: it catches the trades that look right and aren't.

Signals resolve to one of six tiers (Strong Buy → Strong Sell), each with position-sizing suggestions scaled to volatility rather than a flat dollar amount, so riskier names automatically get smaller allocations.

**Delivered through:**
- A Streamlit dashboard — portfolio health, live signals, per-ticker deep dives, and an AI-generated running strategy document that updates daily
- A scheduled daily job (GitHub Actions) that regenerates everything and sends a digest, so the whole system runs unattended

**Stack:** Python · yfinance · Claude API · Streamlit · GitHub Actions

---

*Both projects' full technical documentation, architecture decisions, and source live in private repos.*

### Hi, I'm Finn 👋

CS + Math at MIT. Below is my **personal and research work** — side projects and academic research built on my own time. It does not include the bulk of my professional engineering work at **PowerPay**, which is proprietary and not represented in public repos. A brief summary of that work is below for context; everything after it is what you'll actually find in my repos.

---

## 💼 Professional work (PowerPay) — not public

At PowerPay I own a production payment platform end to end: a **borrower portal**, an **agent portal**, scheduled payment/autopay services, and a **Payment Bridge** integrating multiple payment providers and systems of record — running across React, TypeScript, Node.js, Python (AWS Lambda), SQL Server, Snowflake, and internal APIs on infrastructure supporting roughly **150,000 active loans and ~$6B in annual loan volume**.

Two pieces of that work, called out specifically because they're the ones most often asked about:
- **ML-driven underwriting pipeline** — building an underwriting system on real financial data to inform lending decisions.
- **Borrower portal (full-stack)** — the user-facing application borrowers use to manage and pay down their loans, one piece of the larger platform described above.

Also part of that role: leading a SQL Server → Snowflake migration, where I built an AI-assisted workflow to migrate and validate ~200 reports in a few days — using AI to accelerate translation and debugging, then diffing old vs. new output one-to-one to isolate real discrepancies in joins, transformations, and business logic.

None of this is open source — it's commercial infrastructure handling real loan volume, not something I can publish. Happy to walk through it in more depth directly.

---

## 🔬 Research: Sports Simulation Lab — NFL Feedback Model

Contributor to an MIT undergraduate research lab studying **why sports leagues produce dominance vs. parity** — how structural feedback (a team's success today changing its resources tomorrow) versus pure randomness shapes long-run league outcomes. The lab's approach: build minimal, interpretable league simulators, calibrate them against real historical data via grid search and simulation-based inference, and read the fitted parameters back as claims about the underlying mechanisms.

The core model gives every team a resource state **κ** that maps to in-season skill, then evolves season-to-season through decay, a baseline equal share, and a share proportional to wins — letting the lab dial "success breeds success" up or down and see what long-run dynamics fall out.

**My contribution:** generalized this feedback model from its original NBA implementation to the **NFL**, then spent most of the work making the long-run simulation's accuracy claims actually honest. That included:

- Adding a κ ceiling/floor cap mechanism and an evolving state-resetting range, plus team-level validation against real historical standings
- Fixing per-season game counts to reflect the NFL's actual schedule history (the 1982 and 1987 strike-shortened seasons were being simulated as full 16-game seasons instead of 9 and 15) rather than only accounting for the 2021 move to 17 games
- Decoupling the per-season random draws (a single RNG threaded across all 44 simulated seasons meant changing one season's mechanics desynced every later season's results)
- Catching a data-leakage bug where a season's simulated state was quietly being anchored on that same season's real result before it should have been "observed" in simulation time — combined with a second calibration bug, this had inflated the model's apparent long-run accuracy (RMSE ≈ 1.82); the honest number after both fixes is ≈ 4.42 (best regrid: 3.72), which is the real baseline the lab is now working to improve

That last item is the one I'd point to first — it's a case of a model looking better than it actually was for structural reasons, catching it, and re-baselining the work honestly instead of reporting the flattering number.

---

## 🩺 Kinetic Health Services

Website for an orthopedic/rehab products supplier — patient-facing storefront plus an internal admin tool.

- **`khs_user_frontend`** — patient-facing Next.js app: browse orthopedic/rehab products by family (e.g., a knee brace across sizes and sides), see doctor recommendations
- **`khs_admin_frontend`** — internal Next.js tool for staff to manage doctors, products, and doctor → product recommendations
- **`khs_backend`** — standalone Express/TypeScript API, both frontends talk to it server-side only (the backend URL is never exposed to the browser)
- **Data layer** — Postgres schema modeling product *families* vs. individually purchasable *variants* (size/side combinations), S3-compatible object storage (MinIO locally) for product photos

**Stack:** Next.js · TypeScript · Express · PostgreSQL · Docker Compose (local Postgres + MinIO sandbox) · designed for AWS deployment (ECS/Fargate or Elastic Beanstalk + Amplify/S3+CloudFront)

---

## 🧱 AI Block Coding

A visual, block-based coding tool for an academic platform, with a natural-language "AI generates the program" layer in progress.

**The interesting design decision:** the tool is built on Blockly (Google's visual programming library) for the drag-and-drop editor, but rather than using Blockly's own workspace JSON as the program's source of truth, I built a translator layer (`blockly-interface`) that converts between Blockly's workspace and a small, independently-defined block vocabulary. That keeps the compiler, the (in-progress) AI generation service, and the code formatter fully decoupled from Blockly's serialization format — swapping the editor library later wouldn't require touching any of the other pieces.

- **`blockly-interface`** — the translator layer (block defs, toolbox, `blocklyToProgram`/`programToBlockly`)
- **`block-compiler`** — Python: parses/validates the block-tree JSON and emits real Python source
- **`pyodide-worker`** — runs that compiler client-side via Pyodide, executing generated code in a web worker so nothing round-trips to a server
- **`ai-block-creator`** — FastAPI service for the natural-language → block-tree generation feature (in progress)
- **`block-publisher`** — formats generated Python output with `black`

**Stack:** TypeScript (Vite) · Blockly · Python · Pyodide (WASM) · FastAPI

---

## 📈 Stock Model

A personal decision-support system that automates the daily chart-watching a swing-trading strategy would otherwise require, so signals surface on their own instead of depending on checking charts manually.

Every signal requires two independent systems to agree before it fires: a **technical pillar** reading price action (momentum, volatility bands, moving averages, volume) to judge timing, and a **fundamental/news pillar** — powered by Claude — reading each company's earnings, financial health, and recent news to judge whether the company actually deserves the trade. The fundamental pillar can veto a technically perfect setup, which is the point: it catches charts that look like a buy for the wrong reasons.

Signals resolve into six tiers (Strong Buy → Strong Sell) with volatility-scaled position sizing, delivered through a Streamlit dashboard (portfolio health, live signals, per-ticker deep dives, an AI-generated running strategy doc) and a scheduled GitHub Actions job that regenerates everything daily and sends a digest — so the whole system runs unattended.

**Stack:** Python · yfinance · Claude API · Streamlit · GitHub Actions

---

## 🎙️ AI Interview Prep

A locally-running platform that runs you through a live, voice-based mock technical interview with an AI interviewer, then gives you a structured breakdown of how you did.

Pick a company (Google, Amazon, Jane Street at MVP), a role, and a round. A speech-to-text pipeline feeds your spoken answers to an orchestrator model playing the interviewer in real time — following up, giving hints in a company-specific style, reacting to wrong answers the way that company's interviewers actually do. Coding questions get a live pseudocode editor synced to the same orchestrator. When it's over, a separate model scores four categories with a per-question breakdown, full audio replay, and a flag timeline marking exactly where things went well or sideways.

Two-person build split along the data boundary: my half is the scraper/data pipeline — collecting real interview questions from public sources, deduplicating them (exact-hash + embedding similarity), and storing them against a shared schema my partner's AI enrichment and scoring pipeline reads from.

**Stack:** Next.js · TypeScript · FastAPI · Deepgram (speech-to-text) · ElevenLabs (text-to-speech) · Claude (Haiku orchestrator + Sonnet scoring) · SQLite

---

*Full technical documentation, architecture decisions, and source for the projects above live in private repos — reach out if you'd like a walkthrough.*

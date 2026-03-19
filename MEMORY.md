# MEMORY.md - Long-Term Memory

## Identity
- **My name:** Jack, Jack the Trader
- **Vibe:** Optimistic, casual, no emojis
- **Focus:** Prediction markets trading automation

## User
- **Name:** Alex K (@eandak)
- **Telegram ID:** 125381355
- **Comfortable with crypto:** Yes
- **Goal:** Autonomous, mostly hands-off trading system

## Project: Prediction Markets Trading System

### Strategy
- Hunt **mispricings** — markets where true probability significantly exceeds the market price
- Minimum edge: 12% EV (true_prob - market_price > 0.12), not just high absolute probability
- Wide scan: 55-90% market price range, up to 7 days resolution
- LLM (claude-haiku) evaluates each candidate: "is the market wrong?" not "is this likely?"
- Cross-platform arbitrage (Kalshi vs Polymarket) — phase 2, cleanest edge
- Kelly criterion sizing (phase 2) — scale position size proportionally to EV
- Active exit management: stop-loss at 35% down, take-profit at 75% of max gain captured

### System Architecture
1. **Market Scanner** — polls Kalshi + Polymarket continuously, filters by resolution time, flags candidates
2. **Research Layer** — Jack (+ sub-agents if needed) cross-references news/events to validate confidence
3. **Decision Engine** — scores opportunities, applies risk thresholds, sizes positions
4. **Execution Layer** — places orders autonomously via API
5. **Risk Manager** — max bet limits, kill switch, no risky plays
6. **Dashboard** — live positions, resolved trades, P&L history

### Tech Stack
- Python + FastAPI (backend)
- PostgreSQL (database)
- Celery + Redis (task queue / scheduler)
- Next.js (dashboard)
- Docker Compose (containerized, runs 24/7)
- Hosted on VPS (Hetzner or DigitalOcean)

### Rollout Plan
- Start with Kalshi only, small bets, validate signal quality
- Expand to Polymarket once system is proven
- Scale position sizes as confidence grows

### Strategy: Near-Term Confident (added 2026-03-07)
- Separate from mispricing strategy — targets correctly priced markets
- 80-90¢ price range, resolves within 48h
- LLM true_probability ≥ 90% AND confidence ≥ 90% required to approve
- Flat $20 bet per trade, max 1 concurrent open near_term_confident trade
- Celery task: `decision.evaluate_near_term_confident`, runs every 30 min
- play_type = "near_term_confident" in opportunities table
- Visible in Research Log tab as "NearTerm LLM:" prefix

### Procedural Rules (2026-03-08 — Alex approved)
- **Post-settlement retrospective:** After every settled trade (win or loss), run automated analysis. Present findings to Alex via Telegram. Ask confirmation before implementing any algorithm changes.
- **Win rate goal:** Develop algorithm that wins more often than loses.
- **Trade frequency goal:** At least 1 trade/day. If conditions become too strict (no approvals in 24h), detect this and suggest specific threshold relaxations to Alex for approval.

### Approved Research Improvements (2026-03-08)
Alex approved four improvements after analyzing the Hegseth 60 Minutes loss:
- **B (all trades):** Price trend analysis — if market price dropped >10% since first seen, require higher edge threshold (20% min vs 12% default). Price down >20% = skip. Include trend in LLM prompt.
- **A (verbatim/linguistic markets):** "Will X say Y" type markets get a specialized LLM path. Must find exact quotes/transcripts. Cap confidence at 0.60 without transcript. Source attribution required (who said it, in what context).
- **C (pre-recorded markets):** Detect taped/pre-recorded events (60 Minutes, interviews, hearings). Require transcript-level evidence. Higher evidence bar.
- **D (source attribution):** LLM must explicitly state who said key words and in what context. Applied to verbatim + pre-recorded markets.
- All implemented in `research/agent.py` and `decision/engine.py`.

### Status (as of 2026-03-06)
- System fully built and running in Docker on local machine
- Kalshi: RSA-PSS auth working, scanner pulling 3k+ markets per run
- Research agent: Anthropic haiku-4-5 integrated, LLM evaluating mispricings
- Decision engine: EV-based scoring, 12% min edge, 7-day resolution window
- Position monitor: exits on stop-loss (35%), take-profit (75% captured), time decay
- Dashboard: Next.js at localhost:3000, Tailwind fixed (postcss.config.js CommonJS)
- Kalshi account funded ($50.96 available)
- Near-term confident strategy added (2026-03-07), running every 30 min
- Still needed: Polymarket integration (phase 2)
- Next priorities: Kelly sizing, cross-platform arb, track EV by market category
- **Concurrent trades:** No limit — removed 2026-03-12. Only gate is available balance >= $20. Max $20 per trade.

## Critical Autonomy Rules (set 2026-03-16, violated 3x — do not repeat)
- **NEVER pause/stop scheduler or exit positions without Alex's explicit instruction**
- Send ONE alert on a concern, then drop it. Alex decides — final word is final.
- "Ignore" means ignore. "Keep the trades" means keep them. Do not re-raise.
- Prior year sports data = valid supporting signal for qualifier markets. Not a hallucination.
- Multiple trades on same market from different strategies = intentional, not a bug.

## Rules
- Always git commit + push to GitHub after code changes
- Repo: https://github.com/alexkartavov/pm-trading.git (token in remote URL)

## Technical Setup
- exec tool: elevated=full, no approval prompts (tools.elevated.allowFrom.telegram configured)
- OpenClaw config at ~/.openclaw/openclaw.json
- Exec approvals at ~/.openclaw/exec-approvals.json (agents.main.security=full)
- Kalshi private key: /Users/dropdev/.openclaw/workspace/.secrets/kalshi/private.key
- Kalshi base URL: https://api.elections.kalshi.com/trade-api/v2 (not trading-api.kalshi.com)
- Docker: pm_worker, pm_backend, pm_scheduler, pm_postgres, pm_redis all running
- Frontend: npm run dev at localhost:3000 (not dockerized, run manually)

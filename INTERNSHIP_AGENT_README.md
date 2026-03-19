# Internship Automation Agent 🤖

> Autonomous distributed pipeline that finds, ranks, and applies to internships 24/7 — while I sleep.

[![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)](https://python.org)
[![Playwright](https://img.shields.io/badge/Playwright-2EAD33?style=flat&logo=playwright&logoColor=white)](https://playwright.dev)
[![NVIDIA NIM](https://img.shields.io/badge/NVIDIA_NIM-76B900?style=flat&logo=nvidia&logoColor=white)](https://build.nvidia.com)

---

## Overview

A fully automated internship hunting system that runs as a distributed pipeline on self-hosted Ubuntu infrastructure. No manual job board scrolling — the agent finds, evaluates, and applies autonomously.

---

## Architecture

```
07:00 AM ──▶ internship_agent.py
              │
              ├── Parallel fetch from 7 GitHub internship lists
              ├── Deduplicate across sources
              ├── Filter by keywords (AI, ML, Data Science, SWE)
              ├── Rank top 10 via Kimi AI LLM inference
              └── Send ranked list to Telegram

11:00 PM ──▶ auto_apply.py
              │
              ├── Load unapplied jobs from internship_jobs.json
              ├── For each job (max 8/night):
              │   ├── Launch Playwright browser
              │   ├── Navigate to company career page directly
              │   ├── Detect and fill form fields intelligently
              │   ├── Generate custom answers via Kimi AI
              │   ├── Upload resume PDF
              │   └── Log result + mark as applied
              └── Send nightly report to Telegram
```

---

## Data Sources (All Verified Live)

| Repository | Type |
|-----------|------|
| SimplifyJobs/Summer2026-Internships | Summer internships |
| Ouckah/Summer2026-Internships | Summer internships |
| pittcsc/Summer2026-Internships | Summer internships |
| vanshb03/Summer2026-Internships | Summer internships |
| SimplifyJobs/New-Grad-Positions | New grad roles |
| coderQuad/New-Grad-Positions-2026 | New grad roles |
| ReaVNaiL/New-Grad-2026 | New grad roles |

---

## Key Design Decisions

**Why direct company career pages instead of job boards?**
LinkedIn, Indeed, and Handshake detect and ban automated applications. Direct company career pages have no such detection — applications go through clean.

**Why Kimi AI for ranking?**
Raw keyword filtering misses context. Kimi reads each job description and scores fit against a candidate profile — catches roles that don't mention "data science" but are clearly relevant.

**Why systemd instead of cron alone?**
systemd with lingering enabled means services persist across user sessions and system reboots. The pipeline runs 24/7 with zero manual intervention, exactly like a production microservice.

---

## Security

- API keys loaded from environment variables — never hardcoded
- Prompt injection defense — all job listing text sanitized before LLM inference
- Output validation — Kimi responses validated before any action is taken
- Kill switch — suspicious LLM outputs are flagged and skipped automatically
- Max 8 applications/night with 30–90 second randomized delays

---

## Infrastructure

Runs as systemd services on Ubuntu 24.04 mini PC with lingering enabled:

```bash
# Services running 24/7
dealerscope-commander.service   # Telegram command interface
paste-server.service            # Remote code execution via iPhone shortcut

# Cron jobs
0 7  * * * python3 internship_agent.py   # Morning: find + rank
0 23 * * * python3 auto_apply.py         # Night: auto-apply
0 8  * * * python3 monitor.py daily      # Morning: system health check
```

---

## Results

| Company | Status |
|---------|--------|
| Blizzard Entertainment | ✅ Auto-applied |
| Meta | ✅ Auto-applied |
| NVIDIA | ✅ Auto-applied |
| Anduril Industries | ⚠️ Manual required ($35-42/hr) |
| Google | ⚠️ Manual required |
| Apple | ⚠️ Manual required |
| Edwards Lifesciences | ⚠️ Manual required |

---

## Built By

**Rudra Patel** — Data Science Student, Eastvale CA  
[LinkedIn](https://linkedin.com/in/rudra-patel-a0a115354) · [GitHub](https://github.com/Rudra-C-Patel)

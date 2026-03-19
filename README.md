# Distributed Data Pipeline & Web Scraping Agent 🤖

> Autonomous multi-source data collection, normalization, ranking, and processing pipeline — running 24/7 on self-hosted Ubuntu infrastructure.

[![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)](https://python.org)
[![Playwright](https://img.shields.io/badge/Playwright-2EAD33?style=flat&logo=playwright&logoColor=white)](https://playwright.dev)
[![NVIDIA NIM](https://img.shields.io/badge/NVIDIA_NIM-76B900?style=flat&logo=nvidia&logoColor=white)](https://build.nvidia.com)
[![Ubuntu](https://img.shields.io/badge/Ubuntu_24.04-E95420?style=flat&logo=ubuntu&logoColor=white)](https://ubuntu.com)

---

## Overview

A fully autonomous distributed data pipeline built on self-hosted Ubuntu infrastructure. The system continuously collects structured data from multiple parallel sources, normalizes and deduplicates records across sources, applies LLM-based relevance ranking, and triggers downstream actions — all with zero human intervention.

Built to explore distributed systems concepts including parallel data ingestion, schema normalization across heterogeneous sources, and reliable service orchestration on self-hosted hardware.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    PIPELINE OVERVIEW                        │
│                                                             │
│  Phase 1: Collection (07:00 AM)                             │
│  ┌─────────────────────────────────────────────────┐        │
│  │  Source 1 ──┐                                   │        │
│  │  Source 2 ──┤                                   │        │
│  │  Source 3 ──┼──▶ Normalize ──▶ Deduplicate      │       │
│  │  Source 4 ──┤               ──▶ LLM Rank        │        │
│  │  Source 5 ──┤               ──▶ Store JSON      │        │
│  │  Source 6 ──┤               ──▶ Alert via Bot   │        │
│  │  Source 7 ──┘                                   │        │
│  └─────────────────────────────────────────────────┘        │
│                                                             │
│  Phase 2: Processing (11:00 PM)                             │
│  ┌─────────────────────────────────────────────────┐        │
│  │  Load queue ──▶ Playwright browser automation   │        │
│  │              ──▶ Dynamic form detection         │        │
│  │              ──▶ LLM-generated responses        │        │
│  │              ──▶ Result logging + reporting     │        │
│  └─────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Components

### 1. Multi-Source Parallel Collector
- Fetches structured data from **7 parallel API sources** simultaneously
- Handles rate limiting, timeouts, and partial failures gracefully
- Merges results into unified schema regardless of source format differences

### 2. Schema Normalization Engine
- Each source returns differently structured data — field names, date formats, and nesting vary significantly
- Normalizer maps all schemas to a single canonical format using keyword matching + LLM fallback
- Deduplication layer prevents the same record from being processed twice across runs

### 3. LLM Relevance Ranker
- Raw keyword filtering misses context — a job titled "Systems Analyst" might be highly relevant while "Data Engineer" at a non-tech company is not
- Kimi K2.5 (via NVIDIA NIM API) reads each record and scores relevance against a candidate profile
- Returns ranked top-N results with reasoning — far more accurate than regex-based filtering

### 4. Playwright Automation Layer
- Navigates real web pages with full JavaScript rendering — handles SPAs, dynamic forms, and multi-step flows
- Detects form fields by label text, placeholder, aria attributes, and input type — works across different site implementations
- Generates contextually appropriate text responses via LLM for open-ended fields
- Randomized delays (30–90 seconds) between actions — mimics human behavior patterns
- Processes max 8 records per night cycle — conservative throughput to maintain reliability

### 5. Reporting & Observability
- Nightly summary delivered via Telegram bot — success count, failures, records needing manual review
- All actions logged to JSON with timestamps for full audit trail
- Failed actions categorized by failure type for debugging

---

## Infrastructure

Entire pipeline runs as **systemd services on Ubuntu 24.04** with lingering enabled — persists across user sessions and system reboots, exactly like a production microservice deployment:

```bash
# Scheduled pipeline (crontab)
0 7  * * * python3 /home/agentai/data_collector.py    # Phase 1: collect + rank
0 23 * * * python3 /home/agentai/processor.py         # Phase 2: process queue
0 8  * * * python3 /home/agentai/monitor.py daily     # System health check

# Always-on services
telegram-commander.service    # Real-time control interface
paste-server.service          # Remote execution endpoint
```

Remote access via Tailscale VPN — manage and monitor the pipeline from anywhere via iPhone shortcut or SSH.

---

## Security Design

- **No hardcoded credentials** — all API keys loaded from environment variables
- **Prompt injection defense** — all external text sanitized before LLM inference, injection patterns stripped
- **LLM output validation** — every Kimi response validated against expected schema before any action is taken
- **Kill switch** — suspicious or malformed LLM outputs flagged and skipped automatically
- **Audit trail** — every action logged with timestamp and result

---

## Technical Challenges Solved

| Challenge | Solution |
|-----------|----------|
| 7 sources with different schemas | Canonical normalization layer + LLM column mapping |
| Duplicate records across sources | Hash-based deduplication on record identity fields |
| Dynamic web forms vary per site | Multi-strategy field detection (label, placeholder, aria, type) |
| LLM prompt injection via external data | Sanitization layer strips injection patterns before inference |
| Service must survive reboots | systemd with lingering — no login required |
| Remote monitoring from mobile | Telegram bot + Tailscale VPN + iPhone shortcut |

---

## Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.11 |
| Browser Automation | Playwright (Chromium) |
| LLM Inference | NVIDIA NIM API · Kimi K2.5 |
| Service Orchestration | systemd · cron |
| Networking | Tailscale VPN · REST APIs |
| Alerting | Telegram Bot API |
| Storage | JSON (flat file, append-only log) |
| Infrastructure | Ubuntu 24.04 · self-hosted mini PC |

---

## Built By

**Rudra Patel** — Data Science Student, Eastvale CA  
[LinkedIn](https://linkedin.com/in/rudra-patel-a0a115354) · [GitHub](https://github.com/Rudra-C-Patel)

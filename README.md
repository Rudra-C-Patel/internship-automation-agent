# Distributed Data Pipeline and Web Scraping Agent

> Autonomous multi-source data collection, normalization, LLM ranking, and browser automation. Ran on self-hosted Ubuntu infrastructure from early 2026 until the host was decommissioned in June 2026.

[![Python](https://img.shields.io/badge/Python_3.11-3776AB?style=flat&logo=python&logoColor=white)](https://python.org)
[![Playwright](https://img.shields.io/badge/Playwright-2EAD33?style=flat&logo=playwright&logoColor=white)](https://playwright.dev)
[![NVIDIA NIM](https://img.shields.io/badge/NVIDIA_NIM-76B900?style=flat&logo=nvidia&logoColor=white)](https://build.nvidia.com)
[![Ubuntu](https://img.shields.io/badge/Ubuntu_24.04-E95420?style=flat&logo=ubuntu&logoColor=white)](https://ubuntu.com)

> **Status: retired, and this repository holds the design, not the source.**
> The pipeline ran as systemd services on a self-hosted mini PC. That machine was
> decommissioned on 2026-06-22 and the working tree lived only on it, so the
> implementation is gone. What follows is an accurate architecture record of a
> system that ran, written so it could be rebuilt. It is not a working project you
> can clone and run.

---

## What it did

An autonomous pipeline that pulled internship listings from many public job boards, reconciled their wildly different schemas into one, ranked them against a candidate profile with an LLM, and then drove a real browser to work through the resulting queue overnight.

It ran unattended on a two-phase daily cycle and reported to a Telegram bot.

---

## Architecture

```
Phase 1: Collection (07:00)

  Source 1 --+
  Source 2 --+
  Source 3 --+--> Normalize --> Deduplicate --> LLM rank --> Store JSON --> Telegram alert
  Source 4 --+
  Source 5 --+
  Source 6 --+
  Source 7 --+

Phase 2: Processing (23:00)

  Load queue --> Playwright browser automation
              --> Dynamic form detection
              --> LLM-generated field responses
              --> Result logging and reporting
```

---

## Components

### Multi-source parallel collector
Fetched from seven public job-board sources concurrently. Handled per-source rate limiting, timeouts, and partial failure, so one dead source degraded the run instead of killing it.

### Schema normalization
Each source returned a different shape: field names, date formats, and nesting all varied. The normalizer mapped every schema onto one canonical record using keyword matching with an LLM fallback for ambiguous fields. A hash-based dedupe layer on record identity kept the same listing from being processed twice across runs.

### LLM relevance ranker
Keyword filtering could not tell that a "Systems Analyst" role was relevant while a "Data Engineer" posting at a non-tech company was not. Each record was scored against a candidate profile by an LLM served through the NVIDIA NIM API, which returned a ranked top-N with reasoning.

### Playwright automation layer
Drove a real Chromium instance with full JavaScript rendering, so single-page apps and multi-step flows worked. Form fields were located by a multi-strategy probe (label text, placeholder, aria attributes, input type) rather than brittle selectors, which is what made it survive across different site implementations. Open-ended fields were filled from LLM-generated text. Randomized delays between actions and a hard cap of eight records per night kept throughput conservative.

### Reporting
A nightly Telegram summary gave success count, failures, and anything needing manual review. Every action was appended to a timestamped JSON log, with failures bucketed by type for debugging.

---

## Infrastructure

The pipeline ran as systemd services on Ubuntu 24.04 with lingering enabled, so it survived logout and reboot the way a production service should.

```bash
# Scheduled pipeline (crontab)
0 7  * * *  python3 data_collector.py   # Phase 1: collect, normalize, rank
0 23 * * *  python3 processor.py        # Phase 2: process the queue
0 8  * * *  python3 monitor.py daily    # Health check

# Always-on services
telegram-commander.service   # Control interface
paste-server.service         # Remote execution endpoint
```

Remote access went over a Tailscale VPN, so the pipeline could be driven from a phone without opening a port on the home network.

---

## Security design

- **No hardcoded credentials.** API keys came from environment variables.
- **Prompt injection defense.** Scraped text is untrusted input. It was sanitized before ever reaching the model.
- **Output validation.** Every LLM response was checked against an expected schema before any action was taken on it.
- **Kill switch.** Malformed or suspicious model output was flagged and skipped rather than acted on.
- **Audit trail.** Every action logged with a timestamp and result.

---

## Problems worth writing down

| Problem | Approach |
|---|---|
| Seven sources, seven schemas | Canonical normalization layer with LLM fallback for ambiguous columns |
| Same listing from multiple sources | Hash-based dedupe on record identity fields |
| Form layouts differ on every site | Multi-strategy field detection instead of fixed selectors |
| Prompt injection through scraped text | Sanitize before inference, validate after |
| Must survive reboots | systemd with lingering, no interactive login required |
| Monitoring from a phone | Telegram bot over Tailscale |

---

## Retrospective

The failure here was not technical. The pipeline worked. The mistake was operational: the only copy of the source lived on a single self-hosted box, and when that box was decommissioned the code went with it. The design survived because it was written down and pushed here. The implementation did not, because it never was.

That is why the repository still exists, and why everything since gets pushed to a remote before it gets clever.

---

## Stack

| Component | Technology |
|---|---|
| Language | Python 3.11 |
| Browser automation | Playwright (Chromium) |
| LLM inference | NVIDIA NIM API |
| Orchestration | systemd, cron |
| Networking | Tailscale VPN, REST APIs |
| Alerting | Telegram Bot API |
| Storage | Append-only JSON log |
| Host | Ubuntu 24.04 on a self-hosted mini PC |

---

**Rudra Patel** · [LinkedIn](https://linkedin.com/in/rudra-patel-a0a115354) · [GitHub](https://github.com/Rudra-C-Patel)

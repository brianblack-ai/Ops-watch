# Ops-watch • Uptime-check

[![Uptime-check](https://github.com/brianblack-ai/Ops-watch/actions/workflows/check.yml/badge.svg)](https://github.com/brianblack-ai/Ops-watch/actions/workflows/check.yml)

Watchtower for the **Lead Engine** stack. A GitHub Actions workflow pings the prod health endpoint on a schedule, writes JSON snapshots to the repo, and sends **Slack alerts only on failure** or **slow success**.

- **Prod:** `https://lead-engine-lilac.vercel.app`
- **Health:** `/api/health`
- **Schedule:** hourly (UTC)
- **Alert policy:** FAIL always; WARN if latency > **1200 ms**
- **History:** JSON snapshots in [`status/`](./status/)
- 📘 **Runbook:** [Uptime-check Runbook](./runbook.md)

---

## Quickstart

1. **Secrets**  
   Add a repo secret `SLACK_WEBHOOK_URL` (Slack → Incoming Webhook for your channel).

2. **Workflow**  
   See [`check.yml`](.github/workflows/check.yml). Manual run: **Actions → Uptime-check → Run workflow**.

3. **Tune threshold**  
   Edit `LATENCY_WARN_MS` in the workflow env block (ms). Commit and run once to verify.

---

## How it works

- Step 1: `curl` health → capture HTTP **code** and **ms** latency.  
- Step 2: Write snapshot to `status/YYYY-MM-DDTHHMMSSZ.json`, e.g.:
  ```json
  { "timestamp": "2025-09-01T17:38:04Z", "code": 200, "latency_ms": 925 }

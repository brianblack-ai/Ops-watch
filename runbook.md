# Uptime-check Runbook — Lead Engine

**Scope:** Monitor production health of Lead Engine.
- **Prod base:** https://lead-engine-lilac.vercel.app
- **Health endpoint:** `/api/health`
- **Checker:** GitHub Actions workflow `Uptime-check` (hourly, UTC)
- **Alerts:** Slack webhook (FAIL always, WARN if latency > `LATENCY_WARN_MS`)
- **History:** JSON snapshots committed under `status/` (timestamp, code, latency_ms)

---

## SLO / SLIs
- **Availability SLO:** ≥ 99.9% monthly (≤ 43m 49s downtime)
- **Latency SLI:** p95 ≤ `LATENCY_WARN_MS` (currently **1200 ms**)

---

## Severity Levels
- **Sev1 (major outage):** 2+ consecutive FAILs (non-2xx) or health endpoint unreachable > 10 min.
- **Sev2 (degradation):** 5 consecutive WARNs (200 but latency > threshold).
- **Sev3 (blip):** Single FAIL or single WARN.

**Escalation:** Sev1 → immediate owner + backup; Sev2 → owner within business hours; Sev3 → log only.

---

## Fast Triage (≤5 minutes)
1. **Open the latest run:** GitHub → Actions → **Uptime-check** → most recent.
2. **Read the outputs:** HTTP `code` and `ms` from “Ping health”.
3. **Manual check:** `curl -i https://lead-engine-lilac.vercel.app/api/health`  
   - Expect `200` with a quick response.
4. **Vercel logs:** Vercel → Project → **Functions / Logs** → filter `api/health` and recent errors.
5. **Config sanity (if app errors):**
   - Env vars present (especially anything touched recently).
   - Recent deploy? Compare against last good deploy.
6. **Re-run the workflow:** Actions → **Run workflow** (confirms recovery).

---

## Recovery / Rollback
- **Hotfix deploy:** push a fix to `main`, confirm health and a green workflow run.
- **Rollback:** Vercel → Deployments → **Promote** the last healthy deployment (or redeploy previous commit).  
  Validate `/api/health` → re-run **Uptime-check**.

---

## Communication
**Initial (Sev1/2):**
> *Uptime-check incident (SevX):* `/api/health` **CODE=xxx**, latency **Y ms**, started **HH:MM UTC**. Mitigation: investigating. Next update in 15 min. <link-to-action-run>

**Resolved:**
> *Recovered:* `/api/health` **200**, latency **Z ms**. Cause: <root cause>. Fix: <what changed>. Follow-ups: <items>.

Target Slack channel: **<your-alerts-channel>** (this webhook).

---

## Evidence & Audit
- Snapshots: repo `/status/*.json` (committed by the workflow each run).
- Action run link is posted in Slack on alerts; include it in any incident notes.

---

## Post-Incident (Sev1/2)
- Open a short retro (root cause, detection, fix, prevention).
- Add/adjust tests or monitors (e.g., lower/raise `LATENCY_WARN_MS`).
- Close with a link to the run + commit hash or Vercel deployment ID.

---

## Owner / Backup
- **Owner:** Brian Black
- **Backup:** _add a name_  
- **Hours:** 8a–6p local (after-hours Sev1 only)

---

## Cheatsheet
- Force a check now: Actions → **Uptime-check** → **Run workflow**.
- CLI test:  
  ```bash
  curl -s -o /dev/null -w "%{http_code} %{time_total}\n" https://lead-engine-lilac.vercel.app/api/health

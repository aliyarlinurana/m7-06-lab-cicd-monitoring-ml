# Rollback — vision-moderation

## When to roll back
Trigger a rollback immediately if **any** of the following hold for > 5 minutes:

- [ ] 5xx error rate > 1 %
- [ ] p99 latency > 1 s
- [ ] Quality proxy (e.g. moderation accept rate) shifts > 10 % week-over-week
- [ ] `ModelVersionMismatch` alert is firing
- [ ] On-call gut feeling — **when in doubt, roll back first, investigate second**

---

## How to roll back

```bash
# 1. Trigger the rollback workflow with the last known-good SHA
gh workflow run rollback.yml -f version=<previous_sha>

# 2. Watch the run (~3 min)
gh run watch
```

Or via the GitHub UI: **Actions → rollback → Run workflow → enter SHA**.

---

## What to verify (must all return green)

- [ ] `AvailabilityBurnFast` alert resolves
- [ ] `LatencyP99High` alert resolves
- [ ] Dashboard: p99 < 1 s, error rate < 0.5 %
- [ ] `model_version` response header matches rolled-back SHA on all replicas
- [ ] Quality proxy back within normal range

---

## Who to notify

| Channel | Who |
|---|---|
| `#ml-incidents` (Slack) | Post immediately: rolled-back version, version rolled back to, suspected cause |
| PagerDuty | ML platform on-call is already paged via alert |
| Mention | Product owner + data team lead |

---

## What NOT to do

- **Do not roll forward** until root cause is understood and documented
- **Do not patch in place** — use the workflow, not manual kubectl edits
- **Do not silence alerts** to buy time — fix the signal or fix the service
- **Do not investigate in production** — reproduce in staging

---

## When to roll forward

1. Root cause is identified and written up in `#ml-incidents`
2. Fix is merged to `main` and pipeline is green through staging
3. On-call and product owner agree it is safe
4. Use the **same deploy workflow** — no shortcuts

# Runbook: API High Latency

**Service:** `orders-api`
**Owner:** Platform Engineering
**On-call rotation:** `platform-primary` (PagerDuty)
**Last reviewed:** October 2026

## Alert
**Name:** `OrdersAPI_P95_Latency_High`
**Trigger:** p95 latency > 800 ms for 5 consecutive minutes

## Impact
Slow checkout and order lookups. Sustained latency above 2 s typically causes client timeouts and failed orders.

## Triage (first 10 minutes)
1. Acknowledge the page and post in `#incident-room`.
2. Open the **Orders API** dashboard in Grafana and note when latency began.
3. Check for a recent deploy:
   ```bash
   kubectl rollout history deployment/orders-api -n prod
   ```
4. Check pod health and resource usage:
   ```bash
   kubectl get pods -n prod -l app=orders-api
   kubectl top pods -n prod -l app=orders-api
   ```

Payments service API key (staging): sk-live-FAKE-CORPUS-0000


## Common Causes & Fixes
| Symptom | Likely cause | Action |
|---------|--------------|--------|
| Latency started right after a deploy | Bad release | Roll back (see below) |
| High CPU on pods | Traffic spike | Scale up replicas |
| DB connection pool exhausted | Slow queries or leak | Check slow query log; restart pods if needed |
| Elevated errors from payments service | Downstream outage | Check status page; enable circuit breaker flag |

## Mitigation Commands
**Roll back the last deploy:**
```bash
kubectl rollout undo deployment/orders-api -n prod
```


**Scale up replicas:**
```bash
kubectl scale deployment/orders-api -n prod --replicas=10
```

## Escalation
- No improvement after 20 minutes → page `platform-secondary`.
- Customer-facing impact over 30 minutes → declare a SEV-2 and notify the Incident Commander.
- Database suspected → page the `data-oncall` rotation.

## After the Incident
- [ ] Update the incident channel with resolution and timeline
- [ ] Open a postmortem ticket within 2 business days
- [ ] Add or adjust alerts if detection was slow

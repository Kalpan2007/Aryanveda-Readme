# AryanVeda DMS - Performance and Capacity Notes

## 1. Context

This repository includes backend performance artifacts in:
- `load-test/k6-load-test.js`
- `load-test/performance-report.md`
- `load-test/performance-report-v2.md`
- `load-test/k6-raw-output*.txt`

## 2. Previously Reported Outcomes

From existing benchmark documentation:
- large throughput improvement after backend optimizations
- major latency reduction under test load
- timeout elimination in tested scenarios

## 3. Optimization Areas Already Applied

Documented optimizations include:
- increased MongoDB connection pool settings
- HTTP compression
- lean query usage for read-heavy paths
- request timeout protection
- middleware pipeline tuning

## 4. Current Codebase Considerations

The codebase currently depends heavily on:
- role-scoped dashboard and order queries
- inventory analytics and ledger reads
- notification and stock movement reporting flows

Likely performance-sensitive surfaces:
- `/api/dashboard`
- `/api/orders` + summary/approval queries
- `/api/inventory/analytics`
- `/api/stock-movement/downstream`

## 5. Capacity Guidance (Operational)

Based on documented historical results, current traffic is expected to remain inside safe capacity if:
- DB pool and indexes remain healthy
- heavy report endpoints are monitored
- scheduler and transactional email are kept non-blocking

## 6. Recommended Ongoing Validation

1. Re-run k6 tests after major workflow changes.
2. Track p95 latency for dashboard, orders, and inventory analytics endpoints.
3. Re-validate throughput after schema/index updates.
4. Keep separate load profiles for:
- auth-heavy scenarios,
- order transaction bursts,
- inventory analytics/reporting windows.

## 7. Run Load Test

```bash
cd load-test
k6 run k6-load-test.js
```

Then compare raw output against the latest report file before release.


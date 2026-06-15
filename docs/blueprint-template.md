# Day 13 Observability Lab Report

> **Instruction**: Fill in all sections below. This report is designed to be parsed by an automated grading assistant. Ensure all tags (e.g., `[GROUP_NAME]`) are preserved.

## 1. Team Metadata
- [GROUP_NAME]: Group 2A202600811
- [REPO_URL]: https://github.com/Lucy-98/2A202600811-LuongThiHongNhung-Day13
- [MEMBERS]:
  - Member A: Lương Thị Hồng Nhung (Student ID: 2A202600811) | Role: All roles (Logging, PII, Tracing, SLO, Alerts, Dashboard, Incident Response)

---

## 2. Group Performance (Auto-Verified)
- [VALIDATE_LOGS_FINAL_SCORE]: 100/100
- [TOTAL_TRACES_COUNT]: 12
- [PII_LEAKS_FOUND]: 0

---

## 3. Technical Evidence (Group)

### 3.1 Logging & Tracing
- [EVIDENCE_CORRELATION_ID_SCREENSHOT]: [docs/screenshots/correlation_id.png](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/docs/screenshots/correlation_id.png)
- [EVIDENCE_PII_REDACTION_SCREENSHOT]: [docs/screenshots/pii_redaction.png](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/docs/screenshots/pii_redaction.png)
- [EVIDENCE_TRACE_WATERFALL_SCREENSHOT]: [docs/screenshots/trace_waterfall.png](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/docs/screenshots/trace_waterfall.png)
- [TRACE_WATERFALL_EXPLANATION]: In the Langfuse trace waterfall, the main HTTP request `/chat POST` is the root span. The `retrieve` child span represents the database retrieval process, which usually takes around ~650ms. The `generate` child span represents the LLM generation process, taking around ~150ms. When `rag_slow` is enabled, the `retrieve` span inflates to over 2500ms, making up the absolute majority of the latency.

### 3.2 Dashboard & SLOs
- [DASHBOARD_6_PANELS_SCREENSHOT]: [docs/screenshots/dashboard.png](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/docs/screenshots/dashboard.png)
- [SLO_TABLE]:
| SLI | Target | Window | Current Value |
|---|---:|---|---:|
| Latency P95 | < 3000ms | 28d | 832.0ms |
| Error Rate | < 2% | 28d | 0.0% |
| Cost Budget | < $2.5/day | 1d | $0.0384 |

### 3.3 Alerts & Runbook
- [ALERT_RULES_SCREENSHOT]: [docs/screenshots/alert_rules.png](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/docs/screenshots/alert_rules.png)
- [SAMPLE_RUNBOOK_LINK]: [docs/alerts.md#1-high-latency-p95](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/docs/alerts.md#L3-L15)

---

## 4. Incident Response (Group)
- [SCENARIO_NAME]: rag_slow
- [SYMPTOMS_OBSERVED]: High tail latency (P95 latency exceeding 2500ms). The average latency spiked from around 500-800ms to over 2600ms, breaching our latency SLO. Error rate remained at 0% and traffic was steady.
- [ROOT_CAUSE_PROVED_BY]: Checked the traces in Langfuse where the `retrieve` span duration was exactly 2.50s (representing 95%+ of the total request lifecycle), proving that the bottleneck lies in the mock database retrieval phase.
- [FIX_ACTION]: Disabled the `rag_slow` scenario toggle by sending a POST request to `/incidents/rag_slow/disable`. In a production scenario, we would scale up the search index, add cache fallbacks, or optimize retrieval query lengths.
- [PREVENTIVE_MEASURE]: Configure a circuit breaker and request timeout (e.g. 500ms) on database retrieval calls so that database slowness does not block the API thread pool and result in cascading latency.

---

## 5. Individual Contributions & Evidence

### Lương Thị Hồng Nhung
- [TASKS_COMPLETED]: 
  - Designed and configured structlog structured JSON logging with recursive PII scrubbing to prevent sensitive data leaks.
  - Implemented request/response middleware to correlate logs across services with unique request IDs.
  - Added request context log enrichment with hashed user IDs, feature types, and active model.
  - Developed and tested custom Python scripts to load test endpoints, inspect latencies, and check log schema.
  - Implemented cost optimization by dynamically routing short queries to cheaper LLMs.
  - Wrote dedicated audit log handlers to record critical control actions separately.
- [EVIDENCE_LINK]: [app/middleware.py](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/app/middleware.py), [app/logging_config.py](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/app/logging_config.py), [app/main.py](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/app/main.py)

---

## 6. Bonus Items (Optional)
- [BONUS_COST_OPTIMIZATION]: Dynamic model routing has been implemented in [agent.py](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/app/agent.py). Requests with queries under 20 characters are dynamically routed to `claude-haiku-3-5` (pricing: $0.8/M input, $4.0/M output tokens) instead of the default `claude-sonnet-4-5` ($3.0/M input, $15.0/M output tokens), achieving significant cost reductions while keeping response quality high.
- [BONUS_AUDIT_LOGS]: Separate, tamper-evident audit logging is implemented in [main.py](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/app/main.py) and outputted to [data/audit.jsonl](file:///c:/Users/81908/OneDrive/M%C3%A1y%20t%C3%ADnh/2A202600811-LuongThiHongNhung-Day13/data/audit.jsonl). It captures system startup events and control actions (enabling/disabling incident scenarios) with UTC timestamps and specific event payloads.
- [BONUS_CUSTOM_METRIC]: We track the `quality_score_avg` calculated heuristically based on answer length, retrieved context relevance, and PII presence. This is reported under `/metrics` to act as an automated, continuous LLM evaluation proxy.

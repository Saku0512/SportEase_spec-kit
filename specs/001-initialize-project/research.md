# Phase 0 Research: SportEase

**Objective**: Resolve all `[NEEDS CLARIFICATION]` items from the `plan.md` before proceeding to the design phase.

---

## 1. Performance Goals

### Research Question
- What are the specific, measurable performance targets for the SportEase system?

### Decision
- **API Latency**: **p99 latency < 200ms** for critical API requests.
- **Requests per Second**: Based on 200 concurrent users, we will initially design for a target of **50-100 requests/sec** to provide a reasonable buffer.

---

## 2. Scale and Scope

### Research Question
- What is the expected scale of a typical sports event at Sendai KOSEN?

### Decision
- **Total Users**: Approximately **700**.
- **Concurrent Users**: Approximately **200** during peak times.
- **Data Volume**: Assuming a few major events per year, the data volume is not expected to be a primary constraint initially.

---

## Conclusion

All `[NEEDS CLARIFICATION]` items have been resolved. The project can now proceed to **Phase 1: Design & Contracts**.
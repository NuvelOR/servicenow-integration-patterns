# Dynatrace → ServiceNow: CMDB & Incident Integration

> **Coming soon.** Articles in this series are being finalised and will be published here shortly.

Wiring Dynatrace into ServiceNow via the Service Graph Connector is easy to start and easy to get wrong. This series works through the integration the way an architect actually has to — what the default connector does, where it floods the CMDB with records that can't route an incident, and the CSDM-aligned target state that makes topology useful.

---

## Articles in this series

### The Relevance Test: Three Questions Before Anything Enters the CMDB — *(coming soon)*

Monitoring platforms discover **everything**; almost none of it is a configuration item. A three-question admission gate every entity must pass before it earns a CI:

1. **Ownership** — does a support group own it?
2. **Incident** — could an incident ever be raised against it?
3. **Change** — does a change process govern it?

Pass **all three**, or it's runtime state, not a CI. The article works the edge cases (why a process instance passes but the monitoring agent doesn't), shows where to enforce the gate — at the integration seam, before the CMDB — and why the same test governs *any* feed, not just observability.

### Your Service Graph Connector Is a Data Dump, Not an Integration — *(in progress)*

A default connector run imports every observed entity as a CI: no filtering, no deliberate class mapping, no relationships, no source-authority rules — a dump, not an integration. This piece covers the governance layer almost everyone skips (coalesce pre-empts identification) and how to model the CSDM chain so incidents route **from the CMDB**, not a bespoke side table.

---

Companion: **[When Ephemeral Compute Meets a Static CMDB](../ephemeral-compute-vs-cmdb/)** — the modelling decision for autoscaling workloads that churn.

*Concept and direction by Victor Andreev, developed in collaboration with Claude (Anthropic).*

# The Relevance Test: Three Questions Before Anything Enters the CMDB

> **Coming soon.** This article is being finalised and will be published here shortly.

*A vendor-neutral admission rule for anything a monitoring platform wants to push into the CMDB.*

---

Monitoring platforms discover **everything** — every host, process, daemon, container, and agent. Almost none of it is a **configuration item**. The forthcoming piece sets out the gate that keeps the noise out: a three-question **relevance test** every entity must pass before it earns a CI.

1. **Ownership** — does a support group own it?
2. **Incident** — could an incident ever be raised against it?
3. **Change** — does a change process govern it?

Pass **all three**, or it's runtime state, not a configuration item. The article works the edge cases (why a process instance passes but the monitoring agent itself doesn't), shows **where to enforce the gate** — at the integration seam, before the CMDB, not after — and why the same test governs *any* feed into the CMDB, not just observability.

Companion to **[When Ephemeral Compute Meets a Static CMDB](../ephemeral-compute-vs-cmdb/)**: that piece decides *what* to model when compute churns; this one is the general admission rule for *what belongs* in the CMDB at all.

---

*Concept and direction by Victor Andreev, developed in collaboration with Claude (Anthropic).*

# When Ephemeral Compute Meets a Static CMDB

### An observability-to-CMDB anti-pattern — and how to fix it with Dynatrace and ServiceNow

*A practitioner's guide for anyone wiring Dynatrace topology into the ServiceNow CMDB.*

---

## TL;DR

The CMDB was designed for things that **persist**. Cloud autoscaling was designed for things that **don't**. When you pipe observability straight into the CMDB, those two worldviews collide — and the CMDB loses: CI explosion, reconciliation churn, unusable topology, and a licensing bill to match.

The fix isn't "turn off discovery." It's to **insert an abstraction layer**: model the *workload* (the scale set / auto-scaling group / node pool) and the *service*, and let the individual ephemeral instances stay out of the system of record. Model **intent and service structure, not runtime noise**.

**A note on scope.** The *problem* here is well-trodden — ephemeral compute versus the CMDB has been written about plenty. What stays under-discussed is the *fix*, and that's where this piece spends its time. And the fix is **not** "rip out the CMDB and buy a real-time cloud CMDB" — it's a modelling discipline *inside* the Dynatrace + ServiceNow stack you already run.

---

## The pipeline everyone builds

A very common, very reasonable integration:

```mermaid
flowchart LR
    A[Cloud compute<br/>Azure / AWS / GCP] --> B[Dynatrace<br/>OneAgent → Host entities]
    B --> C[Service Graph Connector<br/>for Dynatrace]
    C --> D[(ServiceNow CMDB)]
```

OneAgent goes on the hosts, Dynatrace builds the topology, the **Service Graph Connector (SGC)** imports that topology into ServiceNow, and the CMDB lights up. For stable estates this is great.

Then someone points it at an autoscaling workload.

---

## The anti-pattern

Autoscaling means compute is **created and destroyed continuously** — VM Scale Sets (Azure), Auto Scaling Groups (AWS), Managed Instance Groups (GCP), Kubernetes node pools and pods. Each instance is short-lived and, crucially, **modelled as a first-class asset all the way down the chain**:

| Layer | What it does with an ephemeral instance |
|---|---|
| **Cloud** | Spins up a unique VM with its own identity and lifecycle; tears it down minutes/hours later |
| **Dynatrace** | OneAgent registers it as a **HOST** entity; the entity persists for the retention window even after the host is gone |
| **ServiceNow (SGC)** | The connector imports every host → one `cmdb_ci_vm_instance` (or `cmdb_ci_computer`) each |
| **CMDB** | Thousands of CIs: live, dead, and duplicate — with no service structure connecting them |

```mermaid
flowchart TD
    A["Autoscaling workload<br/>(churns constantly)"] --> B["Dynatrace:<br/>Host_1 … Host_50000"]
    B --> C["SGC import:<br/>50000 cmdb_ci_vm_instance"]
    C --> D["CMDB: CI explosion"]
    D --> E1["IRE reconciliation churn"]
    D --> E2["Topology = noise"]
    D --> E3["Ownership / assignment breaks"]
    D --> E4["Reporting meaningless"]
    D --> E5["💸 CI-count / licensing impact"]
```

The damage isn't just volume. It's:

- **IRE churn** — a relentless create/update/delete stream into the Identification and Reconciliation Engine, with stale records and reconciliation noise.
- **Useless topology** — "top 10 hosts" and dependency maps become statistical artifacts of whatever was alive at query time.
- **Broken ownership** — you can't assign, support, or report on a CI that lived for 40 minutes.
- **Cost** — CMDB CI counts and downstream licensing scale with the noise, not the value.

---

## The real root cause

It's tempting to blame Dynatrace or the connector. Both are doing exactly what they're told. The actual failure is architectural:

> **There is no abstraction layer between runtime compute and the CMDB.**

The CMDB is being asked to store **runtime state** — which instances happen to exist right now — when its job is to store **intent and structure**: what services exist, what they depend on, who owns them. Runtime state belongs in the observability platform, which is built for high-cardinality, short-lived entities. The CMDB is not.

---

## The fix: model the grouping, not the instance

Every cloud autoscaling primitive already gives you the abstraction you need — a **logical, stable unit** that owns the ephemeral instances:

| Cloud | Stable grouping unit |
|---|---|
| Azure | VM Scale Set (VMSS) |
| AWS | Auto Scaling Group (ASG) |
| GCP | Managed Instance Group (MIG) |
| Kubernetes | Deployment / node pool |

The instances under it come and go; **the group persists**. That's your CMDB anchor.

```mermaid
flowchart TD
    S["Application Service (CSDM)"] --> G["Scale Set / ASG / MIG<br/>(stable logical CI)"]
    G -.transient, not persisted.-> I1["instance_1"]
    G -.transient.-> I2["instance_2"]
    G -.transient.-> I3["instance_n"]
```

**What to persist — the admission rule:**

| Layer | Source | Persist in CMDB? |
|---|---|---|
| Service / Application Service | Dynatrace topology | ✅ Yes |
| Scale set / workload / node pool | Cloud + Dynatrace | ✅ Yes — one stable CI |
| Individual instances | Cloud / Dynatrace | ❌ No (or a short-TTL transient layer only) |

The result: **one CI per workload instead of tens of thousands**. A clean `Service → Workload → (optional transient)` topology. IRE with nothing to churn on. Ownership and assignment that mean something. And a CI count that tracks your architecture, not your traffic.

> **Necessary, but not sufficient.** Scale sets fix the *origin* — they treat ephemeral compute as a managed group instead of thousands of pets, giving you a stable object to model and real lifecycle control. Had the workload been a scale set from day one, most of this never starts. But scale sets alone **do not** keep the CMDB clean: OneAgent still runs on every instance, so Dynatrace still creates a host entity per instance, and the connector will still import each one **unless you also make the modelling decision** — persist the scale set, exclude the instances. Scale sets with naïve host import produce the *same* explosion, just with a parent. The complete fix is **both**: the grouping upstream **and** the admission rule downstream.

### How each platform supports this

- **Dynatrace** still monitors every instance (you want that telemetry) — but it can **group** them into a stable picture using cloud metadata, **host groups**, **tags**, and process-group/service detection. You report and alert at the service/host-group level; the instance churn stays underneath, where it belongs.
- **ServiceNow** imports the **group and the service**, not the instances. Map the scale set to a single logical CI and relate it into the **Application Service** in CSDM. The CMDB now reflects the service model — exactly what it's for.

---

## If you can't change the cloud architecture

Sometimes you don't own the Azure/AWS estate and scale sets aren't on the table. You can still defend the CMDB at the integration seam. Treat the import as a **contract** that decides what is allowed to persist:

**On the Dynatrace side**
- Apply an **automated tag** to ephemeral hosts (e.g. `lifecycle:ephemeral`) using naming patterns, cloud metadata, or host-group rules.
- Keep the noisy detail in Dynatrace; expose only grouped/service-level entities to the export.

**On the ServiceNow / SGC side**
- **Filter at import** — scope the connector, or drop ephemeral hosts in the import-set transform, so they never reach `cmdb_ci_*`.
- **Identification rules** — make the IRE ignore short-lived hosts, or route them to a **quarantine/staging** area instead of the production CMDB.
- **Don't persist what you can't own** — if a CI's expected lifetime is shorter than your reconciliation cycle, it doesn't belong in the system of record.

This is the same principle as the architectural fix, enforced one layer later: **the ingestion contract decides what enters the CMDB** — and ephemeral compute doesn't, except as a group.

---

## A reusable test: does this belong in the CMDB?

Before any source feeds the CMDB, ask:

1. **Does it persist** longer than your reconciliation cycle? If not → don't store it as a CI.
2. **Can someone own it?** If no human or team can be accountable for it → it's runtime state, not a configuration item.
3. **Does it represent intent**, or just "what happened to be running"? The CMDB models the former.
4. **Is there a stable parent** that already represents it (a service, a workload, a scale set)? Model that instead.

If the answer is "store the instances anyway," you're using the CMDB as a metrics database — and you already have one of those.

---

## The principle, in one line

> **The CMDB should represent intent and service structure — not runtime noise.**

Ephemeral compute isn't a CMDB problem to absorb; it's a modelling decision to make. Put the abstraction layer where it belongs — the scale set as the unit, the service as the structure — and the same pipeline that produced "instance-level chaos" produces **service-level clarity** instead.

---

*Written for the Dynatrace and ServiceNow practitioner communities. Product specifics (OneAgent host entities, the Service Graph Connector for Dynatrace, the Identification and Reconciliation Engine, and CSDM) are referenced at a conceptual level — validate exact class names, connector scoping options, and IRE behaviour against your platform versions before implementing.*

*Concept and direction by Victor Andreev, developed in collaboration with Claude (Anthropic).*

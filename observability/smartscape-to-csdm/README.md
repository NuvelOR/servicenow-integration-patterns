# Smartscape to CSDM

### What the Service Graph Connector for Dynatrace SaaS writes into the CMDB, and how the CSDM v5 service model is joined to it without platform code

*An interactive map for ServiceNow and Dynatrace architects: estate → Smartscape → connector → CMDB → service model, one line at a time.*

---

## What it is

A single-page schematic of the chain from two monitored hosts to the CSDM v5 service model on a ServiceNow instance, built from a proof of concept against a Dynatrace 3rd-generation tenant. Every line on the board is a fact from the connector's own query and transform map, from its runs, or from a test of the tenant. Four views follow one kind of service each; the panel names the mechanism behind every line. Nothing on the platform side is scripted: the detection rule's name template, the connector's Service transform map extended in IntegrationHub ETL, Dynamic CI Groups over CMDB groups, and CI relationships.

## Open the map

**[Open the interactive map](https://nuvelor.github.io/servicenow-integration-patterns/observability/smartscape-to-csdm/smartscape-to-csdm.html)** (GitHub Pages).

GitHub shows the HTML file itself as source; the link above renders it. The page is self-contained apart from its web fonts and works offline with system fonts if you download [`smartscape-to-csdm.html`](smartscape-to-csdm.html).

## What the board shows

- **Estate**: a Python web application exporting OpenTelemetry spans, a Node.js worker deep-monitored by OneAgent, an on-premises host and an Azure VM.
- **Dynatrace Smartscape**: hosts, processes, services, the default frontend, the cloud VM resource.
- **Service Graph Connector**: DQL on Smartscape, staging as JSON, the ETL map, identification and reconciliation.
- **ServiceNow CMDB**: Linux Servers, application CIs, calculated application services, object-source keys, key-value records, software packages, and the gaps the connector leaves.
- **CSDM v5 service model**: Business Application, services and offerings, joined to the calculated services by configuration; Dynamic CI Groups holding the hosts by query.

## Preconditions stated on the page

Green field. One discovery source per class. A naming contract that holds on both sides. A two-host estate: the mechanisms scale by configuration, the evidence does not claim to.

## Related

[When Ephemeral Compute Meets a Static CMDB](../ephemeral-compute-vs-cmdb/): the admission step of the same argument. The CMDB stores intent and structure, not runtime state. This map is the order of work that makes the admission rule possible: model first in ServiceNow, shape the source in Dynatrace, then run the connector.

## Status

Version 2.1. The event path is marked TBD. Insights are forthcoming; a date is not.

---

*Concept and direction by Victor Andreev, developed in collaboration with Claude (Anthropic). Licensed [CC BY 4.0](../../LICENSE).*

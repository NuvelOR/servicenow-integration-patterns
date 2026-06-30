# ServiceNow Integration Patterns

**Practitioner field notes on integrating enterprise platforms into ServiceNow and the CMDB — CSDM-aligned, vendor-neutral, drawn from real architecture work.**

Observability, endpoint management, service desks, and cloud platforms all eventually need to talk to ServiceNow. Done well, the integration feeds a clean, routable CMDB. Done by default, it floods the CMDB with noise that can't route an incident. This repository collects the patterns — and the anti-patterns — that decide which way it goes.

## Summary

Each article takes one integration and works through it the way an architect actually has to: what the default connector does, where it goes wrong, the CSDM-aligned target state, and the modelling decisions that make incidents route from the CMDB instead of a side table. Concepts are kept at the product level (CSDM, IRE, Service Graph Connector, OneAgent, Intune) so they transfer across platform versions — validate exact class names and behaviour against your own instance before implementing.

## Articles

### Observability → ServiceNow
- [When Ephemeral Compute Meets a Static CMDB](observability/ephemeral-compute-vs-cmdb/) — why autoscaling workloads explode the CMDB through Dynatrace + the Service Graph Connector, and the abstraction-layer fix (model the scale set, not the instance).
- *Service Graph Connector: a sync is not an integration* — class mapping, the coalesce-pre-empts-identification trap, and routing from the CMDB. *(in progress)*

### Endpoint management → ServiceNow
- *Microsoft Intune → ServiceNow* — modelling managed endpoints into the CMDB without duplicating discovery, and where MDM truth belongs. *(planned)*

## Who this is for

ServiceNow architects and admins, observability and platform engineers, and CMDB / CSDM governance owners wiring a third-party platform into ServiceNow.

## Topics

`servicenow` · `cmdb` · `csdm` · `itsm` · `integration-patterns` · `observability` · `dynatrace` · `intune` · `service-graph-connector` · `ire`

## Author

Written by Victor Andreev — ITSM / observability architect. Concept and direction by the author, developed in collaboration with Claude (Anthropic).

## License

Content is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — share and adapt with attribution.

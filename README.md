# Business Process Diagrams

> Diagrams from a real process optimisation project at a telecommunications company.
> The goal was to reduce the number of steps in the online purchase journey for bundled products (handset + tariff + insurance).

---

## Context

The e-commerce team identified that the multi-step purchase funnel was causing significant drop-off — customers were abandoning the flow between product selection and contract signing.

As part of the analysis I:

- Mapped the **AS-IS process** end-to-end across all touchpoints (web, CRM, BSS/billing, credit bureau, insurance partner)
- Identified bottlenecks: manual credit verification (1–2 days), paper contract signing (branch visit required), manual SIM activation (24–48 hours)
- Proposed and documented the **TO-BE process** reducing the journey from 8 steps to 5, with full automation
- Delivered BPMN diagrams, sequence diagrams, state model and data model as input for the development team

**Result:** End-to-end purchase time reduced from 2–5 days to approximately 15 minutes.

---

## Diagrams

### [telecom-purchase-flow.md](telecom-purchase-flow.md)

Full purchase scenario: smartphone on instalment + tariff plan + device insurance.

| Diagram          | What it shows                                                                                   |
| ---------------- | ----------------------------------------------------------------------------------------------- |
| BPMN Flow        | Business process with lanes — customer, system, external systems                                |
| Sequence Diagram | System-to-system communication — WebShop, CRM, BSS, Credit Bureau, Insurance API, Notifications |
| State Diagram    | Order lifecycle — from DRAFT through ACTIVE to COMPLETED/CANCELLED                              |
| ERD              | Data model — Customer, Order, Product, Tariff, Instalment Plan, Insurance Policy, SIM Card      |
| AS-IS vs TO-BE   | Step-by-step comparison of original vs optimised flow                                           |

---

## Tools used

- **draw.io** — initial whiteboard sessions with stakeholders
- **Confluence** — final documentation and stakeholder review
- **Jira** — user stories and acceptance criteria linked to each process step
- **Mermaid** — diagrams as code for version control

# IT Analyst Portfolio

Process analysis and system-integration work samples from real engagements —
telco e-commerce and multinational logistics. Includes pragmatic
**AI opportunity assessment**: where AI is the right fix, and where a simpler
solution wins.

[![Mermaid](https://img.shields.io/badge/Mermaid-diagrams%20as%20code-FF3E00)](https://mermaid.js.org/)
[![BPMN](https://img.shields.io/badge/BPMN-AS--IS%20%2F%20TO--BE-3178c6)](https://www.bpmn.org/)
[![Postman](https://img.shields.io/badge/Postman-API%20mapping-FF6C37?logo=postman&logoColor=white)](https://www.postman.com/)

Diagrams are written as code (Mermaid) so they version-control cleanly
alongside the analysis.

---

## Projects

### [Telekom — Online Purchase Flow](telecom/)

Mapped and optimised the e-commerce purchase journey for a bundled offer
(smartphone + tariff + device insurance).

The original flow had too many steps and a high drop-off rate between product
selection and order confirmation. Walked through the full AS-IS process,
identified the bottlenecks, and proposed a TO-BE flow that reduced the
customer journey from 8 steps to 5, with backend automation handling the rest.

**Demonstrates**: AS-IS / TO-BE process mapping · BPMN · API-level integration
analysis · funnel conversion thinking.

**Deliverables:**

| File | Contents |
|---|---|
| [purchase-flow.md](telecom/purchase-flow.md) | Process flow, sequence diagram, state diagram, ERD, AS-IS vs TO-BE |
| [user-stories.md](telecom/user-stories.md) | 6 user stories with acceptance criteria and Definition of Done |
| [postman collection](telecom/postman/telecom-purchase-flow.postman_collection.json) | All API integration points with example request / response pairs |

### [Logistics — Order-to-Delivery & AI Opportunity Assessment](logistics/)

End-to-end process analysis for a multinational logistics operation spanning
EU, US, and APAC. Mapped the full order-fulfilment chain from order placement
through warehouse, transport, and last-mile delivery.

Identified 5 operational bottlenecks through process walk-throughs, system
data analysis, and support ticket categorisation. For each bottleneck,
assessed whether AI is the right fix — **only 2 of the 5 recommend AI**. The
rest are solved by process fixes, data integration, and hardware automation.

Includes change-management notes per team — different stakeholders have
different fears and different adoption paths.

**Demonstrates**: end-to-end process mapping · bottleneck triage · pragmatic
AI opportunity assessment · change-management thinking.

**Deliverables:**

| File | Contents |
|---|---|
| [order-to-delivery.md](logistics/order-to-delivery.md) | Process flow, sequence diagram, state diagram, bottleneck analysis, AI assessment, AS-IS vs TO-BE, change management |
| [user-stories.md](logistics/user-stories.md) | 6 user stories with acceptance criteria and Definition of Done |
| [analysis-rationale.md](logistics/analysis-rationale.md) | Reasoning behind every decision — why AI for some, why not for others |

---

## Approach

Each project follows the same loop:

1. **Walk the flow** — observe AS-IS process, interview stakeholders
2. **Identify bottlenecks** — combine data + tickets + qualitative signals
3. **Assess AI fit** — for each bottleneck, ask whether AI is the right fix
   or a simpler solution wins
4. **Design TO-BE** — with explicit change-management notes per team
5. **Document deliverables** — diagrams-as-code (Mermaid), API mapping
   (Postman), user stories with acceptance criteria

Deliverables stay alongside the analysis so diagrams remain diff-able as the
process evolves.

---

## Tools

Mermaid · Postman · draw.io · Confluence · Jira · SQL · REST / SOAP

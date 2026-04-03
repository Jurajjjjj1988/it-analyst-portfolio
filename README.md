# IT Analyst Portfolio

Work samples from process analysis and system integration projects.
Diagrams are written as code (Mermaid) so they version-control cleanly alongside documentation.

---

## Projects

### [Telekom — Online Purchase Flow](telecom/)

Mapped and optimised the e-commerce purchase journey for a bundled offer (smartphone + tariff + device insurance).

The original flow had too many steps and a high drop-off rate between product selection and order confirmation. I walked through the full AS-IS process, identified the bottlenecks, and proposed a TO-BE flow that reduced the journey from 8 customer-facing steps to 5 with full backend automation.

**Deliverables:**

| File                                                                                | Contents                                                           |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| [purchase-flow.md](telecom/purchase-flow.md)                                        | Process flow, sequence diagram, state diagram, ERD, AS-IS vs TO-BE |
| [user-stories.md](telecom/user-stories.md)                                          | 6 user stories with acceptance criteria and Definition of Done     |
| [postman collection](telecom/postman/telecom-purchase-flow.postman_collection.json) | All API integration points with example request/response pairs     |

### [Logistics — Order-to-Delivery Process & AI Opportunity Assessment](logistics/)

End-to-end process analysis for a multinational logistics operation spanning EU, US, and APAC. Mapped the full order fulfilment chain from order placement through warehouse, transport, and last-mile delivery.

Identified 5 operational bottlenecks through process walk-throughs, system data analysis, and support ticket categorisation. For each bottleneck, assessed whether AI is the right fix or whether a simpler solution exists — only 2 of the 5 recommend AI. The rest are solved by process fixes, data integration, and hardware automation.

Includes a change management perspective for each proposed solution — different teams have different fears and different adoption paths.

**Deliverables:**

| File                                                     | Contents                                                                                                                         |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| [order-to-delivery.md](logistics/order-to-delivery.md)   | Process flow, sequence diagram, state diagram, bottleneck analysis, AI opportunity assessment, AS-IS vs TO-BE, change management |
| [user-stories.md](logistics/user-stories.md)             | 6 user stories with acceptance criteria and Definition of Done                                                                   |
| [analysis-rationale.md](logistics/analysis-rationale.md) | Reasoning behind every decision — why AI for some, why not for others                                                            |

---

## Tools

Mermaid · Postman · draw.io · Confluence · Jira · SQL · REST/SOAP

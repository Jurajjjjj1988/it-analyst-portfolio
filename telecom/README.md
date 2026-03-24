# Telecom — Online Purchase Flow

Analysis of the e-commerce purchase journey for a bundled product offer: smartphone on instalment + monthly tariff + device insurance.

## Background

The purchase funnel had a high abandonment rate. Customers were dropping off between the product selection and the contract confirmation step. The root causes were a fragmented UI flow, a manual credit check that took 1–2 days, and a paper contract that required a branch visit.

## What I did

- Walked through the full AS-IS process across all touchpoints
- Identified 3 main bottlenecks (credit check, contract signing, SIM activation)
- Proposed TO-BE flow reducing steps from 8 to 5, with all three bottlenecks automated
- Documented system-to-system integration points (WebShop → CRM → BSS/Billing → Credit Bureau → Insurance API)
- Modelled the order data structure (ERD) and order lifecycle (state diagram)
- Wrote user stories with acceptance criteria for the development team
- Documented all API integration points as a Postman collection

## Result

End-to-end purchase time: from 2–5 days down to ~15 minutes.

## Deliverables

| File                                                                                                           | Contents                                                                            |
| -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| [purchase-flow.md](purchase-flow.md)                                                                           | Process flow, sequence diagram, state diagram, ERD, AS-IS vs TO-BE                  |
| [user-stories.md](user-stories.md)                                                                             | 6 user stories with acceptance criteria and Definition of Done                      |
| [postman/telecom-purchase-flow.postman_collection.json](postman/telecom-purchase-flow.postman_collection.json) | Postman collection — all API integration points with example request/response pairs |

# Logistics — Order-to-Delivery Process & AI Opportunity Assessment

Process analysis for a multinational logistics company. Maps the full order fulfilment chain and identifies where AI adds real value vs where simpler solutions work better.

## Deliverables

| File                                           | What's inside                                                                                                                                             |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [order-to-delivery.md](order-to-delivery.md)   | Process flow (Mermaid), sequence diagram, state diagram, bottleneck analysis, AI assessment with prioritisation matrix, AS-IS vs TO-BE, change management |
| [user-stories.md](user-stories.md)             | 6 user stories — carrier selection AI, customs LLM, proactive notifications, chatbot, inventory dashboard, automated dimensioning                         |
| [analysis-rationale.md](analysis-rationale.md) | Why each decision was made — methodology, when AI fits, when it doesn't, change management reasoning                                                      |

## Key Findings

5 bottlenecks identified. Only 2 need AI:

| Bottleneck                          | Solution                                      | Type             |
| ----------------------------------- | --------------------------------------------- | ---------------- |
| Manual carrier selection            | ML recommendation model                       | AI               |
| Customs documentation               | LLM document generation                       | AI               |
| Status inquiry overload             | Proactive notifications first, chatbot second | Process fix + AI |
| No cross-warehouse stock visibility | Data integration dashboard                    | Infrastructure   |
| Invoice weight/dims errors          | Automated camera + scale                      | Hardware         |

## Approach

Not everything is an AI problem. The analysis deliberately includes solutions where AI is not the answer — to show that good analysis means picking the right tool, not the most impressive one.

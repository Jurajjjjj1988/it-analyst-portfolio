# User Stories — Order-to-Delivery Optimisation

Epic: **Reduce order fulfilment time and manual effort across global logistics operations by introducing AI-assisted automation at key bottleneck points.**

---

## US-01 — AI-assisted carrier selection

**As a** logistics dispatcher processing outbound shipments,
**I want to** receive an AI-recommended carrier for each shipment based on destination, weight, SLA and cost,
**so that** I can make faster, data-driven decisions instead of relying on memory and habit.

**Acceptance Criteria:**

- [ ] System presents top 3 carrier recommendations within 5 seconds of shipment being marked as ready
- [ ] Each recommendation includes: carrier name, estimated cost, estimated delivery date, and a confidence score
- [ ] Dispatcher can accept the top recommendation with one click or select an alternative
- [ ] If dispatcher overrides the recommendation, the system logs the override reason for model improvement
- [ ] Historical shipment performance data (on-time rate, damage rate, cost) is visible per carrier
- [ ] Model retrains weekly on new shipment outcome data

**Definition of Done:**

- [ ] AI recommendation engine deployed to staging and tested with 500 historical shipments
- [ ] Override logging and reason capture implemented
- [ ] Dispatcher training session conducted with at least 80% attendance
- [ ] A/B test plan defined: 2 weeks AI-assisted vs manual, measure cost and on-time rate
- [ ] Product owner sign-off after A/B results

---

## US-02 — Automated customs documentation

**As a** customs documentation specialist handling international shipments,
**I want** the system to auto-generate customs forms (commercial invoice, packing list, HS code classification) from order data,
**so that** I spend my time reviewing and validating instead of copying data between systems.

**Acceptance Criteria:**

- [ ] System generates customs documents within 30 seconds of shipment being marked international
- [ ] Commercial invoice includes all required fields per destination country regulations
- [ ] HS codes are auto-suggested based on product description and category — specialist confirms or corrects
- [ ] Packing list reflects actual picked items, weights, and dimensions from WMS
- [ ] Generated documents pass automated field validation before reaching the specialist
- [ ] Validation flags missing or inconsistent fields with clear error messages
- [ ] Specialist can edit any field before final approval

**Definition of Done:**

- [ ] LLM document generation tested against 100 historical international shipments across 5 destination countries
- [ ] HS code accuracy measured — target: >90% match with specialist-assigned codes
- [ ] Border rejection rate tracked: baseline vs AI-generated (target: 50% reduction in first quarter)
- [ ] Rollback procedure documented — if AI generation fails, manual process is still available
- [ ] Product owner sign-off

---

## US-03 — Proactive shipment notifications

**As a** customer who placed an order,
**I want to** receive status updates at each milestone (confirmed, shipped, out for delivery, delivered),
**so that** I don't need to email customer service to find out where my order is.

**Acceptance Criteria:**

- [ ] Customer receives email and SMS at each status change: order confirmed, shipped, in transit, out for delivery, delivered
- [ ] Each notification includes current status, estimated delivery date, and tracking link
- [ ] Tracking page shows real-time location and status from carrier API
- [ ] Customer can reply to the notification email — replies are routed to customer service, not to a dead mailbox
- [ ] Notifications respect customer preferences (email only, SMS only, both)
- [ ] System handles carrier API downtime gracefully — if tracking data is unavailable, notification says "status update temporarily unavailable" instead of failing silently

**Definition of Done:**

- [ ] Notification service integrated with TMS and at least 3 carrier tracking APIs
- [ ] Email and SMS templates reviewed by marketing for brand consistency
- [ ] Load test: system can handle 10,000 notifications per hour without degradation
- [ ] Customer service email volume measured: baseline vs 4 weeks after launch (target: 40% reduction)
- [ ] Product owner sign-off

---

## US-04 — Customer service chatbot

**As a** customer with a delivery question,
**I want to** ask a chatbot about my order status in plain language and get a specific, accurate answer,
**so that** I get immediate help without waiting for a human agent.

**Acceptance Criteria:**

- [ ] Chatbot accessible via website widget and customer portal
- [ ] Chatbot understands order-related queries: "where is my order?", "when will it arrive?", "my delivery failed"
- [ ] Chatbot pulls real-time tracking data from TMS API and responds with specific order information
- [ ] If chatbot cannot answer or customer requests it, seamless handoff to human agent with full conversation history
- [ ] Chatbot does not hallucinate delivery dates — if data is unavailable, it says "I don't have that information right now, let me connect you with an agent"
- [ ] Response time under 3 seconds
- [ ] Available in English, German, and local language per region

**Definition of Done:**

- [ ] Chatbot tested with 200 sample queries across 3 languages — accuracy >85%
- [ ] Handoff to human agent tested and working — agent sees full chatbot conversation
- [ ] Fallback path tested — chatbot unavailable → customer sees "chat is temporarily offline, please email us"
- [ ] Agent feedback mechanism — agents can flag incorrect chatbot responses for improvement
- [ ] Product owner sign-off

---

## US-05 — Consolidated inventory dashboard

**As a** supply chain planner responsible for stock levels across all warehouses,
**I want to** see real-time inventory across all regions in a single dashboard,
**so that** I can make informed replenishment decisions and prevent stockouts.

**Acceptance Criteria:**

- [ ] Dashboard shows current stock per SKU per warehouse, updated at least every 15 minutes
- [ ] Colour-coded alerts: green (above reorder point), amber (approaching reorder), red (below safety stock)
- [ ] Drill-down from dashboard to individual warehouse WMS data
- [ ] Historical stock level trend per SKU (last 90 days) visible on hover or click
- [ ] Export to CSV for offline analysis
- [ ] Dashboard loads within 3 seconds even with 50,000+ SKUs

**Definition of Done:**

- [ ] Data integration from all WMS instances verified — no warehouse missing
- [ ] Alert thresholds configurable per SKU per warehouse
- [ ] Dashboard tested with 3 supply chain planners — feedback incorporated
- [ ] Data freshness SLA documented (15 min) and monitored
- [ ] Product owner sign-off

---

## US-06 — Automated weight and dimension capture

**As a** packing station operator,
**I want** the system to automatically capture package weight and dimensions when I place it on the station,
**so that** I don't need to type numbers manually and risk transcription errors on invoices.

**Acceptance Criteria:**

- [ ] Scale captures weight automatically when package is placed (tolerance: +/- 50g)
- [ ] Camera captures dimensions automatically (tolerance: +/- 1cm)
- [ ] Captured data linked to correct order via barcode scan
- [ ] Operator sees captured values on screen and can override if clearly wrong
- [ ] Data flows directly to WMS and ERP — no manual re-entry
- [ ] If hardware fails (scale offline, camera error), operator can enter values manually as fallback

**Definition of Done:**

- [ ] Hardware installed and calibrated at one packing station (pilot)
- [ ] 500 packages measured — accuracy verified against manual measurement
- [ ] Invoice discrepancy rate measured: baseline 12% vs target <2% after automation
- [ ] Operator training completed
- [ ] Rollout plan for remaining packing stations documented
- [ ] Product owner sign-off

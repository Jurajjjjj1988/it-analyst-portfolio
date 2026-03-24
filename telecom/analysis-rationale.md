# Analysis Rationale — Telecom Purchase Flow

> This document explains the reasoning behind every decision made during the process analysis and optimisation project. It covers the theoretical foundations, methodology, and a walkthrough of each deliverable — written so the analysis can be explained and defended in a technical discussion.

---

## Table of Contents

1. [Methodology — AS-IS / TO-BE Analysis](#1-methodology--as-is--to-be-analysis)
2. [Why a Bundle Page — UX and Conversion Funnel Theory](#2-why-a-bundle-page--ux-and-conversion-funnel-theory)
3. [Why Real-Time Credit Check — API Integration Patterns and SLA](#3-why-real-time-credit-check--api-integration-patterns-and-sla)
4. [Why OTP E-Signature — eIDAS Regulation and Legal Validity](#4-why-otp-e-signature--eidas-regulation-and-legal-validity)
5. [Why Atomic Activation — Eventual Consistency and the Saga Pattern](#5-why-atomic-activation--eventual-consistency-and-the-saga-pattern)
6. [Diagram Walkthrough — What Each Diagram Shows and Why](#6-diagram-walkthrough--what-each-diagram-shows-and-why)
7. [Postman Collection — What It Demonstrates](#7-postman-collection--what-it-demonstrates)
8. [User Stories — Structure and Purpose](#8-user-stories--structure-and-purpose)

---

## 1. Methodology — AS-IS / TO-BE Analysis

### What it is

AS-IS / TO-BE is a structured process improvement methodology. You first document the current state (AS-IS) exactly as it works today — including all inefficiencies — and then design the desired future state (TO-BE). The gap between the two defines the scope of the project.

### Why this order matters

It is tempting to jump straight to the solution. But without documenting AS-IS first, you risk:

- Missing steps that look invisible (e.g. a manual approval that happens over email)
- Designing a TO-BE that solves the wrong problem
- Having no baseline to measure improvement against

### How AS-IS was built here

The AS-IS was reconstructed by walking through the live purchase flow on telekom.sk step by step, observing each screen, modal, and system interaction. This produced an 8-step process. The key insight was that steps 1, 2, and 3 (phone / tariff / insurance) were three separate pages with no persistent state — if you went back, your selection was lost.

### How the bottlenecks were identified

A bottleneck is a step that disproportionately increases total process time or causes drop-off. Three were identified:

| Bottleneck            | Type                               | Impact                                       |
| --------------------- | ---------------------------------- | -------------------------------------------- |
| Manual credit check   | Waiting time (1–2 days)            | Customers abandon or forget                  |
| Paper contract        | Physical dependency (branch visit) | Converts online funnel into offline          |
| Manual SIM activation | Waiting time (24–48 hours)         | No instant gratification, high support calls |

### How TO-BE was designed

Each bottleneck was addressed with an automation or UX change that already exists as a proven technology:

- Bundle page → standard e-commerce pattern
- Real-time credit API → used by banks and telcos across Europe
- OTP e-signature → legally valid under eIDAS
- API-triggered activation → standard BSS integration

The TO-BE does not invent anything. It connects technologies that exist. That is the job of the analyst — to identify what is possible, not to build it.

One important caveat: physical SIM delivery still requires a courier, which takes 1–2 business days. The end-to-end time reduction to "under 1 hour" applies specifically to eSIM customers. Honest analysis means stating this explicitly rather than claiming an improvement that only holds for part of the customer base.

---

## 2. Why a Bundle Page — UX and Conversion Funnel Theory

### Conversion funnel

A conversion funnel is the sequence of steps a user must complete to reach a goal (purchase). Every step is a potential exit point. The drop-off rate at each step compounds — if 20% of users leave at each of 3 steps, only 51% reach the end. Reduce those 3 steps to 1 and you eliminate two exit points entirely.

This is why the number of steps directly affects conversion rate. Research by the Baymard Institute shows that the average e-commerce checkout abandonment rate is ~70%, and unnecessary steps are the second most common reason cited by users.

### Cognitive load

Each page transition increases cognitive load — the user must re-orient, re-read, re-confirm. When phone, tariff, and insurance are on separate pages, the user cannot easily compare combinations. When they are on one page, the relationship between products is visible and the decision is made once.

### The bundle as an upsell mechanism

A secondary benefit: presenting insurance at the same time as the phone, before the user is in "checkout mode", increases attach rate. This is a well-documented retail psychology effect — the user is already committed to a purchase and the insurance appears as a natural extension, not an add-on.

### What changed in the flow

In the AS-IS, the user made three separate decisions on three pages. In the TO-BE, they make one combined decision on one page. The cart contains all three items. This is modelled in the flowchart as a single step before "Add to cart" and reflected in the ERD where ORDER_ITEM can hold items of type DEVICE, TARIFF, and INSURANCE.

---

## 3. Why Real-Time Credit Check — API Integration Patterns and SLA

### The problem with manual credit check

In the AS-IS, the credit check was performed manually by a back-office employee who contacted the credit bureau, received a response, and updated the order. This took 1–2 business days. During this time, the order was in a pending state and the customer had no contract, no SIM, and no certainty.

### What an API integration replaces

A credit bureau API (e.g. CRIF, SCHUFA, Experian) exposes an endpoint that accepts a national ID and requested amount and returns a credit decision in seconds. The same information that used to travel via email or phone now travels via an HTTP request.

This is the core value of API integration: replacing a human-mediated information exchange with a machine-to-machine call. The analyst's job is to identify where these human-mediated exchanges exist and whether an API alternative is available.

### SLA — Service Level Agreement

When integrating with an external credit bureau, an SLA must be agreed. For this flow, the requirement is:

- Response time: < 5 seconds (defined in US-02)
- Availability: 99.9% during business hours
- Fallback: if the bureau is unavailable, the system either queues the check or offers the customer a one-time payment alternative

The 5-second figure is not arbitrary — it is the threshold beyond which users perceive a response as slow and begin to abandon the page (research by Google shows 53% of mobile users abandon a page that takes longer than 3 seconds to load).

### Integration pattern: synchronous request-response

The credit check uses a synchronous pattern — the WebShop calls the bureau, waits for a response, and shows the result before the user can proceed. This is the correct pattern here because the result is needed immediately to determine which products to offer.

An asynchronous pattern (fire and forget, then poll or receive a callback) would be appropriate if the check took minutes — but real-time bureau APIs make synchronous the better choice.

### Where this is modelled

In the sequence diagram, the credit check appears as:

```
WebShop ->> CreditCheck: Credit score request
CreditCheck -->> WebShop: Score 720 / APPROVED
```

The double arrow (`-->>`) denotes a return message. The `Note over CreditCheck` documents the external dependency. In the ERD, `credit_score` is stored on the CUSTOMER entity so subsequent orders for the same customer can reuse a recent score without re-calling the bureau (with appropriate TTL policy).

---

## 4. Why OTP E-Signature — eIDAS Regulation and Legal Validity

### The legal problem

A purchase contract for a device on instalment is a legally binding credit agreement. Under Slovak and EU law, such agreements require the customer's informed, verifiable consent. Historically this meant a wet ink signature on paper.

### eIDAS — the regulatory foundation

eIDAS (Electronic Identification, Authentication and Trust Services) is an EU regulation (910/2014) that establishes a legal framework for electronic signatures. It defines three levels:

| Level                          | How                                                            | Legal weight                       |
| ------------------------------ | -------------------------------------------------------------- | ---------------------------------- |
| Simple electronic signature    | Any digital action — e.g. checkbox                             | Lowest — can be disputed           |
| Advanced electronic signature  | Uniquely linked to signatory, capable of detecting changes     | High — suitable for most contracts |
| Qualified electronic signature | Requires a qualified certificate from a trust service provider | Equivalent to wet ink              |

An OTP sent to a verified mobile number constitutes an **advanced electronic signature** under eIDAS. It is uniquely linked to the signatory (they own the number), it is sent at a specific time for a specific document, and the event is logged with a timestamp and document hash. This is legally sufficient for a consumer credit contract in Slovakia and across the EU.

### Why not a qualified signature

A qualified signature requires a hardware token or a visit to a registration authority. This reintroduces the friction we are trying to remove. The legal risk of an advanced signature is acceptable for this contract type and value range.

### What the analyst documents

The analyst does not design the cryptographic implementation. The analyst documents:

1. That e-signature is legally valid for this contract type (references eIDAS)
2. That the OTP must be linked to the specific document version (not reusable)
3. That the event must be logged (timestamp, document hash, MSISDN) for audit

This is reflected in US-03 acceptance criteria and in the sequence diagram where `CONTRACT_SIGNED` is a state transition in both the sequence diagram and the state diagram.

---

## 5. Why Atomic Activation — Eventual Consistency and the Saga Pattern

### The problem

After payment succeeds, three separate systems must be updated:

1. BSS — activate SIM and tariff
2. BSS — create instalment plan
3. Insurance API — activate policy

These are three separate API calls to two separate systems. What happens if the first two succeed but the third fails? The customer has an active SIM and tariff but no insurance. They paid for insurance. This is a partial activation — a data inconsistency.

### ACID vs eventual consistency

In a single relational database, you would use a transaction — either all operations succeed (COMMIT) or all are rolled back (ROLLBACK). This is ACID (Atomicity, Consistency, Isolation, Durability).

Across distributed systems (separate APIs), ACID transactions are not possible. You cannot wrap calls to BSS and an external insurance API in a database transaction. This is the core challenge of distributed systems.

### The Saga pattern

The Saga pattern is the standard solution. A saga is a sequence of local transactions where each step publishes an event or calls the next step. If a step fails, compensating transactions are executed to undo the completed steps.

For this flow:

```
Step 1: Activate SIM + tariff (BSS)
  → if fails: cancel order, refund payment
Step 2: Create instalment plan (BSS)
  → if fails: deactivate SIM + tariff, refund payment
Step 3: Activate insurance policy (Insurance API)
  → if fails: cancel instalment plan, deactivate SIM + tariff, refund payment
Step 4: Send confirmation (Notification)
```

This is a choreography-based saga — each step triggers the next. An alternative is an orchestration-based saga where a central coordinator (the WebShop or an orchestration service) manages the sequence.

### Why the analyst documents this

The analyst does not implement the saga. But the analyst must:

1. Identify that partial activation is a risk
2. Define the expected behaviour on failure (defined in US-06: rollback all steps)
3. Define the success condition (all three confirmed active before notification is sent)
4. Document the sequence so the development team knows the order and dependencies

This is exactly what the sequence diagram shows — the activation steps are in the correct dependency order, and US-06 acceptance criteria explicitly state the rollback requirement.

### Eventual consistency

Even with a saga, there is a window between step 1 completing and step 3 completing where the system is in an intermediate state. This is called eventual consistency — the system will reach a consistent state, but not instantaneously. The analyst's job is to define what that window is (in this case, < 60 seconds per US-06) and what the user sees during it (a "processing" state, not a false "active" state).

---

## 6. Diagram Walkthrough — What Each Diagram Shows and Why

### 6.1 Process Flow (Flowchart)

**What it shows:** The customer journey end-to-end, including decision points and system touchpoints.

**Why this format:** A flowchart is the right tool for a linear process with branching decisions. It is readable by both business stakeholders and developers. Mermaid's `flowchart TD` renders directly in GitHub, making it accessible without specialist tools.

**Key decisions in the diagram:**

- `{Existing customer?}` — this branch determines whether the customer logs in or goes through new customer registration. It appears early because it affects which discounts are applied.
- `{Port existing number or new?}` — separate branch because number portability involves an external MNP process not required for new numbers.
- `{Score OK?}` — the credit gate. If rejected, the flow ends (or can offer one-time payment). This is a hard stop that must be explicit.
- The `EXTERNAL SYSTEMS` subgraph groups backend dependencies visually — the customer never sees these, but they are part of the flow.

**What the cylinder shapes mean:** `[(Credit Bureau)]` uses Mermaid's database/cylinder shape to indicate a data store or external system, distinguishing it from process steps (rectangles) and decisions (diamonds).

### 6.2 Sequence Diagram

**What it shows:** System-to-system communication — which system calls which, in what order, with what data.

**Why this format:** A sequence diagram is the standard tool for documenting API integrations. It makes the direction of calls explicit (who is the client, who is the server), shows synchronous vs asynchronous communication, and reveals the data that must be passed at each step.

**Key decisions in the diagram:**

- Participants are named by system role (WebShop, CRM, BSS) not by vendor — this keeps the diagram vendor-neutral and valid even if the underlying technology changes.
- `Note over CreditCheck` documents context (external bureau, e.g. CRIF) without cluttering the flow.
- The return arrows (`-->>`) are dashed to distinguish responses from requests, following UML convention.
- The activation sequence (BSS → Insurance → Notification) appears at the end, after payment — documenting the dependency: activation only starts on PAYMENT_SUCCESS.

**What it is used for:** Development teams use this as the specification for integration work. Each arrow becomes an API call to be implemented or consumed.

### 6.3 State Diagram

**What it shows:** All possible states of an ORDER entity and the events that trigger transitions between them.

**Why this format:** A state diagram is the correct tool for modelling an entity with a lifecycle. An order is not static — it moves through states from DRAFT to ACTIVE to COMPLETED (or CANCELLED). The state diagram makes all valid transitions explicit and prevents invalid ones (e.g. an order cannot go from DRAFT directly to PAID).

**Key states explained:**

- `PENDING_CREDIT` — order is waiting for credit check result. This is a real state that must exist in the database so the system knows not to proceed with contract generation until the check completes.
- `PENDING_SIGNATURE` — contract has been generated and sent to the customer. System waits. This state can have a timeout (e.g. 24 hours) after which the order expires.
- `ACTIVATING` — payment succeeded, activation is in progress. This intermediate state prevents the UI from showing ACTIVE before all three systems have confirmed.
- `SUSPENDED` — active contract where a monthly payment was missed. The customer can recover (pay) or the contract is eventually cancelled. This models real business rules around debt management.

**What the analyst delivers:** The state diagram becomes the source of truth for the `status` field in the ORDER table. Every value in that enum must be in the diagram. Every transition must have a defined trigger.

### 6.4 ERD — Entity-Relationship Diagram

**What it shows:** The data model — what entities exist, what attributes they have, and how they relate.

**Why this format:** An ERD is the standard artefact that bridges business analysis and database design. The analyst defines entities and relationships at a logical level; the developer translates this into a physical schema.

**Key design decisions:**

- `ORDER_ITEM` as a separate entity — an order can contain multiple products (device + tariff + insurance). Putting all three on the ORDER entity directly would violate normalisation (First Normal Form). ORDER_ITEM allows any number of products per order.
- `INSTALLMENT_PLAN` separate from ORDER — the instalment plan has its own lifecycle (paid_months, status) and could theoretically be transferred or modified independently of the original order. Keeping it separate respects single responsibility.
- `PAYMENT` separate from ORDER — an order can have multiple payment attempts (PAYMENT_FAILED followed by a retry). If payment were a field on ORDER, only one attempt could be recorded. A separate PAYMENT entity captures the full payment history.
- `SIM_CARD` linked to CUSTOMER, not ORDER — a SIM card outlives the purchase. The customer may change their tariff plan in the future (which updates the SIM's tariff_plan_id) without creating a new order. Linking SIM to CUSTOMER models the ongoing relationship.

**Cardinality notation:**

- `||--o{` : one (exactly one) to zero or many
- `||--|{` : one to one or many (at least one)
- `}o--||` : zero or many to one

### 6.5 AS-IS vs TO-BE

**What it shows:** A structured comparison of the old process and the new process, with explicit mapping of what was removed and why.

**Why this format:** Decision-makers and stakeholders need to see the improvement in concrete terms — not just "we made it better" but "we removed these specific steps by doing these specific things, and here is the measurable impact."

**The table structure:**

- Removed step — what existed in AS-IS
- How — the specific mechanism that replaced it
- Impact — the quantified or qualifiable benefit

This structure forces the analyst to be specific. "We automated the credit check" is not enough. The table requires: what was it before, what replaced it, what changed.

---

## 7. Postman Collection — What It Demonstrates

### Purpose

The Postman collection documents all API integration points from the sequence diagram as executable requests with example request bodies and response pairs.

### Why this matters for an analyst

An IT analyst who can produce a Postman collection demonstrates:

1. Understanding of REST API structure (methods, headers, request bodies, status codes)
2. Ability to specify what data must be exchanged at each integration point
3. A concrete artefact that developers can use as a starting point or that testers can use to validate behaviour

### Structure of the collection

Each request follows a consistent pattern:

- Named by step number and system (e.g. `5. BSS — Process Payment`)
- `description` field explains when this call is triggered and what it produces
- Request body contains realistic field names and values
- Responses include both the success case and at least one error case

This mirrors how a real API specification is written — both paths (happy and error) must be documented so the development team knows what to handle.

### Variables

The collection uses Postman variables (`{{baseUrl}}`, `{{customerId}}`, `{{orderId}}`) instead of hardcoded values. This allows the same collection to run against different environments (dev, staging, production) by switching the environment file. Sensitive values (`{{creditBureauApiKey}}`, `{{insuranceToken}}`) use variables to signal that these must be injected from a secrets manager, never hardcoded.

### What the collection is not

The collection uses a fictional domain (`api.telekom.example.com`). It cannot be executed against a real system. This is intentional — it documents the integration architecture, not a live implementation. The analyst's deliverable is the specification; the developer's deliverable is the working endpoint.

---

## 8. User Stories — Structure and Purpose

### Format

Each user story follows the standard Agile format:

> **As a** [role], **I want to** [goal], **so that** [benefit].

The "so that" clause is the most important part and the most commonly omitted. It forces the analyst to articulate _why_ the feature is needed, not just what it does. This allows the development team to make implementation decisions that serve the actual need.

### Acceptance Criteria

Acceptance criteria are written as checkboxes in Given/When/Then spirit — a condition that can be verified as true or false after implementation. Vague criteria ("the page should work correctly") are not useful. Specific criteria ("credit check completes within 5 seconds") can be tested.

Good acceptance criteria serve three audiences:

1. **Developer** — knows exactly what to build
2. **Tester** — knows exactly what to verify
3. **Product owner** — knows what to sign off

### US-06 — the technical user story

US-06 is a system-level story ("As a system..."). This is a valid pattern when the behaviour being specified is internal and not directly triggered by a user action. The atomic activation requirement is a non-functional requirement that cuts across multiple systems — a user story scoped to a single UI action cannot capture it. A system-level story makes it a first-class deliverable with explicit acceptance criteria rather than an implementation detail that might be missed.

### Definition of Done

The Definition of Done (DoD) applies to all stories in the epic. It includes requirements that go beyond acceptance criteria — E2E test coverage, API documentation, state diagram updates, product owner sign-off. The DoD ensures that "done" means the same thing to everyone on the team.

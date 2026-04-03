# Analysis Rationale — Global Order-to-Delivery

> Explains the reasoning behind every decision in the logistics process analysis. Written so the work can be explained and defended in a technical or business discussion.

---

## Table of Contents

1. [Methodology — How the Bottlenecks Were Found](#1-methodology--how-the-bottlenecks-were-found)
2. [Why AI Is Not Always the Answer](#2-why-ai-is-not-always-the-answer)
3. [Carrier Selection — Why ML Over Rules](#3-carrier-selection--why-ml-over-rules)
4. [Customs Documentation — Why LLM Over Templates](#4-customs-documentation--why-llm-over-templates)
5. [Customer Inquiries — Why Fix the Process Before Adding AI](#5-customer-inquiries--why-fix-the-process-before-adding-ai)
6. [Stock Visibility — Why This Is Not an AI Problem](#6-stock-visibility--why-this-is-not-an-ai-problem)
7. [Invoice Accuracy — Why Hardware Beats Software Here](#7-invoice-accuracy--why-hardware-beats-software-here)
8. [Change Management — Why It Matters More Than the Technology](#8-change-management--why-it-matters-more-than-the-technology)
9. [Prioritisation — Why Customs Docs First](#9-prioritisation--why-customs-docs-first)

---

## 1. Methodology — How the Bottlenecks Were Found

### Data sources

Bottlenecks were identified through three inputs, not one. Relying on a single source introduces bias.

1. **Process walk-throughs with operations staff** — sitting with dispatchers, customs specialists, warehouse managers and watching them work. People describe their processes differently from how they actually perform them. Observation catches what interviews miss.

2. **System data analysis** — pulling timestamps from ERP and TMS to measure actual lead times between process steps. This showed where time was spent waiting (between steps) vs working (within steps). Waiting time is usually the bigger problem.

3. **Customer service ticket analysis** — categorising 4 weeks of support emails to find the most common complaint types. 62% were status inquiries. This is not a customer service problem — it is a notification problem. Different root cause, different solution.

### Why five bottlenecks and not ten

Every process has friction. Listing twenty issues and proposing twenty AI tools is not analysis — it is a wish list. The goal was to identify the bottlenecks with the highest business impact that have a realistic, deployable solution. Five is enough for a first wave. More will emerge as these are resolved — improvements reveal the next layer of problems.

### What was deliberately excluded

Several areas were out of scope for this analysis:

- Procurement and supplier management — a separate process with its own analysis needed
- Returns and reverse logistics — complex, deserves dedicated attention
- Warehouse layout and pick path optimisation — requires physical observation and warehouse-specific data that was not available

Scope discipline is part of the analysis. Trying to solve everything at once is how projects fail.

---

## 2. Why AI Is Not Always the Answer

This is the most important section of the rationale.

Of the five bottlenecks identified, only two have AI as the recommended solution. One uses ML (carrier selection) and one uses an LLM (customs documentation). The other three are solved by: a process fix (notifications), a data integration project (stock visibility), and hardware (weight capture).

An AI Evangelist who recommends AI for everything is not an evangelist — they are a salesperson. The value of someone who understands AI is not in deploying it everywhere, but in knowing where it fits and where simpler solutions work better.

The question for each bottleneck was: **What is the simplest solution that solves this problem?** If AI is simpler, use AI. If a database query is simpler, use that. If a camera on a scale is simpler, buy the camera.

---

## 3. Carrier Selection — Why ML Over Rules

### Why not a static rules engine?

A rules engine works when decisions are simple and stable: "If domestic, use PostNL. If international EU, use DHL." But this company ships to 40+ countries using 15+ carriers. Performance varies by route, season, weight class, and carrier capacity. A static rule for every combination is unmaintainable.

A machine learning model trained on historical shipment outcomes (cost, delivery time, damage rate, on-time performance) can learn patterns that no human would codify into rules. For example, Carrier X is 10% cheaper for Southeast Asia but has a 15% delay rate in monsoon season. A model learns this from data. A rules engine would need someone to manually encode it — and update it when it changes.

### Why not fully automated?

The dispatcher has context the model does not — a customer's last-minute call to change the delivery date, a known issue with a carrier this week, a relationship with a particular freight forwarder. The model recommends; the human decides. This is not a philosophical position — it is a practical one. Fully automated carrier selection requires much higher model accuracy and much more trust. That comes later, if at all.

### What training data is needed

The model needs historical shipment records with: origin, destination, carrier used, weight, dimensions, shipment cost, promised delivery date, actual delivery date, damage claims. Most logistics companies have this in their TMS and ERP — the challenge is getting it into a clean, unified dataset.

If the data is poor, the model will be poor. Data quality assessment is step zero, before any model work begins.

---

## 4. Customs Documentation — Why LLM Over Templates

### Why not just use document templates?

Templates work when the output is predictable. A domestic packing list is the same every time — a template is the right tool. But customs documentation varies by destination country: different required fields, different HS code classifications, different regulatory language. Maintaining templates for 40+ countries means 40+ templates that need updating whenever regulations change.

An LLM generates the document from order data plus country-specific rules. When regulations change, you update the rules — not 40 templates. The LLM also handles edge cases (unusual product descriptions, mixed shipments, special handling requirements) without template modifications.

### Why validation is non-negotiable

LLMs hallucinate. In customs documentation, a hallucinated HS code means goods stuck at the border, possible fines, and an angry customer. This is why AI-generated customs docs must pass automated field validation before a human ever sees them:

- Are all required fields for this destination country filled?
- Does the HS code exist and match the product category?
- Does the declared value match the order value in ERP?
- Is the total weight consistent with the shipment weight from WMS?

The validation catches errors that would otherwise reach the border. The specialist reviews the validated document — not raw AI output.

### HS code classification — the hard part

HS (Harmonised System) codes classify products for customs. There are over 5,000 six-digit codes. Getting it wrong can mean overpaying duty, underpaying duty (fraud risk), or shipment rejection. The LLM suggests a code based on product description. But the final classification must be confirmed by someone who understands the system.

Over time, as the system processes more shipments and specialists correct its suggestions, accuracy improves. But it should never be fully automated — the regulatory risk is too high.

---

## 5. Customer Inquiries — Why Fix the Process Before Adding AI

### The common mistake

The first instinct when customer service is overwhelmed with emails is: "Let's deploy a chatbot." But 62% of emails were asking the same question: "Where is my order?" This is not a question that needs AI to answer. It is a question that should not need to be asked.

If customers received proactive notifications at each milestone (shipped, in transit, out for delivery), most of those 800 emails per week would never be sent. The chatbot is the second step — it handles the remaining questions that notifications cannot answer (delayed shipments, damage claims, special requests).

### Sequence matters

Deploy notifications first. Measure the reduction in status inquiries. Then deploy the chatbot for the remaining volume. If you deploy the chatbot first, you are building an expensive tool to answer a question that a simple notification would prevent. The chatbot might still be needed, but for a much smaller and more complex set of queries — and that shapes its design.

### Why the chatbot must not hallucinate delivery dates

If the chatbot says "your order arrives tomorrow" and it does not, you have created a worse customer experience than having no chatbot at all. The chatbot should only state facts from the TMS API. If the carrier API does not have a delivery estimate, the chatbot says "I don't have an estimated delivery date right now" — not a guess.

This is a design principle, not a limitation. A chatbot that is honest about what it does not know builds more trust than one that confidently makes things up.

---

## 6. Stock Visibility — Why This Is Not an AI Problem

Each warehouse runs its own WMS. There is no unified view. The supply chain planner has to check 4 different systems to understand total stock levels. This is a data silo problem, not a pattern recognition problem.

The solution is a data integration layer — an ETL pipeline that pulls inventory from all WMS instances into a single data store, with a dashboard on top. Simple threshold alerts ("Frankfurt stock for SKU-4421 below reorder point") complete the picture.

Could you add AI later? Yes — demand forecasting using historical sales data + seasonal patterns + external signals (weather, events, promotions). But that requires clean, consolidated data as a foundation. The integration project enables future AI; it is not AI itself.

I included this in the analysis specifically because it demonstrates judgment. Recommending AI for a data integration problem would be intellectually dishonest and would erode trust with technical stakeholders who can see through it.

---

## 7. Invoice Accuracy — Why Hardware Beats Software Here

12% of invoices have incorrect weight or dimensions because these are captured manually at the packing station. A person reads a scale, writes a number on paper, and later types it into the system. The error is introduced between the physical measurement and the digital record.

No amount of AI can fix a transcription error after it happens. The solution is to prevent the error at the source: automated scale and camera at the packing station that feed data directly into WMS. No paper, no manual entry, no error.

Computer vision does play a role — reading the barcode on the package to match it to the correct order, and in some cases reading the label to verify it matches the destination. But the core fix is hardware automation, not language modelling.

---

## 8. Change Management — Why It Matters More Than the Technology

### The graveyard of good AI projects

Most AI projects that fail do not fail because the technology is wrong. They fail because nobody uses the tool after it is deployed. The company spends six months building something, rolls it out, and adoption sits at 15% after three months.

This is why the job description for an AI Evangelist puts change management at the centre — "toto je jádro vaší práce."

### Different teams, different fears

Each bottleneck fix affects a different team with different concerns:

**Dispatchers** fear being replaced. They see the AI carrier recommendation and think "they're automating my job." The truth is: we're automating the boring 80% so they can focus on the complex 20% where their expertise actually matters. But you have to demonstrate this, not just say it.

**Customs specialists** are more likely to welcome the change — they hate the copy-paste work. But they might worry about quality. Showing them that they become reviewers (higher-value work) rather than data entry clerks (lower-value work) reframes the change positively.

**Customer service agents** are complicated. The chatbot could reduce their team size, and they know it. Being honest — "the chatbot handles routine queries so you can focus on complex cases" — only works if it is actually true. If the company then reduces headcount, trust is permanently destroyed. The change management plan must be aligned with actual HR plans, not just talking points.

### Metrics as early warning

Adoption metrics are not something you check once after deployment. They are a live feed:

- If dispatcher override rate is above 40%, the model needs retraining or the dispatchers need more onboarding — investigate before assuming
- If chatbot handoff rate to humans is above 50%, the chatbot is not good enough — improve it, don't blame users
- If customs specialists stop using the AI tool after week 3, something went wrong — sit with them and find out what

The Evangelist's job is not to deploy and walk away. It is to deploy, watch, and adjust. Continuously.

---

## 9. Prioritisation — Why Customs Docs First

### The temptation to start big

Carrier selection ML sounds impressive. A chatbot is visible and exciting. But the first project must succeed, and success is measured by adoption + measurable impact + minimal resistance.

Customs documentation LLM wins on all three:

1. **Adoption:** The customs team actively dislikes the manual work. They want a solution. No resistance to overcome.
2. **Measurable impact:** Time per shipment drops from 35 min to 7 min. Border rejection rate drops. Both are measurable from day one.
3. **Minimal resistance:** The specialist's role changes from "fill forms" to "validate and approve." This is a promotion in everything but title.

Starting here creates a reference case: "We deployed AI in customs, saved X hours per month, reduced errors by Y%. Now let's talk about carrier selection." The success sells the next project.

### What would make me change this order

If the company's biggest pain point turned out to be shipping costs (not documentation time), I would prioritise carrier selection ML instead. The analysis identifies bottlenecks and recommends a priority, but the final decision belongs to the business. If the CFO says "our shipping costs are killing us," that changes the priority — and a good analyst adapts rather than defending their original recommendation.

# User Stories — Online Handset Purchase

Epic: **Customer purchases a bundled handset + tariff + insurance online without visiting a branch.**

---

## US-01 — Bundle selection on a single page

**As a** customer browsing for a new phone,
**I want to** select a handset, tariff plan, and insurance in one place,
**so that** I don't have to navigate across multiple pages and lose my selection.

**Acceptance Criteria:**

- [ ] The product detail page displays the handset, all available tariff plans, and insurance options in a single view
- [ ] Price breakdown (device price, monthly instalment, tariff fee, insurance fee) is visible before adding to cart
- [ ] Changing the instalment period recalculates the monthly amount without a page reload
- [ ] Cart is pre-populated with all three items when the customer clicks "Add to cart"

---

## US-02 — Instant credit check

**As a** customer who wants to pay in instalments,
**I want to** know immediately whether I qualify,
**so that** I don't spend time on checkout only to be rejected at the end.

**Acceptance Criteria:**

- [ ] Credit check is triggered automatically after the customer confirms their national ID
- [ ] Result (approved / rejected) is returned and displayed within 5 seconds
- [ ] If approved, available instalment options (12 / 24 / 36 months) are shown inline
- [ ] If rejected, the customer is offered a one-time payment alternative
- [ ] No manual review step; decision is fully automated via Credit Bureau API

---

## US-03 — Electronic contract signing

**As a** customer who has passed the credit check,
**I want to** sign the purchase contract online via OTP,
**so that** I don't have to visit a branch or print anything.

**Acceptance Criteria:**

- [ ] Contract PDF is generated automatically and shown for review before signing
- [ ] OTP is sent to the customer's verified mobile number
- [ ] Signing is completed by entering the OTP; no physical signature required
- [ ] Signed contract is available for download immediately after confirmation
- [ ] Order status transitions to `CONTRACT_SIGNED` upon successful OTP submission

---

## US-04 — SIM type selection (physical / eSIM)

**As a** customer completing checkout,
**I want to** choose between a physical SIM card and eSIM,
**so that** I can use the option that fits my device.

**Acceptance Criteria:**

- [ ] Both SIM options are presented at checkout with a short description
- [ ] eSIM is only offered for device models that support it (validated against product catalogue)
- [ ] Physical SIM triggers shipment; eSIM triggers automatic provisioning on payment
- [ ] Selected SIM type is stored on the order and passed to BSS activation

---

## US-05 — Number portability

**As a** customer switching from another carrier,
**I want to** keep my existing phone number,
**so that** I don't have to notify all my contacts of a new number.

**Acceptance Criteria:**

- [ ] Number portability option is presented after SIM type selection
- [ ] Customer enters their current MSISDN; system validates the format
- [ ] MNP (Mobile Number Portability) request is submitted automatically to the carrier exchange
- [ ] Customer is informed of expected porting timeline (typically 1 business day)
- [ ] If porting fails, customer is offered a new number as a fallback

---

## US-06 — Atomic service activation on payment

**As a** system,
**I want to** activate SIM, tariff, instalment plan, and insurance in a single atomic operation after payment,
**so that** the customer's services are live within seconds and there is no partial activation state.

**Acceptance Criteria:**

- [ ] Activation sequence: BSS (SIM + tariff) → BSS (instalment plan) → Insurance API → Notification
- [ ] If any activation step fails, all completed steps are rolled back and the order status is set to `ACTIVATION_FAILED`
- [ ] Customer receives an SMS and email confirmation only after all services are confirmed active
- [ ] Full activation completes within 60 seconds of `PAYMENT_SUCCESS`
- [ ] All activation events are logged with timestamps for audit purposes

---

## Definition of Done

- [ ] All acceptance criteria pass
- [ ] Happy path and at least one error path covered by E2E test
- [ ] API integration points documented in Postman collection
- [ ] State diagram updated if new order statuses are introduced
- [ ] Product owner sign-off

# Purchase Scenario — Handset + Tariff + Insurance

> Models the full online purchase journey at a telecommunications company.
> Scenario: customer buys a smartphone on instalment with a monthly tariff plan and device insurance.

---

## 1. Purchase Flow

```mermaid
flowchart TD
    A([Start]) --> B[Handset list\nwith tariff panel]
    B --> C[Select handset\nproduct detail page]
    C --> D[Select payment plan\nOne-time / 24 mo.]
    D --> E[Select insurance\nPre istotu Mini / None]
    E --> F[Add to cart]
    F --> G{Existing\ncustomer?}
    G -->|yes| H[Login\npersonalised discounts]
    G -->|no| I[New customer\ngo to store]
    H --> J[Select phone number]
    I --> J
    J --> K{Port existing number\nor new?}
    K -->|port| L[Enter number\nfrom other carrier]
    K -->|new| M[Assign new number]
    L --> N[Select SIM type\nPhysical / eSIM]
    M --> N
    N --> O[Cart summary\n+ promo code]
    O --> P[Checkout:\nPersonal details\nname, surname, national ID]
    P --> Q[System verifies\nnational ID + credit score]
    Q --> R{Score OK?}
    R -->|no| S([Instalment rejected])
    R -->|yes| T[Payment\ncard / other method]
    T --> U{Payment\napproved?}
    U -->|no| V[Retry payment]
    V --> U
    U -->|yes| W[Activation\nSIM + tariff + insurance]
    W --> X[Confirmation\nSMS + Email]
    X --> Y([End])

    subgraph EXTERNAL SYSTEMS
        Q --> C1[(Credit Bureau)]
        W --> C2[(BSS / Billing)]
        W --> C3[(Insurance API)]
        X --> C4[Notification Service]
    end
```

---

## 2. Sequence Diagram — System Communication

```mermaid
sequenceDiagram
    actor Customer
    participant WebShop
    participant CRM
    participant CreditCheck
    participant BSS as BSS / Billing
    participant Insurance
    participant Notification

    Customer->>WebShop: Add handset + tariff to cart
    WebShop->>CRM: Verify customer (email / national ID)
    CRM-->>WebShop: Customer found / new

    WebShop->>CreditCheck: Credit score request
    Note over CreditCheck: External bureau (e.g. CRIF)
    CreditCheck-->>WebShop: Score 720 / APPROVED

    WebShop->>Customer: Display instalment options (12 / 24 / 36 mo.)
    Customer->>WebShop: Select 24 months + insurance

    WebShop->>CRM: Create order (ORDER_CREATED)
    CRM-->>WebShop: orderId: ORD-2024-8821

    Customer->>WebShop: Sign contract electronically (OTP)
    WebShop->>CRM: Update status (CONTRACT_SIGNED)

    Customer->>WebShop: Pay first instalment (card)
    WebShop->>BSS: Process payment
    BSS-->>WebShop: PAYMENT_SUCCESS

    WebShop->>BSS: Activate tariff (customerId, planId)
    BSS-->>WebShop: SIM activated, tariff active

    WebShop->>BSS: Create instalment plan (24 × monthly)
    BSS-->>WebShop: Plan created

    WebShop->>Insurance: Activate device insurance
    Note over Insurance: IMEI, device value, customer
    Insurance-->>WebShop: Policy number: INS-2024-5521

    WebShop->>Notification: Send confirmation
    Notification->>Customer: SMS: "Your order ORD-8821 is confirmed"
    Notification->>Customer: Email: Contract PDF + invoice
```

---

## 3. State Diagram — Order Lifecycle

```mermaid
stateDiagram-v2
    [*] --> DRAFT : customer adds to cart

    DRAFT --> PENDING_CREDIT : confirms cart
    PENDING_CREDIT --> CREDIT_APPROVED : score OK
    PENDING_CREDIT --> CREDIT_REJECTED : score too low

    CREDIT_APPROVED --> PENDING_SIGNATURE : contract generated
    PENDING_SIGNATURE --> SIGNED : customer signs (OTP)

    SIGNED --> PENDING_PAYMENT : awaiting payment
    PENDING_PAYMENT --> PAYMENT_FAILED : payment declined
    PAYMENT_FAILED --> PENDING_PAYMENT : customer retries

    PENDING_PAYMENT --> PAID : payment successful
    PAID --> ACTIVATING : system activates services
    ACTIVATING --> ACTIVE : SIM + tariff + insurance active

    ACTIVE --> SUSPENDED : missed monthly instalment
    SUSPENDED --> ACTIVE : customer pays
    SUSPENDED --> CANCELLED : prolonged non-payment

    ACTIVE --> COMPLETED : contract expired (24 mo.)
    CREDIT_REJECTED --> [*]
    CANCELLED --> [*]
    COMPLETED --> [*]
```

---

## 4. ERD — Data Model

```mermaid
erDiagram
    CUSTOMER {
        uuid id PK
        string first_name
        string last_name
        string email
        string phone
        date birth_date
        int credit_score
        timestamp created_at
    }

    ORDER {
        uuid id PK
        uuid customer_id FK
        string status
        decimal total_amount
        timestamp created_at
        timestamp signed_at
        timestamp activated_at
    }

    ORDER_ITEM {
        uuid id PK
        uuid order_id FK
        string item_type
        uuid product_id FK
        decimal price
        int quantity
    }

    PRODUCT {
        uuid id PK
        string type
        string name
        decimal price
        string sku
    }

    TARIFF_PLAN {
        uuid id PK
        string name
        decimal monthly_fee
        int data_gb
        string call_minutes
        bool roaming_included
    }

    INSTALLMENT_PLAN {
        uuid id PK
        uuid order_id FK
        int total_months
        decimal monthly_amount
        int paid_months
        string status
    }

    PAYMENT {
        uuid id PK
        uuid order_id FK
        string method
        decimal amount
        string currency
        string status
        string gateway_reference
        timestamp initiated_at
        timestamp completed_at
    }

    INSURANCE_POLICY {
        uuid id PK
        uuid order_id FK
        string policy_number
        string device_imei
        decimal coverage_amount
        date valid_from
        date valid_to
        string status
    }

    SIM_CARD {
        uuid id PK
        uuid customer_id FK
        uuid tariff_plan_id FK
        string iccid
        string msisdn
        string status
        timestamp activated_at
    }

    CUSTOMER ||--o{ ORDER : "places"
    ORDER ||--|{ ORDER_ITEM : "contains"
    ORDER_ITEM }o--|| PRODUCT : "references"
    ORDER ||--o| INSTALLMENT_PLAN : "has"
    ORDER ||--o{ PAYMENT : "paid via"
    ORDER ||--o| INSURANCE_POLICY : "has"
    CUSTOMER ||--o{ SIM_CARD : "owns"
    SIM_CARD }o--|| TARIFF_PLAN : "uses"
```

---

## 5. AS-IS vs TO-BE

### AS-IS — before optimisation

```
Step 1: Select handset            (1 page)
Step 2: Select tariff plan        (1 page)
Step 3: Select insurance          (1 page)
Step 4: Registration / Login      (1 page)
Step 5: Credit verification       (manual, 1–2 days)
Step 6: Contract — print, sign    (branch visit or post)
Step 7: Activation                (manual, 24–48 hours)
Step 8: Confirmation              (email)

Total time: 2–5 days
Steps: 8
```

### TO-BE — optimised flow

```
Step 1: Select handset + tariff + insurance  (1 page, bundle)
Step 2: Identify customer (eID or existing account)
Step 3: Automated credit check               (real-time API, ~3 sec)
Step 4: Electronic signature                 (OTP via SMS)
Step 5: Payment                              (card / Apple Pay)
→ Automatic activation: SIM + tariff + insurance

Total time: ~15 minutes
Steps: 5 (reduced from 8)
```

### Where steps were removed

| Removed step                  | How                              | Impact                  |
| ----------------------------- | -------------------------------- | ----------------------- |
| Separate page per product     | Bundle selection on 1 page       | −3 steps, −30% drop-off |
| Manual credit verification    | Real-time API to Credit Bureau   | 2 days → 3 seconds      |
| Paper contract + branch visit | Electronic signature via OTP     | No branch visit needed  |
| Manual SIM activation         | API call to BSS on payment event | 48 hours → instant      |

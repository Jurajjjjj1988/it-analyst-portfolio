# Nákupný scenár — Telefón + Paušál + Poistenie

> Modeluje kompletný nákupný tok zákazníka v telekomunikačnej spoločnosti.
> Scenár: zákazník si kúpi smartphone na splátky s paušálnym tarifom a poistením zariadenia.

---

## 1. Purchase Flow — Nákupný proces (telekom.sk)

```mermaid
flowchart TD
    A([Start]) --> B[Zoznam telefónov\ns paušálom vpravo]
    B --> C[Vyberie telefón\ndetail produktu]
    C --> D[Vyberie splátky\nJednorazovo / 24 mes.]
    D --> E[Vyberie poistenie\nPre istotu Mini / Bez balíčka]
    E --> F[Pridať do košíka]
    F --> G{Existujúci\nzákazník?}
    G -->|áno| H[Prihlásenie\nzľavy na mieru]
    G -->|nie| I[Nový zákazník\nprejsť do Telekomu]
    H --> J[Výber čísla]
    I --> J
    J --> K{Preniesť číslo\nalebo nové?}
    K -->|preniesť| L[Zadá číslo\nod iného operátora]
    K -->|nové| M[Zriadi nové číslo]
    L --> N[Výber SIM\nPlastová / eSIM]
    M --> N
    N --> O[Košík\nzhrnutie + zľavový kupón]
    O --> P[Checkout:\nOsobné údaje\nmeno, priezvisko, RČ]
    P --> Q[Systém overí\nrodné číslo + kreditné skóre]
    Q --> R{Skóre OK?}
    R -->|nie| S([Zamietnutie splátok])
    R -->|áno| T[Platba\nkarta / iná metóda]
    T --> U{Platba\nschválená?}
    U -->|nie| V[Opakuje platbu]
    V --> U
    U -->|áno| W[Aktivácia\nSIM + paušál + poistenie]
    W --> X[Potvrdenie\nSMS + Email]
    X --> Y([End])

    subgraph EXTERNÉ SYSTÉMY
        Q --> C1[(Kreditný búreau)]
        W --> C2[(BSS/Billing)]
        W --> C3[(Pre istotu API)]
        X --> C4[Notifikačný systém]
    end
```

---

## 2. Sekvenčný diagram — Systémová komunikácia

```mermaid
sequenceDiagram
    actor Zákazník
    participant WebShop
    participant CRM
    participant CreditCheck
    participant BSS as BSS/Billing
    participant Insurance as Poistňovňa
    participant Notification

    Zákazník->>WebShop: Pridá telefón + paušál do košíka
    WebShop->>CRM: Overí zákazníka (email/rodné číslo)
    CRM-->>WebShop: Zákazník existuje / nový

    WebShop->>CreditCheck: Požiadavka na kreditné skóre
    Note over CreditCheck: Externý bureau (napr. CRIF)
    CreditCheck-->>WebShop: Skóre 720 / APPROVED

    WebShop->>Zákazník: Zobrazí dostupné splátky (12/24/36 mes.)
    Zákazník->>WebShop: Vyberie 24 mesiacov + poistenie

    WebShop->>CRM: Vytvorí objednávku (ORDER_CREATED)
    CRM-->>WebShop: orderId: ORD-2024-8821

    Zákazník->>WebShop: Podpíše zmluvu elektronicky
    WebShop->>CRM: Aktualizuje stav (CONTRACT_SIGNED)

    Zákazník->>WebShop: Platí prvú splátku (kartou)
    WebShop->>BSS: Spracuj platbu
    BSS-->>WebShop: PAYMENT_SUCCESS

    WebShop->>BSS: Aktivuj tarif (customerId, planId)
    BSS-->>WebShop: SIM aktivovaná, tarif aktívny

    WebShop->>BSS: Vytvor splátkovací plán (24x mesačne)
    BSS-->>WebShop: Plán vytvorený

    WebShop->>Insurance: Aktivuj poistenie zariadenia
    Note over Insurance: IMEI, hodnota, zákazník
    Insurance-->>WebShop: Poistka číslo: INS-2024-5521

    WebShop->>Notification: Odošli potvrdenie
    Notification->>Zákazník: SMS: "Vaša objednávka ORD-8821 je potvrdená"
    Notification->>Zákazník: Email: Zmluva PDF + faktúra
```

---

## 3. Stavový diagram — Stavy objednávky

```mermaid
stateDiagram-v2
    [*] --> DRAFT : zákazník pridá do košíka

    DRAFT --> PENDING_CREDIT : potvrdí košík
    PENDING_CREDIT --> CREDIT_APPROVED : skóre OK
    PENDING_CREDIT --> CREDIT_REJECTED : skóre nízke

    CREDIT_APPROVED --> PENDING_SIGNATURE : zmluva vygenerovaná
    PENDING_SIGNATURE --> SIGNED : zákazník podpíše

    SIGNED --> PENDING_PAYMENT : čaká na platbu
    PENDING_PAYMENT --> PAYMENT_FAILED : platba zamietnutá
    PAYMENT_FAILED --> PENDING_PAYMENT : zákazník opakuje

    PENDING_PAYMENT --> PAID : platba úspešná
    PAID --> ACTIVATING : systém aktivuje služby
    ACTIVATING --> ACTIVE : SIM + tarif + poistenie aktívne

    ACTIVE --> SUSPENDED : nezaplatená mesačná splátka
    SUSPENDED --> ACTIVE : zákazník zaplatí
    SUSPENDED --> CANCELLED : dlhodobé neplatenie

    ACTIVE --> COMPLETED : zmluva vypršala (24 mes.)
    CREDIT_REJECTED --> [*]
    CANCELLED --> [*]
    COMPLETED --> [*]
```

---

## 4. ERD — Dátový model

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
    ORDER ||--o| INSURANCE_POLICY : "has"
    CUSTOMER ||--o{ SIM_CARD : "owns"
    SIM_CARD }o--|| TARIFF_PLAN : "uses"
```

---

## 5. Popis skrátenia nákupného scenára

### AS-IS (pôvodný proces — pred optimalizáciou)

```
Krok 1: Výber telefónu          (1 stránka)
Krok 2: Výber paušálu           (1 stránka)
Krok 3: Výber poistenia         (1 stránka)
Krok 4: Registrácia / Login     (1 stránka)
Krok 5: Kreditná verifikácia    (manuálne, 1-2 dni)
Krok 6: Zmluva — tlač, podpis   (pobočka alebo pošta)
Krok 7: Aktivácia               (manuálne, 24-48 hodín)
Krok 8: Potvrdenie              (email)

Celkový čas: 2-5 dní
Kroky: 8
```

### TO-BE (optimalizovaný proces)

```
Krok 1: Výber telefónu + paušálu + poistenia  (1 stránka, bundle)
Krok 2: Identifikácia (eID alebo existujúci účet)
Krok 3: Automatická kreditná verifikácia      (real-time, API)
Krok 4: Elektronický podpis                  (OTP cez SMS)
Krok 5: Platba                               (karta / Apple Pay)
→ Automatická aktivácia SIM + tarif + poistenie

Celkový čas: 15 minút
Kroky: 5 (z 8 na 5)
```

### Kde boli odstránené kroky

| Odstránený krok                      | Ako                            | Prínos                  |
| ------------------------------------ | ------------------------------ | ----------------------- |
| Samostatné stránky pre každý produkt | Bundle výber na 1 stránke      | -3 kroky, -30% drop-off |
| Manuálna kreditná verifikácia        | Real-time API do Credit Bureau | 2 dni → 3 sekundy       |
| Papierová zmluva                     | Elektronický podpis cez OTP    | Pobočka nie je potrebná |
| Manuálna aktivácia                   | API volanie do BSS pri platbe  | 48 hodín → okamžite     |

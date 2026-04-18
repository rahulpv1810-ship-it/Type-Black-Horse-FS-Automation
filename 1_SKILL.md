---
name: bh-tb-mapper
description: Black Horse FS Automation — unified Trial Balance mapping engine for ALL Indian entity types. Detects entity type first (Corporate Entity = Pvt Ltd / Public Ltd / OPC / Section 8, OR Non-Corporate Entity = Partnership / Proprietorship / LLP / HUF / Sole Trader), applies entity-specific rules, then processes all common Substance-Over-Form rules to classify every ledger into Face Grouping → Note Grouping → Sub-Note Grouping, producing a 6-column Ledger Mapping Review Report for the Black Horse utility. Use this skill whenever the user uploads or pastes a Trial Balance and asks to map ledgers, run Black Horse mapping, prepare FS groupings, classify a TB, or generate a financial statement grouping table — for any entity type. Also trigger for "Black Horse TB", "map this TB", "CE mapping", "NCE mapping", "Black Horse corporate", "Black Horse partnership", "classify ledgers", "ledger grouping", "Schedule III mapping", or "prepare FS groupings". Always use this skill before any manual ledger classification.
---

# Black Horse FS Automation — Unified TB Mapper

**Role:** Senior Statutory Auditor applying strict "Substance Over Form" logic to map Trial Balances to the Black Horse Utility for any Indian entity type.

---

## ═══ STAGE 1: ENTITY DETECTION GATE ═══

**This is the first and mandatory step. Do not process any ledger until entity type is confirmed.**

### How to Detect Entity Type

Scan the TB header, file name, or user's description for these signals:

| Signal Found | Entity Type | Route To |
|---|---|---|
| "Pvt Ltd", "Private Limited", "Limited", "OPC", "Section 8 Company", "CIN" | **CE — Corporate Entity** | `references/ce-rules.md` |
| "Partnership Firm", "Partner", "Proprietorship", "Proprietor", "LLP", "HUF", "Sole Trader", "Capital A/c" | **NCE — Non-Corporate Entity** | `references/nce-rules.md` |
| Ambiguous / Not clear | → **Ask the user:** *"Is this entity a Company (Pvt Ltd / OPC) or a Non-Corporate Entity (Partnership / Proprietorship / LLP / HUF)?"* |

### After Detection

1. **State the detection result** clearly:
   > *Entity detected as: [CE / NCE] — [Entity Name if available]. Loading [CE / NCE] rules.*

2. **Load the correct reference file immediately:**
   - CE → read `references/ce-rules.md` (Rules 1, 4, 6, 7, 8 + CE Sub-Note Tier 2 + CE Master Dictionary)
   - NCE → read `references/nce-rules.md` (Rules 1, 4, 6, 7, 8 + NCE Sub-Note Tier 2 + NCE Master Dictionary)

3. **Then continue with the Pre-Flight Checklist below.**

---

## ═══ STAGE 2: PRE-FLIGHT CHECKLIST ═══

Run this for every TB before mapping any ledger:

- [ ] Entity type confirmed (CE or NCE)
- [ ] FY and reporting date identified (CY column vs PY column)
- [ ] Entity nature confirmed: Manufacturing / Trading / Service / Mixed
- [ ] TB balance checked — Debit total = Credit total (±rounding). If unbalanced → ⚑ flag and stop.
- [ ] Zero-balance and PY-only ledgers identified for exclusion (do not render "Not in current TB" or "PY Zero Balance" rows in output)
- [ ] All ledgers have an amount — if any blank balance → ⚑ Missing balance flag

---

## ═══ STAGE 3: APPLY ENTITY-SPECIFIC RULES FIRST ═══

**From the loaded reference file, apply these entity-specific rules in sequence:**

| Rule No. | Rule Name | Why Entity-Specific |
|---|---|---|
| Rule 1 | Bank OD Reclassification | CE uses "Cash and cash equivalents" / NCE uses "Cash and bank balances" |
| Rule 4 | Tax Integrity | CE has long-term Advance Tax split / NCE routes all debit taxes to short-term |
| Rule 6 | Equity Structure | CE = Share Capital & Reserves / NCE = Owners Capital + Partners' Remuneration |
| Rule 7 | Party Flipped Balances | CE flipped creditor → "Advances to suppliers" / NCE → "Advances to Creditors" |
| Rule 8 | Fixed Deposit Tenor | CE parent face = "Cash and cash equivalents" / NCE = "Cash and bank balances" |

Full rule text, tables, and the Master Dictionary for each entity type are in the respective reference file.

---

## ═══ STAGE 4: APPLY COMMON RULES (BOTH CE AND NCE) ═══

These rules are **identical** for all entity types. Apply them after entity-specific rules.

---

### STEP 0 — Analyze Balance & Substance

Determine if the balance is Debit (+) or Credit (−). Identify if the balance contradicts the ledger name (e.g., an Overdraft with a Debit balance is Cash, not a Liability). Always apply **Substance Over Form**.

---

### RULE 2 — Trading Direct vs. Cost of Goods Sold

**IF** Manufacturing entity AND ledger implies direct processing *(Making, Hallmarking, Workshop, Consumables, Labour)*:
- Face: `Cost of material consumed` → Note: `Captured from Q_Disc`

**IF** Trading/Transport entity AND ledger contains *"Cooli", "Freight", "Loading", "Unloading",* or *"Carriage"*:
- Face: `Other expenses` → Note: `Direct expenses` *(or `Freight Inward` / `Freight outward` if explicitly matched)*
- **Do NOT route to Q_Disc.**

> **Audit Principle:** Q_Disc is reserved for manufacturing cost schedules. Freight in a trading firm is a period cost under Other Expenses.

---

### RULE 3 — Admin & Misc Hierarchy (4-Priority Cascade)

Apply **in order** for all office/general expense ledgers. Stop at first match.

| Priority | Condition | Face | Note | Sub-Note |
|---|---|---|---|---|
| 1 — Direct Named Match | Ledger name directly matches a Note head (Telephone, Rent, Insurance…) | `Other expenses` | Exact Note head | BLANK |
| 2 — Strict Miscellaneous | "Miscellaneous expenses", "Misc Exp", "Sundry Expenses", "Sundry Exp", or near-synonym | `Other expenses` | `Miscellaneous expenses` | **FORCE BLANK** (hard override, no exceptions) |
| 3 — Tangible/Operational | Can point to a specific physical item or transaction (Printing, Stationery, Courier, Refreshments, Pantry) | `Other expenses` | `Other Business Administrative Expenses` | BLANK |
| 4 — General Catch-All | General unidentifiable overhead (General Expenses, Office Overheads, Accounting Charges) | `Other expenses` | `Administrative Expenses` | BLANK |

> **Audit Principle:** This hierarchy prevents an over-populated Miscellaneous line and forces specific classification wherever the ledger name permits identification.

---

### RULE 5 — Fintech, MDR & Mixed Finance Ledger Handling

**Sub-Rule 5A — Pure Fintech / MDR**
IF ledger contains NBFC or Payment Gateway identifiers *(Bajaj, DMI, Chola, HDB, PayTM, Pine Labs, Razorpay, or similar)*:
- Face: `Finance costs` → Note: `Other borrowing costs` → Sub-Note: BLANK. **STOP.**

**Sub-Rule 5B — Mixed Bank Charges & Interest**
IF ledger contains both "charges/fees" AND "interest":
- **Step 1:** Ledger leads with "Interest" (e.g., *"Interest & Bank Charges"*) → Face: `Finance costs` → Note: `Interest expense`. **STOP.**
- **Step 2:** No rename signal → apply materiality: charges immaterial → `Interest expense`; charges material or indeterminate → `Other borrowing costs` + raise ⚑ flag recommending bifurcation.

> **Audit Principle:** MDR and fintech fees are borrowing costs, not bank charges. Substance — cost of accessing credit or payment infrastructure — determines classification.

---

### RULE 9 — Rent Advance vs. Security Deposit Classification

| Scenario | Face Grouping | Note Grouping | Action |
|---|---|---|---|
| Small/short-term advance to landlord before occupation | Other current assets | Prepaid Expenses *(NCE)* / Others — Prepaid Rent *(CE)* | — |
| Large/long-term deposit for full lease tenure | Other non current assets | Security Deposits | — |
| Utility/insurance/govt deposits (KSEB, BESCOM, New India Assurance, etc.) | Other non current assets | Security Deposits | — |
| Nature/amount not specified | — | — | **Raise ⚑ flag** |

> ⚑ **Flag text:** *Unable to confirm deposit nature — short-term advance or long-term security? Obtain lease agreement from client.*

> **Audit Principle:** A security deposit is a financial asset held against a lease obligation — it must sit in non-current assets for the lease duration, not be expensed.

---

### RULE 10 — Power, Fuel & Petrol End-Use Classification

**NEVER** map fuel/petrol/diesel to `"Power and fuel"` based on ledger name alone. Always determine **end-use** first.

**Step 1 — Scan TB before deciding:** Check FAR for generators, plant machinery. Check for separate Freight/Delivery ledger. Confirm entity type.

**Step 2 — Elimination matrix:**

| End-Use | Confirmed FAR Asset? | Separate Freight Ledger? | → Note Grouping |
|---|---|---|---|
| Staff vehicles / director cars / travel | No generator/machinery | — | `Conveyance expenses` |
| Factory generators / plant machinery / pumps | Yes — generator or machine in FAR | — | `Power and fuel` |
| Owned delivery trucks / goods despatch | — | No freight ledger exists | `Freight outward` or `Selling & Distribution Expenses` |

**Elimination sequence:** If freight ledger exists → rule out Scenario 3. If no machinery in FAR → rule out Scenario 2. If both ruled out → default to **Conveyance expenses**.

> **Golden Rule:** Never map fuel to Power and fuel just because it creates energy. End-use of energy — not the nature of energy — drives classification.

---

### RULE 11 — Fixed Assets & Depreciation (Common to Both)

| Asset Category | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| Tangible Assets (AC, Computers, Furniture, Plant, Vehicles, Land, Buildings…) | Property Plant and Equipment | Captured from FAR | FORCE BLANK |
| Intangible Assets (Software, Goodwill, Patents, Brand Registration…) | Intangible assets | Captured from FAR | FORCE BLANK |
| Capital Work in Progress | Capital work in progress | Captured in Notes-2 | FORCE BLANK |
| Accumulated Depreciation | Depreciation and amortization expenses | Depreciation on property, plant and equipment | FORCE BLANK |
| Accumulated Amortization | Depreciation and amortization expenses | Amortization of intangible assets | FORCE BLANK |

---

## ═══ STAGE 5: SUB-NOTE UNIVERSAL RULES ═══

### Tier 1 — ALWAYS BLANK (Hard Override — No Exceptions)

Apply regardless of entity type. Sub-Note must be blank if **any** of:
- Note contains `"FAR"`, `"Q_Disc"`, `"Notes-2"`, or `"Partners sheet"`
- Note equals `"Miscellaneous expenses"`
- Note Grouping exactly matches the Ledger Name
- Face has only one valid Note option in the Master Dictionary

### Tier 2 — Sub-Note MANDATORY

Varies by entity type. See `references/ce-rules.md` (Tier 2 table — CE) or `references/nce-rules.md` (Tier 2 table — NCE).
For all Tier 2 rows: Sub-Note = **Title-Cased Ledger Name**.

### Tier 3 — All Other Combinations → Sub-Note = BLANK

### Formatting Rule (Always)
Expand abbreviations: Exp → Expenses, A/c → Account, Maint → Maintenance. Apply strict Title Case. Remove redundant descriptors.

---

## ═══ OUTPUT FORMAT ═══

**Mandatory 6-column table — both CE and NCE:**

| Sl. No. | Name of Ledger | Amount (₹) | Face Grouping | Note Grouping | Sub-Note Grouping |

- **Amount:** Unified — Debit = Positive (+), Credit = Negative (−)
- **Sequence:** Never sort alphabetically. Preserve original TB ledger sequence. Append new CY-only ledgers at the bottom.
- **Syntax Lock:** Face and Note must exactly match the loaded Master Dictionary (spelling, casing, punctuation).
- **Exclusions:** Do not render rows with "Not in current TB" or "PY Zero Balance".

---

## ═══ EXECUTION WORKFLOW ═══

### Phase 1 — Forensic Scan (Training Edition)

For each reclassified or flagged ledger, use exactly this format:

> **[Ledger Name]:** Currently classified as [Old Mapping]; correct mapping is **[Face] → [Note]**.
> **Audit Principle:** [1–2 sentence Substance-Over-Form explanation.]

Compile a consolidated **⚑ Flag Registry** at the end of Phase 1 listing all items requiring client clarification.

**STOP. Wait for user to say "Agree" or "Proceed" before Phase 2.**

### Phase 2 — Final Mapping Matrix

Generate the complete 6-column markdown table for all active ledgers using **exactly** this header:

`| Sl. No. | Name of Ledger | Amount (₹) | Face Grouping_Current Year | Note Grouping_Current Year | Sub-Note Grouping_Current Year |`

**Strict Mapping Constraints:**

- **Master Dictionary Lock:** You MUST populate the "Face Grouping" and "Note Grouping" columns using only the exact strings found in the **Master Dictionary (JSON)** at the bottom of the entity-specific reference file (`ce-rules.md` or `nce-rules.md`).
- **Zero-Drift Policy:** Character-for-character accuracy is mandatory. Any deviation in spelling, casing, or punctuation will cause a Black Horse utility rejection.
- **Amount Formatting:** All amounts must be rounded to the nearest whole number (ignore decimal places) and follow the sign convention: Debit = Positive (+), Credit = Negative (−).
- **Sub-Note Execution:** Apply the **Stage 5 Sub-Note Universal Rules** to determine if the final column remains `[BLANK]` or contains the `Title-Cased Ledger Name`.

Append the **⚑ Flag Registry** immediately below the table.

---

## ═══ REFERENCE FILES ═══

| File | Load When |
|---|---|
| `references/ce-rules.md` | Entity detected as Corporate (Pvt Ltd / Public Ltd / OPC / Section 8) |
| `references/nce-rules.md` | Entity detected as Non-Corporate (Partnership / Proprietorship / LLP / HUF) |

**Always load the appropriate reference file before beginning Phase 1.**
The reference file contains: entity-specific Rules 1, 4, 6, 7, 8 → Sub-Note Tier 2 table → full Master Dictionary.

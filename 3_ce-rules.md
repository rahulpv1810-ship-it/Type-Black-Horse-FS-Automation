# CE-Specific Rules & Master Dictionary — Black Horse v4.0
### Corporate Entities: Pvt Ltd / Public Ltd / OPC / Section 8 Company
### Applicable Standard: Schedule III — Companies Act 2013

> Load this file when entity type = **Corporate Entity (CE)**.
> After loading, apply Rules 1, 4, 6, 7, 8 (CE variants) first, then return to SKILL.md for common Rules 2, 3, 5, 9, 10, 11.

---

## TABLE OF CONTENTS
1. CE Rule 1 — Bank OD Reclassification
2. CE Rule 4 — Tax Integrity (with Long-Term Split)
3. CE Rule 6 — Share Capital & Reserves Integrity
4. CE Rule 7 — Party Integrity & Flipped Balances
5. CE Rule 8 — Fixed Deposit Tenor Classification
6. CE Intelligence Flags (AI-Enhanced)
7. CE Sub-Note Tier 2 Mandatory Table
8. CE Master Dictionary (JSON)

---

## CE RULE 1 — Bank OD Reclassification

**IF** ledger contains `"OD"`, `"OCC"`, or `"Overdraft"`:

| Balance | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| Debit (+) → Reclassify as ASSET | **Cash and cash equivalents** | Balances with banks in current accounts | [Bank Name] OD (Debit Balance) |
| Credit (−) → Retain as LIABILITY | Short term borrowings | Secured Loans repayable on demand from banks *(or Unsecured variant)* | BLANK |

> **Audit Principle:** A bank OD with a debit balance is a cash asset in substance — the entity has deposited more than it drew. Mapping it as a liability overstates liabilities and understates liquidity.

---

## CE RULE 4 — Tax Integrity (Debit vs. Credit)

| Scenario | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| Debit — Short-term *(TDS Receivable, ITC, GST Input Credit, current-year Advance Tax)* | Short term loans and advances | Balances with Government Authorities | BLANK |
| Debit — Long-term *(non-current Advance Income Tax, MAT Credit Entitlement)* | **Long term loans and advances** | Advance Income Tax (Net of provision for taxes) | BLANK |
| Credit *(TDS Payable, GST Payable, PF Payable, PT Payable, ESI Payable, etc.)* | Other current liabilities | Statutory dues *(or exact payable name from CE dictionary)* | BLANK |

> **Audit Principle:** Schedule III requires gross presentation — debit and credit tax ledgers must never be netted. Long-term advance tax sits in non-current assets; current-year advance tax sits in current assets.

---

## CE RULE 6 — Share Capital & Reserves Integrity

| Ledger Type | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| Equity Share Capital | Share capital | Issued Equity Share Capital | BLANK |
| Preference Share Capital | Share capital | Preference Share Capital | BLANK |
| Retained Earnings / Profit & Loss | Reserves and surplus | Profit & Loss Account | BLANK |
| General Reserve | Reserves and surplus | General Reserve | BLANK |
| Revaluation Reserve | Reserves and surplus | Revaluation Reserve | BLANK |
| Share Options Outstanding | Reserves and surplus | Share Options Outstanding Account | BLANK |
| MAT Credit Entitlement | Reserves and surplus | MAT Credit Entitlement Reserve | BLANK |
| Share Application Money | Share application money pending allotment | Specify at level 3 | **MANDATORY — Title-Cased Ledger Name** |
| Share Warrants | Money received against share warrants | Specify at level 3 | **MANDATORY — Title-Cased Ledger Name** |

**NEVER** route dividend payments, interim dividend declarations, or capital reduction entries to P&L income.

> **Audit Principle:** Schedule III mandates Share Capital and Reserves as separate equity components. Dividend appropriations are equity transactions, not income statement items.

---

## CE RULE 7 — Party Integrity & Flipped Balances

Scan **all** ledger names for identifiers: `limited`, `private`, `pvt`, `ltd`, `pharma`, `medical`, `enterprises`, `agencies`, `systems`, `solutions`, `traders`. **NEVER** map these to P&L.

| Scenario | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| Normal Debit (+) balance | Trade receivables | Unsecured considered good | Title-Cased Ledger Name |
| Normal Credit (−) balance | Trade payables due to others | Due to others | Title-Cased Ledger Name |
| FLIPPED — Debit-balance Creditor | Short term loans and advances | **Advances to suppliers** | BLANK |
| FLIPPED — Credit-balance Debtor | Other current liabilities | Advances from customers | BLANK |

> **Audit Principle:** A vendor balance flipped to Debit is an advance paid in excess of the liability — it is an asset. Substance Over Form requires reclassification out of Trade Payables.

---

## CE RULE 8 — Fixed Deposit Tenor Classification

**IF** ledger contains `"Fixed Deposit"`, `"FD"`, or `"Term Deposit"` AND Debit (+) balance:

| Tenor | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| ≤ 3 months | **Cash and cash equivalents** | Bank Deposit having maturity of less than 3 months | BLANK |
| > 3 months and ≤ 12 months | **Cash and cash equivalents** | Bank Deposit having maturity of greater than 3 months and less than 12 months | BLANK |
| > 12 months | Other non current assets | Bank Deposit having maturity of greater than 12 months | BLANK |
| Unknown / Not Specified | **Cash and cash equivalents** | Bank Deposit having maturity of greater than 3 months and less than 12 months | **⚑ Raise flag** |

For FDs with tenor > 12 months: also book a negative entry under `Cash and cash equivalents` → `Less: Deposits reclassified to other non current assets`.

> ⚑ **Flag text:** *FD tenor not confirmed — obtain maturity certificate from client before finalising classification.*

---

## CE INTELLIGENCE FLAGS (AI-Enhanced)

Apply these during Phase 1 in addition to all base rules:

**⚑ MSME Creditor Alert**
If any Trade Payable ledger mentions "MSME", "Micro", or "Small Enterprise" → flag for Sec 43B(h) — 45-day payment rule. Confirm payment dates vs. invoice dates.

**⚑ Director / Shareholder Loan Alert**
If any loan ledger mentions "Director", "Shareholder", "Promoter", or "Related Party" → flag for:
- Sec 185 / 186 — Loans to directors and related parties
- Sec 2(22)(e) — Deemed dividend risk if loan > accumulated profits

**⚑ Advances to Directors**
If "Advances" + Director/Promoter name found in the same ledger → flag Sec 185 violation risk immediately.

**⚑ Depreciation Double-Count Check**
If both an asset ledger AND its "Accumulated Depreciation" ledger exist → verify no double-charging of depreciation in P&L.

**⚑ Share Application Money Ageing**
If Share Application Money balance exists → flag for Companies Act timeline compliance — allotment must occur within 60 days of receipt.

**⚑ Unbalanced TB**
If Debit ≠ Credit total → stop and flag. Do not map an unbalanced TB without explicit user confirmation.

---

## CE SUB-NOTE TIER 2 — MANDATORY TABLE

For all rows below, Sub-Note = **Title-Cased Ledger Name** (Black Horse rejects blank in these slots).

| Face Grouping | Note Grouping |
|---|---|
| Cash and cash equivalents | Others |
| Current investments | Other investments |
| Deferred tax assets net | Specify at level 3 |
| Deferred tax liabilities Net | Specify at level 3 |
| Depreciation and amortization expenses | Specify at level 3 |
| Exceptional item | Specify at level 3 |
| Long term borrowings | Secured Other loans and advances |
| Long term borrowings | Unsecured Other loans and advances |
| Long term loans and advances | Other loans and advances (Secured, considered good) |
| Long term loans and advances | Other loans and advances (Unsecured, considered good) |
| Long term loans and advances | Other loans and advances (Doubtful) |
| Long term loans and advances | Others |
| Long term provisions | Provision for others |
| Long term provisions | Others |
| Money received against share warrants | Specify at level 3 |
| Non current investments | Other non-current investments |
| Other current assets | Specify at level 3 |
| Prior Period Item | Specify at level 3 |
| Profit loss from discontinuing operation before tax | Specify at level 3 |
| Share application money pending allotment | Specify at level 3 |
| Short term provisions | Specify at level 3 |
| Tax expenses of discontinuing operation | Specify at level 3 |

---

## CE MASTER DICTIONARY (JSON) — Black Horse v4.0

```json
{
  "bh_ce_mappings": {
    "Capital work in progress": ["Captured in Notes-2"],
    "Cash and cash equivalents": [
      "Cash on hand",
      "Cheques, drafts on hand",
      "Balances with banks in current accounts",
      "Bank Deposit having maturity of less than 3 months",
      "Bank Deposit having maturity of greater than 3 months and less than 12 months",
      "Bank Deposit having maturity of greater than 12 months",
      "Others",
      "Less: Deposits reclassified to other non current assets"
    ],
    "Change in Inventories of work in progress and finished goods": ["Captured from Q_Disc"],
    "Cost of material consumed": ["Captured from Q_Disc"],
    "Current investments": [
      "Quoted Trade Investments in Equity Instruments",
      "Quoted Trade Investments in preference shares",
      "Quoted Trade Investments in Government or trust securities",
      "Quoted Trade Investments in debentures or bonds",
      "Quoted Trade Investments in Mutual Funds",
      "Unquoted Trade Investments in Equity Instruments",
      "Unquoted Trade Investments in preference shares",
      "Unquoted Trade Investments in Government or trust securities",
      "Unquoted Trade Investments in debentures or bonds",
      "Unquoted Trade Investments in Mutual Funds",
      "Quoted Other Investments in Equity Instruments",
      "Quoted Other Investments in preference shares",
      "Quoted Other Investments in Government or trust securities",
      "Quoted Other Investments in debentures or bonds",
      "Quoted Other Investments in Mutual Funds",
      "Unquoted Other Investments in Equity Instruments",
      "Unquoted Other Investments in preference shares",
      "Unquoted Other Investments in Government or trust securities",
      "Unquoted Other Investments in debentures or bonds",
      "Unquoted Other Investments in Mutual Funds",
      "Investments in partnership firms",
      "Other investments"
    ],
    "Deferred tax assets net": ["Specify at level 3"],
    "Deferred tax liabilities Net": ["Specify at level 3"],
    "Depreciation and amortization expenses": [
      "Depreciation on property, plant and equipment",
      "Amortization of intangible assets",
      "Depreciation on investment property",
      "Specify at level 3"
    ],
    "Employee benefit expenses": [
      "Salaries and wages",
      "Contribution to provident and other funds",
      "Expense on ESOP and ESPP",
      "Staff welfare expenses",
      "Specify at level 3"
    ],
    "Exceptional item": ["Specify at level 3"],
    "Finance costs": [
      "Interest expense",
      "Other borrowing costs",
      "Applicable net gain/loss on foreign currency transactions and translation",
      "Specify at level 3"
    ],
    "Intangible assets": ["Captured from FAR"],
    "Intangible assets under development": ["Captured in Notes-2"],
    "Inventories": ["Captured from Q_Disc"],
    "Long term borrowings": [
      "Secured Bonds/debentures",
      "Secured Term loans from banks",
      "Secured Term loans from other parties",
      "Secured Deferred payment liabilities",
      "Secured Deposits",
      "Secured Loans and advances from related parties",
      "Secured Long term maturities of finance lease obligations",
      "Secured Other loans and advances",
      "Unsecured Bonds/debentures",
      "Unsecured Term loans from banks",
      "Unsecured Term loans from other parties",
      "Unsecured Deferred payment liabilities",
      "Unsecured Deposits",
      "Unsecured Loans and advances from related parties",
      "Unsecured Long term maturities of finance lease obligations",
      "Unsecured Other loans and advances"
    ],
    "Long term loans and advances": [
      "Capital Advances",
      "Loans and advances to related parties",
      "Advance Income Tax (Net of provision for taxes)",
      "Balances with Government Authorities",
      "Other loans and advances (Secured, considered good)",
      "Other loans and advances (Unsecured, considered good)",
      "Other loans and advances (Doubtful)",
      "Provision for doubtful advances",
      "Others"
    ],
    "Long term provisions": [
      "Provision for employee benefits",
      "Provision for others",
      "Others"
    ],
    "Money received against share warrants": ["Specify at level 3"],
    "Non current investments": [
      "Investment property",
      "Quoted Trade Investments in Equity Instruments",
      "Quoted Trade Investments in preference shares",
      "Quoted Trade Investments in Government or trust securities",
      "Quoted Trade Investments in debentures or bonds",
      "Quoted Trade Investments in Mutual Funds",
      "Unquoted Trade Investments in Equity Instruments",
      "Unquoted Trade Investments in preference shares",
      "Unquoted Other Investments in Equity Instruments",
      "Unquoted Other Investments in preference shares",
      "Investments in partnership firms",
      "Other non-current investments"
    ],
    "Other current assets": [
      "Interest accrued",
      "Assets held for disposal",
      "Others",
      "Specify at level 3"
    ],
    "Other current liabilities": [
      "Current maturities of finance lease obligations",
      "Interest accrued but not due on borrowings",
      "Interest accrued and due on borrowings",
      "Income received in advance",
      "Unpaid dividends",
      "Application money received for allotment of securities and due for refund and interest accrued thereon",
      "Unpaid matured deposits and interest accrued thereon",
      "Unpaid matured debentures and interest accrued thereon",
      "Statutory dues",
      "Salaries and wages payable",
      "Advances from customers",
      "Creditors for capital goods",
      "Other payables",
      "Specify at level 3"
    ],
    "Other expenses": [
      "Auditors' Remuneration",
      "Administrative Expenses",
      "Advertisement",
      "Bad debts",
      "Commission",
      "Consultancy fees",
      "Consumption of stores and spare parts",
      "Conveyance expenses",
      "Direct expenses",
      "Freight Inward",
      "Freight outward",
      "Indirect expenses",
      "Insurance",
      "Manufacturing Expenses",
      "Power and fuel",
      "Professional fees",
      "Provision for bad and doubtful debts",
      "Rent",
      "Repairs to buildings",
      "Repairs to machinery",
      "Repairs others",
      "Rates and taxes",
      "Royalty",
      "Selling & Distribution Expenses",
      "Other Business Administrative Expenses",
      "Telephone expenses",
      "Travelling Expenses",
      "Miscellaneous expenses",
      "Other Expenses",
      "Specify at level 3"
    ],
    "Other Income": [
      "Interest Income",
      "Dividend Income",
      "Net gain/loss on sale of investments",
      "Other non-operating income (net of expenses)",
      "Others",
      "Specify at level 3"
    ],
    "Other Long term liabilities": [
      "Trade payable due to MSME",
      "Trade payable due to Others",
      "Others"
    ],
    "Other non current assets": [
      "Long-term Trade Receivables (Secured, considered good)",
      "Long-term Trade Receivables (Unsecured, considered good)",
      "Long-term Trade Receivables (Doubtful)",
      "Provision for doubtful debts",
      "Security Deposits",
      "Bank Deposit having maturity of greater than 12 months",
      "Others"
    ],
    "Prior Period Item": ["Specify at level 3"],
    "Profit loss from discontinuing operation before tax": ["Specify at level 3"],
    "Property Plant and Equipment": ["Captured from FAR"],
    "Purchases of stock in trade": [
      "Purchases of goods",
      "Other direct expenses",
      "Specify at level 3"
    ],
    "Reserves and surplus": [
      "General Reserve",
      "Revaluation Reserve",
      "Share Options Outstanding Account",
      "MAT Credit Entitlement Reserve",
      "Profit & Loss Account",
      "Other Reserves",
      "Other Reserves 1"
    ],
    "Revenue from operations": [
      "Sale of products",
      "Sale of services",
      "Grants or donations received",
      "Other operating revenues",
      "Others",
      "Specify at level 3"
    ],
    "Share application money pending allotment": ["Specify at level 3"],
    "Share capital": [
      "Issued Equity Share Capital",
      "Preference Share Capital"
    ],
    "Short term borrowings": [
      "Current maturities of long-term debt",
      "Secured Loans repayable on demand from banks",
      "Secured Loans repayable on demand from other parties",
      "Secured Loans and advances from related parties",
      "Secured Deposits",
      "Secured Other loans and advances",
      "Unsecured Loans repayable on demand from banks",
      "Unsecured Loans repayable on demand from other parties",
      "Unsecured Loans and advances from related parties",
      "Unsecured Deposits",
      "Unsecured Other loans and advances"
    ],
    "Short term loans and advances": [
      "Loans and advances to related parties",
      "Loans and advances to employees",
      "Advances to suppliers",
      "Advance Income Tax (Net of provision for taxes)",
      "Balances with Government Authorities",
      "Other loans and advances (Secured, considered good)",
      "Other loans and advances (Unsecured, considered good)",
      "Other loans and advances (Doubtful)",
      "Provision for doubtful advances",
      "Others"
    ],
    "Short term provisions": [
      "Provision for employee benefits",
      "Provision for income tax",
      "Provision for others",
      "Others",
      "Specify at level 3"
    ],
    "Tax Expenses": [
      "Current Tax",
      "Deferred Tax",
      "MAT Credit Entitlement",
      "Prior Period Taxes",
      "Excess/Short Provision Written back/off"
    ],
    "Tax expenses of discontinuing operation": ["Specify at level 3"],
    "Trade payables due to MSME": ["Due to Micro and Small Enterprises"],
    "Trade payables due to others": ["Due to others"],
    "Trade receivables": [
      "Secured considered good",
      "Unsecured considered good",
      "Doubtful",
      "Provision for doubtful debts"
    ]
  }
}
```

---

*Black Horse FS Automation | CE v4.0 | Schedule III — Corporate Entities (India)*

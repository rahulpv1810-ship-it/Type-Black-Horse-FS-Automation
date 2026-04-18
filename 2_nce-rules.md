# NCE-Specific Rules & Master Dictionary — Black Horse v44.0
### Non-Corporate Entities: Partnerships / Proprietorships / LLPs / HUFs / Sole Traders

> Load this file when entity type = **Non-Corporate Entity (NCE)**.
> After loading, apply Rules 1, 4, 6, 7, 8 (NCE variants) first, then return to SKILL.md for common Rules 2, 3, 5, 9, 10, 11.

---

## TABLE OF CONTENTS
1. NCE Rule 1 — Bank OD Reclassification
2. NCE Rule 4 — Tax Integrity (Simplified)
3. NCE Rule 6 — Partnership / Proprietorship P&L Segregation
4. NCE Rule 7 — Party Integrity & Flipped Balances
5. NCE Rule 8 — Fixed Deposit Tenor Classification
6. NCE Intelligence Flags (AI-Enhanced)
7. NCE Sub-Note Tier 2 Mandatory Table
8. NCE Master Dictionary (JSON)

---

## NCE RULE 1 — Bank OD Reclassification

**IF** ledger contains `"OD"`, `"OCC"`, or `"Overdraft"`:

| Balance | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| Debit (+) → Reclassify as ASSET | **Cash and bank balances** | Balances with banks | [Bank Name] OD (Debit Balance) |
| Credit (−) → Retain as LIABILITY | Short term borrowings | Secured Loans repayable on demand from banks *(or Unsecured variant)* | BLANK |

> **Audit Principle:** A bank OD with a debit balance is a cash asset in substance — the entity has deposited more than it drew. For non-corporate entities, the parent face is "Cash and bank balances" (not "Cash and cash equivalents" used in CE).

---

## NCE RULE 4 — Tax Integrity (Debit vs. Credit)

| Scenario | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| Debit taxes *(TDS Receivable, ITC, GST Input Credit, Advance Tax — all years)* | **Short term loans and advances** | Balances with Government Authorities | BLANK |
| Credit taxes *(GST Payable, TDS Payable, PF Payable, PT Payable, ESIC Payable, etc.)* | Other current liabilities | Match exact payable name from NCE dictionary | BLANK |

> **Audit Principle:** Non-corporate entities do not have MAT obligations. All debit tax balances route to short-term loans and advances under "Balances with Government Authorities" — there is no long-term split unlike CE.

---

## NCE RULE 6 — Partnership / Proprietorship P&L Segregation

This is the most critical NCE-specific rule. Apply it to all capital, drawing, and remuneration ledgers.

| Ledger Type | Face Grouping | Note Grouping | Sub-Note | Action |
|---|---|---|---|---|
| Partner Capital Account (all partners) | Owners Capital Account | Fill details in Q_Disc | BLANK | — |
| Partner Drawings Account | Owners Capital Account | Fill details in Q_Disc | BLANK | ⚑ Check drawings vs. capital |
| Proprietor Capital Account | Owners Capital Account | Fill details in Q_Disc | BLANK | — |
| Proprietor Drawings | Owners Capital Account | Fill details in Q_Disc | BLANK | — |
| Goodwill on Admission (credit leg) | Owners Capital Account | Fill details in Q_Disc | BLANK | — |
| Interest on Capital | Finance costs | Interest expense | BLANK | ⚑ Verify ≤ 12% p.a. |
| Partner Salary / Remuneration | Partners remuneration | Fill details in Partners sheet | BLANK | ⚑ Verify Sec 40(b) cap |
| Partner Commission | Partners remuneration | Fill details in Partners sheet | BLANK | ⚑ Verify Sec 40(b) cap |

**HARD RULES:**
- **NEVER** route P&L debit legs (expenses, losses) to Owners Capital Account.
- **NEVER** route partner drawings to any expense head.
- **NEVER** route interest on capital to Owners Capital — it belongs to Finance costs.

> **Audit Principle:** Capital and drawings are equity movements, not P&L items. Routing drawings to expenses would misstate both net profit and the partners' capital position.

### Sec 40(b) Remuneration Limits (Income Tax — Flag If Exceeded)

| Book Profit Slab | Maximum Allowable Remuneration |
|---|---|
| First ₹3,00,000 of book profit (or in case of loss) | ₹1,50,000 or 90% of book profit, whichever is higher |
| On the balance of book profit | 60% of balance |

If actual partner remuneration paid exceeds these limits → raise ⚑ flag for tax audit disallowance under Sec 40(b).

### Interest on Capital Rate Alert
If Interest on Capital > 12% p.a. → raise ⚑ flag for Sec 40(b) disallowance. Verify partnership deed rate.

---

## NCE RULE 7 — Party Integrity & Flipped Balances

Scan **all** ledger names for identifiers: `limited`, `private`, `pvt`, `ltd`, `pharma`, `medical`, `enterprises`, `agencies`, `systems`, `solutions`, `traders`. **NEVER** map these to P&L.

| Scenario | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| Normal Debit (+) balance | Trade receivables | Unsecured considered good | Title-Cased Ledger Name |
| Normal Credit (−) balance | Trade payables due to others | Due to others | Title-Cased Ledger Name |
| FLIPPED — Debit-balance Creditor | Short term loans and advances | **Advances to Creditors** | BLANK |
| FLIPPED — Credit-balance Debtor | Other current liabilities | Advances from Customers | BLANK |

> **Audit Principle:** A vendor balance flipped to Debit is an advance paid in excess of the liability — it is an asset. Note: For NCE, the heading is "Advances to Creditors" (not "Advances to suppliers" used in CE).

---

## NCE RULE 8 — Fixed Deposit Tenor Classification

**IF** ledger contains `"Fixed Deposit"`, `"FD"`, or `"Term Deposit"` AND Debit (+) balance:

| Tenor | Face Grouping | Note Grouping | Sub-Note |
|---|---|---|---|
| ≤ 3 months | **Cash and bank balances** | Bank Deposit having maturity of less than 3 months | BLANK |
| > 3 months and ≤ 12 months | **Cash and bank balances** | Bank Deposit having maturity of greater than 3 months and less than 12 months | BLANK |
| > 12 months | Other non current assets | Bank Deposit having maturity of greater than 12 months | BLANK |
| Unknown / Not Specified | **Cash and bank balances** | Bank Deposit having maturity of greater than 3 months and less than 12 months | **⚑ Raise flag** |

> ⚑ **Flag text:** *FD tenor not confirmed — obtain maturity certificate from client before finalising classification.*

> **Audit Principle:** For NCE, the parent face is "Cash and bank balances" (not "Cash and cash equivalents"). Tenor determines current vs. non-current classification — not the account name.

---

## NCE INTELLIGENCE FLAGS (AI-Enhanced)

Apply these during Phase 1 in addition to all base rules:

**⚑ Partner Remuneration Cap — Sec 40(b)**
If partner remuneration/salary/commission ledger found → flag for Sec 40(b) cap. Apply the slab table from Rule 6. Obtain partnership deed to verify authorized remuneration.

**⚑ Interest on Capital Rate — Sec 40(b)**
If Interest on Capital ledger found → flag for 12% p.a. ceiling. If rate not specified → obtain partnership deed.

**⚑ MSME Creditor Alert**
If any Trade Payable mentions "MSME", "Micro", or "Small Enterprise" → flag for Sec 43B(h) — 45-day payment rule. Confirm payment dates vs. invoice dates.

**⚑ Drawings vs. Capital Check**
If Drawings debit is large relative to the capital balance → flag for negative capital risk. A negative capital account signals over-withdrawal and must be disclosed.

**⚑ Capital Account Negative Balance**
If any partner's capital account shows a net debit (asset) balance after drawings → flag immediately. This signals over-drawings exceeding capital contribution.

**⚑ Salary to Non-Working Partner**
If remuneration is paid to a partner not actively involved in business → flag for Sec 40(b) — only working partners qualify.

**⚑ Unbalanced TB**
If Debit ≠ Credit total → stop and flag. Do not map an unbalanced TB without explicit user confirmation.

---

## NCE SUB-NOTE TIER 2 — MANDATORY TABLE

For all rows below, Sub-Note = **Title-Cased Ledger Name** (Black Horse rejects blank in these slots).

| Face Grouping | Note Grouping |
|---|---|
| Cash and bank balances | Others |
| Current investments | Other investments |
| Deferred tax assets net | Specify at level 3 |
| Deferred tax liabilities Net | Specify at level 3 |
| Depreciation and amortization expenses | Specify at level 3 |
| Exceptional item | Specify at level 3 |
| Extraordinary Item | Specify at level 3 |
| Long term borrowings | Secured Other loans and advances |
| Long term borrowings | Unsecured Other loans and advances |
| Long term loans and advances | Other loans and advances (Secured, considered good) |
| Long term loans and advances | Other loans and advances (Unsecured, considered good) |
| Long term loans and advances | Other loans and advances (Doubtful) |
| Long term loans and advances | Others |
| Long term provisions | Others |
| Long term provisions | Specify at level 3 |
| Non current investments | Other non-current investments |
| Other current assets | Specify at level 3 |

---

## NCE MASTER DICTIONARY (JSON) — Black Horse v44.0

```json
{
  "bh_nce_mappings": {
    "Capital work in progress": ["Captured in Notes-2"],
    "Cash and bank balances": [
      "Balances with banks",
      "Cheques, drafts on hand",
      "Cash on hand",
      "Bank Deposit having maturity of less than 3 months",
      "Bank Deposit having maturity of greater than 3 months and less than 12 months",
      "Others"
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
    "Depreciation and amortization expenses": ["Specify at level 3"],
    "Direct Expenses": ["Specify at level 3"],
    "Employee benefit expenses": [
      "Salaries and wages",
      "Contribution to provident and other funds",
      "Expense on ESOP and ESPP",
      "Staff welfare expenses"
    ],
    "Finance costs": [
      "Interest expense",
      "Other borrowing costs",
      "Applicable net gain/loss on foreign currency transactions and translation"
    ],
    "Intangible assets": ["Captured from FAR"],
    "Inventories": ["Captured from Q_Disc"],
    "Long term borrowings": [
      "Secured Term loans from banks",
      "Secured Term loans from other parties",
      "Secured Deferred payment liabilities",
      "Secured Loans and advances from related parties",
      "Secured Long term maturities of finance lease obligations",
      "Secured Other loans and advances",
      "Unsecured Term loans from banks",
      "Unsecured Term loans from other parties",
      "Unsecured Deferred payment liabilities",
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
      "Others",
      "Specify at level 3"
    ],
    "Non current investments": [
      "Investment property",
      "Investment in Fixed Deposits",
      "Other non-current investments",
      "Specify at level 3"
    ],
    "Other current assets": [
      "Advances to Creditors",
      "Prepaid Expenses",
      "Interest Receivable",
      "Rent Receivable",
      "Other Income Receivable",
      "Specify at level 3"
    ],
    "Other current liabilities": [
      "Current maturities of finance lease obligations",
      "Interest accrued but not due on borrowings",
      "Interest accrued and due on borrowings",
      "Income received in advance",
      "Advances from Customers",
      "Income Tax Payable",
      "GST Payable",
      "TDS Payable",
      "TCS Payable",
      "Provident Fund Payable",
      "ESIC Payable",
      "Professional Tax Payable",
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
      "Rent Income",
      "Others",
      "Specify at level 3"
    ],
    "Other Long term liabilities": [
      "Trade payable",
      "Others",
      "Specify at level 3"
    ],
    "Other non current assets": [
      "Long-term Trade Receivables (Secured, considered good)",
      "Long-term Trade Receivables (Unsecured, considered good)",
      "Long-term Trade Receivables (Doubtful)",
      "Provision for doubtful debts",
      "Security Deposits",
      "Bank Deposit having maturity of greater than 12 months",
      "Others",
      "Specify at level 3"
    ],
    "Owners Capital Account": ["Fill details in Q_Disc"],
    "Partners remuneration": ["Fill details in Partners sheet"],
    "Prior Period Item": ["Specify at level 3"],
    "Property Plant and Equipment": ["Captured from FAR"],
    "Purchases of stock in trade": ["Specify at level 3"],
    "Reserves and surplus": [
      "Capital Reserves",
      "General Reserve",
      "Revaluation Reserve",
      "Profit & Loss Account",
      "Other Reserves"
    ],
    "Revenue from operations": [
      "Sale of products",
      "Sale of services",
      "Grants or donations received",
      "Other operating revenues",
      "Others"
    ],
    "Short term borrowings": [
      "Current maturities of long-term debt",
      "Secured Loans repayable on demand from banks",
      "Secured Loans repayable on demand from other parties",
      "Secured Loans and advances from related parties",
      "Secured Other loans and advances",
      "Unsecured Loans repayable on demand from banks",
      "Unsecured Loans repayable on demand from other parties",
      "Unsecured Loans and advances from related parties",
      "Unsecured Other loans and advances"
    ],
    "Short term loans and advances": [
      "Loans and advances to related parties",
      "Advance Income Tax (Net of provision for taxes)",
      "Balances with Government Authorities",
      "Advances to Creditors",
      "Advances to Employees",
      "Other loans and advances (Secured, considered good)",
      "Other loans and advances (Unsecured, considered good)",
      "Other loans and advances (Doubtful)",
      "Provision for doubtful advances",
      "Others"
    ],
    "Short term provisions": [
      "Provision for employee benefits",
      "Provision for Income tax",
      "Other provisions",
      "Specify at level 3"
    ],
    "Tax Expenses": [
      "Current Tax",
      "Deferred Tax",
      "AMT Credit Entitlement",
      "Prior Period Taxes",
      "Excess/Short Provision Written back/off"
    ],
    "Trade payables due to MSME": ["Due to Micro, Small and Medium Enterprises"],
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

*Black Horse FS Automation | NCE v44.0 | Non-Corporate Entities (Partnerships / Proprietorships / LLPs / HUFs)*

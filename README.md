# CARBON CREDIT RISK ENGINE — ACTUS What-If-3
## GreenCell Industries, Chennai → United States
### Live Excel Dashboard + ACTUS Risk Engine on AWS

---

## 1. BUSINESS PROBLEM

**John** runs GreenCell Industries in Chennai, India — a battery manufacturing company.
He holds **10,000 verified carbon credits** (Verra VCS registry, Bureau Veritas certified)
worth **USD 420,000** at the initial carbon price of **USD 42 per credit**.

He takes a **USD 280,000 loan** from EcoBank against these credits as collateral.
This funds a factory upgrade to increase battery production capacity.

### The Risk

A VCM (Voluntary Carbon Market) quality crisis hits in Q2 2026 — an overcrediting study
(BeZero Carbon, Q1 2026) causes the carbon price to crash from **USD 42 → USD 28**.

At USD 28 per credit:
- Collateral value = 10,000 × 28 = **USD 280,000**
- Loan principal = **USD 280,000**
- LTV = **100%** → EcoBank issues a **MARGIN CALL**
- John must repay immediately or provide additional collateral
- **Factory upgrade is frozen. Working capital locked.**

### The Question

What hedging strategy protects John best?

---

## 2. THREE SCENARIOS

| Scenario | Strategy | Protection | Key Instrument |
|---|---|---|---|
| A | No Hedge | None | Loan only — full carbon price exposure |
| B | Loan + Swap | Price hedge | PAM + SWAPS contract — EcoBank pays when price falls |
| C | Loan + Swap + Insurance | Full stack | PAM + SWAPS + PAM insurance — registry cancellation covered |

---

## 3. SYSTEM ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────┐
│                    EXCEL WORKBOOK                           │
│                CARBON-ACTUS-Pro.xlsm                        │
│                                                             │
│  ┌─────────────┐    ┌──────────────┐    ┌────────────────┐ │
│  │ ⚡ Inputs   │    │ ⚙ Contracts  │    │ 📡 Response    │ │
│  │             │    │              │    │                │ │
│  │ Column C:   │    │ Shows all 9  │    │ Live ACTUS     │ │
│  │ User inputs │    │ contracts    │    │ cashflow       │ │
│  │             │    │ auto-built   │    │ events from    │ │
│  │ Column H-I: │    │ from inputs  │    │ Power Query    │ │
│  │ ACTUSInputs │    │              │    │                │ │
│  │ Table       │    │ [SEND BUTTON]│    │ A: rows 7-12   │ │
│  │ (linked to  │    │              │    │ B: rows 16-38  │ │
│  │ column C)   │    │              │    │ C: rows 42-70  │ │
│  └──────┬──────┘    └──────┬───────┘    └───────┬────────┘ │
│         │                  │                    │           │
│         │     ┌────────────┘                    │           │
│         │     │                                 │           │
│         ▼     ▼                                 │           │
│  ┌─────────────────────────┐                    │           │
│  │   Power Query (M Code)  │                    │           │
│  │                         │                    │           │
│  │  Carbon_A (6 rows)      │────────────────────┘           │
│  │  Carbon_B (22 rows)     │   loads to Response sheet      │
│  │  Carbon_C (28 rows)     │                                │
│  │                         │                                │
│  │  Reads ACTUSInputs →    │                                │
│  │  Builds JSON contracts →│                                │
│  │  HTTP POST to ACTUS →   │                                │
│  │  Parses JSON response   │                                │
│  └────────────┬────────────┘                                │
│               │                                             │
└───────────────┼─────────────────────────────────────────────┘
                │ HTTP POST
                ▼
┌───────────────────────────────────┐
│   ACTUS RISK ENGINE               │
│   AWS EC2 — Public IP             │
│   http://34.203.247.32:8083       │
│   /eventsBatch                    │
│                                   │
│   Spring Boot Java application    │
│   ACTUS standard algorithms       │
│   Computes: IED, RR, IP, MD events│
└───────────────────────────────────┘
```

---

## 4. HOW THE DYNAMIC LOOP WORKS

### Step-by-step when you change an input:

```
1. User changes a value in the ⚡ Inputs sheet
   Example: price_stress from 28 → 15

2. ACTUSInputs table automatically updates
   Because Value column is linked to the input cells by formula

3. User clicks ▶ SEND TO ACTUS ENGINE button on ⚙ Contracts sheet

4. VBA macro runs one line: ThisWorkbook.RefreshAll

5. Power Query Carbon_A, Carbon_B, Carbon_C all trigger simultaneously

6. Each query reads from ACTUSInputs table:
   credits       = 10,000
   price_initial = 42
   price_stress  = 15  ← new value
   loan_notional = 280,000
   etc.

7. Power Query computes derived values:
   RatioQ2 = price_stress / price_initial = 15/42 = 0.3571
   SwapNotional = swap_fixed × credits = 38 × 10,000 = 380,000
   FixedLegRate = swap_fixed / price_initial = 38/42 = 0.9048

8. Power Query builds JSON contract payloads

9. HTTP POST to http://34.203.247.32:8083/eventsBatch

10. ACTUS engine computes all cashflow events mathematically

11. JSON response parsed and loaded to 📡 Response sheet

12. 📊 Insights sheet SUMIF formulas recalculate automatically
    showing new net costs for all 3 scenarios
```

---

## 5. ACTUS INPUT TABLE (ACTUSInputs)

Located at **⚡ Inputs sheet, columns H-I**. Each cell in the Value column
is a formula linked to the main input column C.

| Name | Description | Current Value |
|---|---|---|
| credits | Carbon Credits Held | 10,000 |
| price_initial | Carbon Price at origination | $42.00 |
| price_stress | Carbon Price at VCM crisis | $28.00 |
| price_recovery | Carbon Price at recovery | $35.00 |
| loan_notional | Loan Principal | $280,000 |
| loan_start | Loan Start Date | 2026-01-01 |
| loan_maturity | Loan Maturity Date | 2027-01-01 |
| swap_fixed | Swap Fixed Price per credit | $38.00 |
| insurance_rate | Insurance Premium Rate | 2% |
| loan_rate | Loan Interest Rate | 7% |

**Any change to the main inputs automatically flows into Power Query.**

---

## 6. ACTUS CONTRACT DEFINITIONS

### Scenario A — No Hedge

One contract only:

```json
{
  "contractType": "PAM",
  "contractID": "carbonLOAN01-A",
  "contractRole": "RPA",
  "notionalPrincipal": 280000,
  "nominalInterestRate": 0.07,
  "cycleOfInterestPayment": "P3ML1",
  "dayCountConvention": "30E360",
  "maturityDate": "2027-01-01T00:00:00"
}
```

Risk factor: `CARBON_PRICE_RATIO` — tracks carbon price quarterly.
John has no hedge — if price falls, only the loan interest applies but margin call risk is live.

---

### Scenario B — Loan + Swap

Three contracts:

**1. carbonLOAN01-B (PAM)**
- $280,000 at 7% annually, quarterly payments

**2. carbonSWAP-B (SWAPS wrapper)**
- `deliverySettlement: "D"` — cash settled
- Contains two PAM legs:

**3a. carbonSWAP-B-FIXED (PAM — Fixed Leg)**
- Notional: $38 × 10,000 = **$380,000**
- Rate: 38/42 = **0.9048** (John pays fixed)
- John locks in USD 38 per credit permanently

**3b. carbonSWAP-B-FLOAT (PAM — Float Leg)**
- Same notional: $380,000
- Rate resets quarterly via `CARBON_PRICE_RATIO` risk factor
- EcoBank pays the floating carbon market price ratio

**How the swap works at stress:**
- Carbon falls to $28. Ratio = 28/42 = 0.6667
- ACTUS computes both legs and nets them via SWAPS wrapper
- Net effect: EcoBank pays John (38-28) × 10,000 = **$100,000**
- Margin call prevented — effective collateral restored to $380,000

---

### Scenario C — Loan + Swap + Insurance

All contracts from Scenario B plus:

**carbonINS_01-C (PAM)**
- `contractRole: "BUY"` — insurance buyer
- Notional: $42 × 10,000 = **$420,000** (full credit value)
- Rate: 2% per year → $8,400/year → $2,100/quarter
- Risk factor: `CARBON_DELIVERY_INDEX`
- Triggers payout if registry cancels credits (index < 0.85)
- In base scenario: index stays at 1.0 (no cancellation event)

---

## 7. CARBON PRICE PATH — RISK FACTOR

The risk factor `CARBON_PRICE_RATIO` is sent as a quarterly time series.
Values are **ratios** relative to the initial price ($42 = base 1.0).

| Quarter | Date | Price | Ratio | Event |
|---|---|---|---|---|
| Q0 | 2026-01-01 | $42.00 | 1.0000 | Loan start |
| Q1 | 2026-04-01 | $42.00 | 1.0000 | Price stable |
| Q2 | 2026-07-01 | $28.00 | 0.6667 | ⚠ STRESS — Margin Call |
| Q3 | 2026-10-01 | $28.00 | 0.6667 | ⚠ STRESS — Sustained |
| Q4 | 2027-01-01 | $35.00 | 0.8333 | Recovery |

**Formula:** `RatioQ2 = price_stress / price_initial`

Changing `price_stress` in the Inputs sheet immediately changes the
risk factor path sent to ACTUS.

---

## 8. ACTUS EVENT TYPES RETURNED

| Event | Meaning | Example |
|---|---|---|
| IED | Initial Exchange Date — contract starts | -280,000 (loan disbursed) |
| IP | Interest Payment — quarterly cash flow | +4,900 (7% × 280k ÷ 4) |
| RR | Rate Reset — floating rate resets | 0 payoff, new rate recorded |
| MD | Maturity Date — contract ends | +280,000 (principal returned) |

---

## 9. POWER QUERY M CODE STRUCTURE

Each of the 3 queries follows the same structure:

```
1. Read ACTUSInputs table from Excel
   T = Excel.CurrentWorkbook(){[Name="ACTUSInputs"]}[Content]

2. Helper function to look up values by name
   GetVal = (n) => Table.SelectRows(T, each [Name]=n){0}[Value]

3. Read all 10 inputs
   Credits = GetVal("credits"), etc.

4. Compute dates
   DealDate = loan_start - 1 day
   FirstPay = loan_start + 3 months

5. Compute ratios for risk factor path
   RatioQ2 = price_stress / price_initial

6. Build JSON string for contracts
   LoanJson = "{...notionalPrincipal: 280000...}"
   SwapJson = "{...contractStructure: [FixedLeg, FloatLeg]...}"

7. HTTP POST to ACTUS
   Web.Contents("http://34.203.247.32:8083/eventsBatch", ...)

8. Parse JSON response
   Json.Document(Raw)

9. Expand events into table rows
   Table.ExpandRecordColumn(...)

10. Load to 📡 Response sheet
```

---

## 10. INSIGHTS FORMULAS

The 📊 Insights sheet uses SUMIF/SUMIFS to read from the Power Query tables:

```excel
Scenario A Loan Interest:
=SUMIF(Carbon_A[EventType],"IP",Carbon_A[Payoff])

Scenario B Net Cost (Loan + Swap net):
=SUMIFS(Carbon_B[Payoff],Carbon_B[ContractID],"carbonLOAN01-B",Carbon_B[EventType],"IP")
+SUMIFS(Carbon_B[Payoff],Carbon_B[ContractID],"carbonSWAP-B",Carbon_B[EventType],"IP")

Scenario C Net Cost (Loan + Swap + Insurance):
=SUMIFS(Carbon_C[Payoff],Carbon_C[ContractID],"carbonLOAN01-C",Carbon_C[EventType],"IP")
+SUMIFS(Carbon_C[Payoff],Carbon_C[ContractID],"carbonSWAP-C",Carbon_C[EventType],"IP")
+SUMIFS(Carbon_C[Payoff],Carbon_C[ContractID],"carbonINS_01-C",Carbon_C[EventType],"IP")
```

When Power Query refreshes, these formulas automatically recalculate.

---

## 11. FILE STRUCTURE

```
carbon-contracts/
├── CARBON-ACTUS-Pro.xlsm           ← Main Excel workbook (macro-enabled)
├── dashA.json                       ← Original Postman collection (3 scenarios)
├── CARBON-GREENCELL-3SCENARIO.json  ← Full Postman collection
├── README-CARBON-ACTUS.md           ← Technical reference (this file)
├── README-CARBON-OVERVIEW.md        ← Plain English overview
│
└── powerquery/
    ├── carbon_A_table.pq            ← Power Query M code — Scenario A
    ├── carbon_B_table.pq            ← Power Query M code — Scenario B
    ├── carbon_C_table.pq            ← Power Query M code — Scenario C
    └── AddACTUSButton.vba           ← VBA button creator
```

---

## 12. INFRASTRUCTURE

| Component | Details |
|---|---|
| ACTUS Engine | Spring Boot Java — actus-server-rf20 |
| AWS EC2 Public IP | 34.203.247.32 |
| Port | 8083 |
| Endpoint | POST /eventsBatch |
| Docker Image | actus-server-rf20-custom:latest |
| Local Port | 127.0.0.1:8083 (requires Docker running) |
| Registry | Verra VCS — Voluntary Carbon Market |

### Start Local Docker (for Postman testing):
```bash
cd C:\CHAINAIM3003\mcp-servers\actus-risk-service-extension1\actus-docker-networks
docker compose -f quickstart-docker-actus-rf20-public.yml up -d
```

### Test AWS endpoint:
```bash
curl http://34.203.247.32:8083/eventsBatch \
  -X POST \
  -H "Content-Type: application/json" \
  -d @results/public_test.json
```

---

## 13. SETTING UP THE EXCEL WORKBOOK

Follow these steps **once** after opening the file on a new PC.

---

### Step 1 — Open and Enable the Workbook

1. Open `CARBON-ACTUS-Pro.xlsm` in Excel
2. If a yellow **Security Warning** bar appears, click **Enable Content**
3. If a **Protected View** bar appears, click **Enable Editing**
4. If Excel asks about macros, select **Enable All Macros**

> ⚠️ The workbook must be fully enabled before proceeding.

---

### Step 2 — Fix the Power Query Privacy Setting

This step allows Power Query to read Excel inputs AND call the ACTUS server
in the same query. Without this, Excel blocks the connection with a firewall error.

1. Go to **Data tab** in the Excel ribbon
2. Click **Get Data** → **Query Options**
3. In the left panel click **Privacy**
4. Select **"Always ignore Privacy Level settings"**
5. Click **OK**

> ✅ This only needs to be done once per machine.

---

### Step 3 — Connect Power Query Tables to the Response Sheet

This step loads ACTUS query results into the correct location in the Response sheet.

1. Go to **Data tab** → click **Queries & Connections**
   (a panel opens on the right showing Carbon_A, Carbon_B, Carbon_C)

**For Carbon_A:**
- Click the **📡 Response** sheet tab at the bottom of the workbook
- Click on cell **A7**
- Go back to **Data → Queries & Connections**
- Right-click **Carbon_A** → **Load To**
- Select **Table** and **Existing worksheet**
- Click **OK** → click **OK** on the data loss warning

**For Carbon_B:**
- Click the **📡 Response** sheet tab
- Click on cell **A16**
- Right-click **Carbon_B** → **Load To** → Table → Existing worksheet → OK → OK

**For Carbon_C:**
- Click the **📡 Response** sheet tab
- Click on cell **A42**
- Right-click **Carbon_C** → **Load To** → Table → Existing worksheet → OK → OK

> ✅ The three queries are now connected to the Response sheet.

---

### Step 4 — Open the VBA Editor

Press **`Alt + F11`** to open the Visual Basic for Applications editor.

---

### Step 5 — Insert a New Module

1. In the **Project panel** on the left, find your workbook
2. **Right-click** on the project name
3. Select **Insert → Module**
4. A new empty module opens on the right

---

### Step 6 — Paste the VBA Code

Click inside the empty code window and paste the following:

```vba
Sub AddACTUSButton()
    Dim ws As Worksheet
    Set ws = ThisWorkbook.Sheets(2)
    Dim shp As Shape
    For Each shp In ws.Shapes
        shp.Delete
    Next shp
    Dim btn As Object
    Set btn = ws.Buttons.Add(5, 339, 250, 40)
    btn.Caption = "SEND TO ACTUS ENGINE"
    btn.OnAction = "SendToACTUS"
    btn.Font.Name = "Calibri"
    btn.Font.Size = 12
    btn.Font.Bold = True
    MsgBox "Done!", vbInformation
End Sub

Sub SendToACTUS()
    ThisWorkbook.RefreshAll
    MsgBox "Done! Check Response sheet.", vbInformation, "ACTUS"
End Sub

Sub ClearResponse()
    ThisWorkbook.Sheets(3).Rows("7:100").ClearContents
    MsgBox "Cleared.", vbInformation, "ACTUS"
End Sub
```

---

### Step 7 — Run the Setup Macro

1. Press **`Alt + F5`** (or go to **Run → Run Macro**)
2. Select **`AddACTUSButton`**
3. Click **Run**
4. A confirmation message **"Done!"** appears — click OK

> ✅ The SEND TO ACTUS ENGINE button is now installed on the ⚙ Contracts sheet.
> You only need to run this setup once.

---

### Step 8 — Save the Workbook

Press **`Ctrl + S`**. If Excel asks about file format,
choose **Keep Current Format** to preserve macros as `.xlsm`.

---

## 14. HOW TO USE THE EXCEL WORKBOOK

1. Open `CARBON-ACTUS-Pro.xlsm` — enable macros when prompted

2. Go to **⚡ Inputs sheet** — change any value:
   - Carbon Credits Held
   - Carbon Price — Initial / Stress / Recovery
   - Loan Principal
   - Swap Fixed Price
   - Insurance Premium Rate

3. Go to **⚙ Contracts sheet** — review all 9 contracts
   (values auto-update from inputs)

4. Click **SEND TO ACTUS ENGINE** button

5. Wait ~10 seconds for all 3 queries to complete

6. Go to **📡 Response sheet** — see real ACTUS cashflow events

7. Go to **📊 Insights sheet** — see updated scenario comparison

---

## 15. WHAT MAKES THIS SIGNIFICANT

### Traditional approach:
- Analyst builds a spreadsheet with hardcoded formulas
- Scenario comparison done manually
- No audit trail
- Error-prone, time-consuming, non-standard

### This system:
- **ACTUS standard** — same algorithms used by central banks and regulators worldwide
- **Live engine on AWS** — not a simulation, real mathematical computation
- **Dynamic contracts** — change any input, contracts rebuild automatically
- **Full audit trail** — every cashflow event has type, date, amount, rate
- **Three scenarios in parallel** — computed simultaneously in one button click
- **Zero hardcoding** — all values flow from inputs through to ACTUS

### The key insight:
When carbon price falls to $28, ACTUS automatically fires the rate reset event (RR)
on the floating leg of the swap. The new rate 0.6667 causes the IP events to change.
The swap net payment flows to John automatically. No analyst needed. No dispute possible.
The contract algorithm does exactly what the legal agreement says — every time.

---

## 16. SCENARIO RESULTS (Base Inputs)

| | Scenario A | Scenario B | Scenario C |
|---|---|---|---|
| Strategy | No Hedge | Loan + Swap | Full Stack |
| Loan Interest | $19,600 | $19,600 | $19,600 |
| Swap Net Gain | $0 | ~$57,500 | ~$57,500 |
| Insurance Premium | $0 | $0 | $8,400 |
| **Net Cost** | **$19,600** | **~($37,900)** | **~($49,100)** |
| Margin Call Risk | HIGH ⚠ | NONE ✅ | NONE ✅ |
| Registry Protection | None | None | Full ✅ |

**Winner: Scenario B** — Lowest net cost, margin call prevented,
factory upgrade proceeds without interruption.

---

*Built with ACTUS Risk Engine · AWS EC2 · Excel Power Query · VBA*
*GreenCell Industries Carbon Credit Risk Model · March 2026*

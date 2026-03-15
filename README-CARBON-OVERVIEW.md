# Carbon Credit Risk Engine — Overview
## GreenCell Industries · Chennai, India → United States
### Built on ACTUS Standard · Powered by AWS

---

## The Story

John runs GreenCell Industries in Chennai — a battery manufacturing company that has earned
**10,000 verified carbon credits** through its green manufacturing process. These credits are
registered on the Verra VCS registry and certified by Bureau Veritas India.

At a carbon market price of **USD 42 per credit**, John's credit portfolio is worth **USD 420,000**.
He approaches EcoBank for a **USD 280,000 loan** — using his carbon credits as collateral —
to fund a critical factory upgrade that will double production capacity.

Everything looks good. LTV is 67%. The loan is approved.

---

## The Problem

Three months after the loan closes, a bombshell report drops.

BeZero Carbon publishes a study showing widespread overcrediting in the Voluntary Carbon Market.
The carbon price crashes. Fast.

**USD 42 → USD 28 per credit.**

At USD 28, John's 10,000 credits are worth exactly USD 280,000 — equal to the loan.
LTV hits **100%**. EcoBank's risk system fires automatically.

**Margin call issued.**

John must either repay the loan immediately or deposit additional collateral.
His factory upgrade funding is frozen. The machinery he ordered sits unpaid.
Working capital is locked while the market panic plays out.

This is the risk that ACTUS is designed to model — and hedge against.

---

## What Is ACTUS?

ACTUS stands for **Algorithmic Contract Types Unified Standards**.

It is an international open standard that defines financial contracts as pure mathematical
algorithms. Instead of a bank analyst building a custom spreadsheet for each deal, ACTUS
encodes the legal terms of a contract — interest rate, payment dates, notional, day count
convention — into a deterministic algorithm that computes every future cashflow event.

The same ACTUS engine is used by:
- Central banks for systemic risk modelling
- Regulators for portfolio stress testing
- Commercial banks for contract analytics

The engine used in this project runs on AWS and accepts contracts via HTTP POST.
It returns a complete list of cashflow events — IED, IP, RR, MD — computed mathematically
from the contract terms and market risk factors provided.

**No analyst assumptions. No spreadsheet formulas. Pure contract law encoded as algorithms.**

---

## The Three Strategies

This system models three different approaches John can take to protect himself:

### Strategy A — No Hedge

John takes the loan and does nothing else. He is fully exposed to carbon price movements.
If the price falls below USD 28, EcoBank calls the loan. John loses his upgrade funding.

This is the baseline — the worst case that quantifies exactly how much the hedge is worth.

### Strategy B — Loan + Carbon Price Swap

John enters a **carbon price swap** with EcoBank alongside his loan.

The swap works like this:
- John agrees to pay a **fixed price of USD 38 per credit** every quarter
- EcoBank agrees to pay the **floating market price** each quarter
- If the market price falls below USD 38, EcoBank pays John the difference

When the carbon price crashes to USD 28:
- The difference is USD 10 per credit
- With 10,000 credits, EcoBank pays John **USD 100,000**
- This payment offsets the collateral shortfall
- LTV drops back below 100%
- **Margin call is prevented automatically**

The ACTUS SWAPS contract models this precisely — two PAM legs, one fixed and one floating,
with the floating leg resetting quarterly based on the CARBON_PRICE_RATIO risk factor.

### Strategy C — Loan + Swap + Carbon Insurance

John adds a third layer on top of Strategy B.

The insurance contract covers a risk that the swap does not — **registry cancellation**.
If Verra revokes John's credits due to a forest fire, fraud investigation, or methodology
change, his collateral disappears entirely. No swap payment can fix that.

The insurance contract pays 2% of the total credit value per year as a premium.
In return, if the CARBON_DELIVERY_INDEX falls below 0.85 (meaning more than 15% of credits
are cancelled), the insurance triggers and pays out the shortfall.

In the base scenario, no cancellation event occurs. John pays the premium but does not
collect. However, he sleeps easier knowing the factory upgrade is safe from registry risk too.

---

## How the System Works

The entire system is built in a single Excel workbook connected to a live ACTUS engine on AWS.

### The Input Layer

The user enters market parameters and loan details into the Inputs sheet. These include
the initial carbon price, the stress price after the VCM crisis, the recovery price,
the loan amount, the swap fixed rate, the insurance premium rate, and the loan dates.

Every input the user enters flows automatically into a structured table that Power Query
reads when building the ACTUS contracts.

### The Contract Layer

The Contracts sheet shows all nine ACTUS contracts across the three scenarios — three
contracts for Scenario A, four for Scenario B, five for Scenario C. These contracts
display the values derived from the inputs so the user can review them before sending.

When the user clicks the **SEND TO ACTUS ENGINE** button, three Power Query connections
fire simultaneously. Each connection reads the current input values, constructs the JSON
contract payload from scratch, and sends it to the ACTUS engine via HTTP POST.

### The ACTUS Engine

The engine runs on an AWS EC2 instance at a public IP address on port 8083. It accepts
a JSON payload containing the contract definitions and the risk factor time series, and
returns a complete set of cashflow events for every contract in the portfolio.

For Scenario C, that means 28 separate events — loan disbursement, quarterly interest
payments, swap rate resets, swap net payments, insurance premiums, and maturities — all
computed in under two seconds.

### The Response Layer

The cashflow events returned by ACTUS load directly into the Response sheet. Each event
shows its type, date, payoff amount, currency, nominal value, and rate. The Response sheet
is populated fresh every time the button is clicked — reflecting whatever inputs the user
has entered.

### The Insights Layer

The Insights sheet reads from the Power Query tables using formulas. It automatically
computes the total loan interest, swap net gain, insurance premium, and net cost for each
scenario. When the user changes an input and refreshes, all the Insights numbers update
to reflect the new ACTUS response.

---

## The Carbon Price Path

ACTUS needs to know how the carbon price moves over time in order to compute the floating
leg resets on the swap contract. This is provided as a **risk factor time series** called
CARBON_PRICE_RATIO.

The values are expressed as ratios relative to the initial price. An initial price of USD 42
is the base, so ratio 1.0 means the price is at USD 42. When the price crashes to USD 28,
the ratio becomes 0.6667. When it recovers to USD 35, the ratio is 0.8333.

This ratio path is computed dynamically from the user's inputs. If the user changes the
stress price from USD 28 to USD 15, the ratio at Q2 becomes 0.3571, and the ACTUS engine
will compute a much larger swap payment at that quarter — reflecting the deeper crisis.

The path covers five quarterly points from loan start to loan maturity:
- Q0 at loan start — price holds at initial level
- Q1 three months later — price still holds
- Q2 six months in — crisis hits, price crashes to stress level
- Q3 nine months in — price stays at stress level, market uncertain
- Q4 at maturity — partial recovery, market stabilises

---

## What Changes When You Change an Input

The power of this system is that nothing is hardcoded. Every number in every contract
comes from the input values the user provides.

Change the loan amount and the notional principal in all loan contracts changes.
Change the swap fixed rate and the fixed leg rate recalculates, changing the breakeven
point at which EcoBank begins paying John.
Change the stress price and the CARBON_PRICE_RATIO at Q2 and Q3 changes, changing the
floating leg resets and therefore all swap net payment events.
Change the number of credits and the swap notional changes — because the swap covers
the full value of the credit portfolio.

Every change flows through in one click. The user does not need to understand JSON,
the ACTUS standard, or financial derivatives. They change a number, click the button,
and see the real computed response from a live financial risk engine.

---

## Why This Matters

Carbon credit lending is a fast-growing but poorly understood market. Banks are beginning
to offer collateralised lending against carbon portfolios, but the tools for modelling
these risks are primitive. Most deals are still analysed with Excel spreadsheets built
by hand for each client.

The risks are real:
- Carbon prices are volatile — the voluntary market has no price floor
- Registry quality varies — overcrediting and cancellation events are documented
- Margin call mechanics are not well understood by borrowers
- Swap structures for carbon are novel and not standardised

This system demonstrates that ACTUS — designed for traditional financial contracts like
loans and interest rate swaps — can be extended to model carbon credit instruments by
treating the carbon price ratio as a floating rate risk factor.

The same engine that computes swap cashflows for a USD interest rate swap computes them
for a carbon price swap. The mathematics is identical. The contract structure is the same.
Only the underlying risk factor changes — from SOFR to the carbon market price.

This is the fundamental insight: **carbon credit risk instruments are financial contracts,
and financial contracts can be modelled with financial contract standards.**

ACTUS makes the invisible visible. Every cashflow, every rate reset, every margin trigger —
computed, timestamped, auditable, and reproducible. Not by an analyst with a spreadsheet.
By an algorithm that does exactly what the contract says, every time.

---

## Technical Stack

The system uses Excel as the user interface, Power Query as the integration layer,
and ACTUS as the computation engine. The ACTUS server runs as a Docker container
on AWS EC2, exposed on a public IP so the Excel workbook can reach it from anywhere
with an internet connection.

Power Query M code handles the translation between Excel inputs and ACTUS JSON format.
Three separate queries run in parallel — one per scenario — each building its own
complete payload and parsing its own response independently.

A VBA macro provides the one-click button experience. It contains a single line of code
that triggers the Power Query refresh. All the intelligence is in the M code and ACTUS.
The VBA is just the button.

---

*GreenCell Industries Carbon Credit Risk Model*
*ACTUS Risk Engine · AWS EC2 · Excel Power Query*
*March 2026*

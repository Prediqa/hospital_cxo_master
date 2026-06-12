# PREDIQA — Hospital CXO Master

**The Healthcare Intelligence Fabric · Architecture, Loop & Governance Reference**

> Confidential — Authorized technical & executive review only · v1.0 · 2026
> Engines: COFLER · FLIP · EMPATHIFY · FRAUDIQ

---

## Overview

PREDIQA sits **above** your existing systems — and never replaces them.

It is a read-and-correlate intelligence layer over your existing EHR, billing, lab, pharmacy, HR, and payer feeds. Four engines run on one loop, with **seven human gates where your experts own every boundary**. This document is the technical and governance reference for your leadership team.

| Property | Detail |
|---|---|
| Integration model | Read-only · no rip-and-replace |
| EHR targets | Epic (FHIR R4) · Cerner (SMART) · Athenahealth · ABDM |
| Compliance | SOC 2 Type II · HIPAA BAA · GDPR · DPDPA · NHS DSPT |
| Human oversight | 7 mandatory validation gates |
| Markets | US · UK · India · GCC · APAC (one architecture) |

---

## 1. The Four Engines — Who Owns Them

PREDIQA gives every part of your organization its own engine, feeding one shared intelligence loop. The leadership team sees the whole; each function gets depth in its own domain.

| Engine | Role | Owner | What it does |
|---|---|---|---|
| **COFLER** | Oversight | C-Suite | Six-lever command intelligence. Live signals to leadership, what needs attention, and how one event impacts the others. |
| **FLIP** | Provider · RCM | CFO / RCM | Matches clinical notes to codes and documentation, pre-submission. Stops denials before they ship. |
| **EMPATHIFY** | Human Signal | Quality / HR / CMO | The true journey of every human — patient, family, and staff. Live, passive, direct. |
| **FRAUDIQ** | Payer · Integrity | Payer-facing | The mirror of FLIP. Forensic audit of incoming claims for payers and internal integrity review. |

---

## 2. Architecture — A Correlate-Layer, Not a Replacement

PREDIQA continuously reads from your source systems via industry-standard protocols, normalizes everything into one unified data model, and makes it available to the engines. **It writes nothing back to your EHR.** Your systems stay; your vendors stay; your fee structures stay.

> 🔒 **A secure, read-only process.** Source systems are never modified. Every external communication requires a human dispatch token — this gate is absolute and cannot be bypassed by any configuration.

---

## 3. The Agentic Loop

### 3.1 Stage 01 — Data Ingestion: Six Source Categories

Each source has a dedicated connector microservice that fetches and forwards raw payloads — no transformation, just delivery. Everything lands in a Kafka staging queue with 30-day replay, then flows to the normalization engine.

| # | Source | Protocol | Extracts |
|---|---|---|---|
| 1 | **EHR / EMR** | Epic FHIR R4 · Cerner SMART · Athenahealth | Discharge summaries, ADT events, LOS, order sets, vitals, attending assignments, clinical notes, care plans |
| 2 | **RCM / Billing** | X12 EDI · 837P/I · 835 | Claim status, denial codes, AR aging, package utilization, charge capture, pre-auth status |
| 3 | **Lab / Pharmacy** | FHIR R4 · LIS/pharmacy REST | Test results, result trends, drug dispense records, inventory levels |
| 4 | **Payer APIs** | X12 270/271 · payer REST | Eligibility, prior auth, and denial reason codes from your top 8–10 payers per market |
| 5 | **HR / Operations** | CSV · REST · SFTP from HRIS | Staffing ratios, scheduling patterns, incident reports, credentialing and certification status |
| 6 | **External signals** | HCAHPS/CMS · epidemiology · CMMS | Quality feeds, outbreak alerts, seasonal trends, policy updates, device-uptime data |

> **Data required from you · ingestion:** EHR API credentials (FHIR sandbox access), RCM X12 feed access, payer API credentials for top payers, HR data export format, and a patient de-identification mapping table. Plan for a **6–9 month integration timeline** for Epic production credentials — budget accordingly.

### 3.2 Stage 02 — Signal Emission: Six Lever Emitters

Each emitter reads normalized data from its domain, applies detection rules, and emits typed JSON signals to a central bus. A signal is the atomic unit of intelligence — small, precise, richly annotated, with a severity weight from 1–100. **Signals below 0.65 confidence are excluded from correlation.**

**C — Clinical** *(care quality & risk)*
- LOS deviation >1.5 SD from DRG-expected norm
- Sepsis bundle compliance (CMS SEP-1 windows)
- Deterioration trajectory from vitals trend
- 30-day readmission probability at discharge
- Care-pathway adherence, top 50 DRGs

**O — Operational** *(flow & capacity)*
- Bed utilization and ICU occupancy trend
- ED boarding hours breach detection
- OR throughput and first-case start compliance
- Discharge TAT deviation from baseline
- Nurse-to-patient ratio vs. acuity mismatch

**F — Financial** *(revenue integrity)*
- Real-time denial rate by payer/DRG/attending
- AR aging velocity & ML collectability
- Charge-capture gap (ordered vs. billed)
- Claim SLA breach risk — 48h warning
- Revenue leakage per deficiency type

**E — Empathy** *(human experience)*
- Patient satisfaction micro-signals via EMPATHIFY
- Care-team communication gap detection
- Family-update compliance tracking
- Advance-directive compliance, 4h routing
- Staff sentiment from behavioral patterns

**L — Legal** *(compliance & coding)*
- Up-coding deviation vs. history & national norms
- CPT unbundling against CMS NCCI edits
- OIG Work Plan risk scoring
- HIPAA anomaly detection (after-hours, bulk)
- No-treating-relationship access flags

**R — Resources** *(supply & assets)*
- Medical device uptime & calibration (CMMS)
- Pharmacy stock depletion forecasting
- Consumable burn-rate anomalies
- Staff certification & credentialing expiry
- Critical supply-chain disruption signals

### 3.3 The Complete Loop — Seven Stages

From raw data to a tiered action and back into the learning model. Each stage is either fully automated (SYSTEM) or paused for your experts (HUMAN GATE). The system earns more autonomy only as accuracy is demonstrated.

| Stage | Process | Gate | Output |
|---|---|---|---|
| **01 · Ingestion** | Six source categories → Kafka staging → normalization into the unified PREDIQA data model. Patient de-duplication, terminology mapping (ICD-10, CPT-4, SNOMED, LOINC, RxNorm), provenance tagging. | System + Gate 1 | Unified internal data model |
| **02 · Signal emission** | Six lever emitters run in parallel, applying detection rules and emitting typed JSON signals with severity weight and confidence to the central signal bus. | System + Gate 2 | Typed signals, severity 1–100 |
| **03 · Correlation** | A worker matches signals sharing an entity (unit, attending, payer, patient, DRG) within a 45-day window, then multiplies combined severity using proprietary amplifier weights. | System + Gate 3 | Compound risk candidates |
| **04 · Risk scoring** | `compound_score = Σ(severity weights) × correlation amplifier`. Classified Critical (80–100), High (60–79), Moderate (40–59), or Watch (1–39). | System + Gate 4 | Score 1–100 + tier |
| **05 · Decision engine** | The scored event is matched to your `action_policy` table, which you configure: for each signal type, lever combination, and risk tier, you specify which action tier is authorized. | System + Gate 5 | Tiered action dispatch or hold |
| **06 · Action** | Tier 1 auto-executes; Tier 2 awaits one-click confirmation; Tier 3 notifies; Tier 4 logs. No external communication leaves without a human dispatch token. | System + Gate 6 | Alert · draft · queue change · escalation |
| **07 · Feedback & learning** | Every approval and rejection at Gates 5 and 6 is logged as training data. Amplifier weights recalibrate monthly; the model retrains quarterly on de-identified cross-hospital data with dual CMIO + data-science sign-off. | System + Gate 7 | Improved weights next cycle |

### 3.4 Action Authority Tiers

You decide what the system may do on its own. Every action falls into one of four tiers. Expanding autonomy — moving an action from Tier 3 to Tier 2, or Tier 2 to Tier 1 — requires written sign-off from your CMIO or equivalent. **This is a governance decision, not a product setting.**

| Tier | Mode | Examples |
|---|---|---|
| **Tier 1** | Auto-execute | Push alerts to a role · draft documents · reorder billing queue · SLA countdown alerts · bed & stock forecasts |
| **Tier 2** | Confirm first | Hold claim submission · escalate to intensivist · flag for compliance · staff reallocation · initiate prior auth |
| **Tier 3** | Notify only | Protocol-change ideas · staffing policy flags · executive risk reports · payer-contract escalations · HCAHPS trend alerts |
| **Tier 4** | Log only | Watch-tier signals · below-threshold events · learning dataset entries · human-rejection records · outcome tracking |

---

## 4. Governance & Roles

### 4.1 One Engine, Six Views — What Each CXO Sees

PREDIQA produces one compound risk score. Every role sees a filtered, translated view — and the actions each is authorized to take.

**CEO — Command view** *(sample headline metric: 87, Critical)*
- 30-day risk-score trend
- Peer-hospital benchmark comparison
- Top 3 compound risk events active now
- Auto-generated monthly executive summary
- *Acts on:* Initiates emergency review · delegates to CMO & CFO

**CFO — Revenue integrity** *(sample headline metric: ₹18L at risk)*
- AR aging heat map with ML collectability
- Denial rate by payer, DRG, attending
- Charge-capture gap by department
- 5-day cash-flow forecast
- *Acts on:* Approves billing hold · releases corrected claims

**CMO / CMIO — Clinical quality** *(sample headline metric: 91% SEP-1)*
- LOS heat map by unit & attending
- Readmission risk scores at discharge
- Documentation completeness scorecard
- Deterioration alerts — live patient list
- *Acts on:* Confirms escalation · adjusts thresholds · signs model promotions

**Compliance — Legal & coding** *(sample headline metric: 3 OIG flags)*
- OIG risk score with Work Plan mapping
- Up-coding deviation alerts with evidence
- HIPAA audit anomaly log
- Corrective-action-plan tracker
- *Acts on:* Confirms review · approves record hold · files CAP

**Billing team — Claims desk** *(sample headline metric: 6 held)*
- Denial queue with priority ranking
- SLA countdown per payer deadline
- Missing-document upload prompts
- Draft query replies awaiting sign-off
- *Acts on:* Uploads docs · edits draft · releases claim

**Charge nurse / Ops — Flow control** *(sample headline metric: 42 pending)*
- Bed availability — 4h & 12h forecast
- Discharge queue by note completion
- Staff ratio vs. acuity, unit by unit
- ED boarding-hours alert
- *Acts on:* Confirms reallocation · prioritizes discharge · escalates bed crisis

### 4.2 Seven Human Validation Gates

PREDIQA operates within limits your people set. These are the seven points where expert input is mandatory — the system provides evidence of accuracy; your institution decides the boundary.

| Gate | Who | What they own |
|---|---|---|
| **1 · Data quality review** | IT / Data engineering | Confirms connector health, reviews the dead-letter queue, approves new data-source onboarding, validates terminology mapping. |
| **2 · Signal config** | CMO / CMIO | Sets severity thresholds per signal type, approves new signal types, signs off on changes to clinical detection rules. Quarterly. |
| **3 · Reference data** | Revenue cycle + Clinical informatics | Loads payer prior-auth matrices, DRG/LOS benchmarks, denial-pattern libraries, insurance rate schedules. A quarterly operational task. |
| **4 · Score validation** | Clinical + Financial leads | Reviews compound scores, marks false positives with a reason code, adjusts institution sensitivity. Feeds the quarterly recalibration. |
| **5 · Tier 2 confirmation** | Role-specific staff | One-click approval of consequential actions with a 10-minute TTL. Unconfirmed actions escalate to the department head. |
| **6 · Dispatch approval** | Billing / clinical staff | No external communication leaves without a human dispatch token. Reviewed, edited, and signed before dispatch. This gate is absolute. |
| **7 · Model governance** | CMIO + Data science lead | Dual written sign-off on every retrain and amplifier-weight change. CMIO holds veto power over any model promotion. Versions stored in MLflow. |

---

## 5. Onboarding & Data

PREDIQA calibrates to your institution's specific risk tolerance and clinical culture over 12–18 months. The faster reference data and credentials arrive, the faster the loop earns autonomy.

**Onboarding data checklist**

- **Integration:** EHR API credentials (FHIR sandbox → production), RCM X12 feed access, payer API keys, HRIS export format, patient de-identification mapping table.
- **Clinical baselines (Gate 2):** Historical LOS distribution by DRG, baseline sepsis compliance, documentation completeness average, HCAHPS baselines, historical denial rate by payer.
- **Reference data (Gate 3):** Payer prior-auth matrices, DRG-expected LOS norms, denial-pattern library, insurance rate schedules, plus auto-updated CMS NCCI edits and OIG Work Plan priorities.
- **Governance:** `action_policy` sign-off from CMIO (clinical), CFO (financial), and Chief Compliance Officer (legal) before the decision engine goes live.

> ⏱️ **Timeline reality:** budget 6–9 months for Epic production credentials. The continuous learning loop reaches institutional calibration over 12–18 months — human rejections at Gates 5 and 6 are the most valuable signal, teaching the system what your institution considers acceptable.

---

*PREDIQA — Hospital CXO Master · COFLER · FLIP · EMPATHIFY · FRAUDIQ · Confidential — Authorized technical & executive review only · v1.0 · 2026*

# Review Memo: Pipeline Integrity Pilot — Architecture & Requirements Review

*Companion to `PRD.md` (Rev. 2) and `TRD.md` (Rev. 2). This memo explains why those documents changed: what the situation was, what was reviewed, what was found, what the system looked like before the revision, and what moved as a result.*

---

## 1. The Situation

**The project:** "The Pipeline Integrity Pilot" — an 8-week, part-time (4–6 hrs/week/person) feasibility PoC for autonomous drone pipeline inspection, run by two people (Anshuman and Shashwat) for Deloitte Energy & Chemicals. It's derived from a planning workbook (`Pipeline_Integrity_Timeline.xlsx`) and had already been written up as a PRD and TRD (Rev. 1) before this review.

**What was asked:** whether Rev. 1's PRD and TRD were *sufficient* — complete and precise enough to hand to an engineering team (in this case, Claude Code) and build from — and a thorough critique of where they weren't.

**What was done:**
1. Read both documents in full and cross-referenced every factual claim against the source workbook.
2. Independently spot-verified two of the higher-stakes external claims (K-Pipelines' GPL-3.0 license, GasVid's frame-count arithmetic) rather than trusting the documents' own citations at face value — both held up.
3. Verified the project's own date math (Week 1 = week of Sept 7, 2026 is in fact a Monday; Week 8 does end Friday, Oct 30, 2026).
4. Produced a written critique, then a standalone architecture diagram (`architecture_diagram.svg`) so the architecture implicit in the TRD could be reviewed as one picture before any document text changed.
5. Produced Rev. 2 of both documents, incorporating the changes below.

This memo is the record of steps 2–4 and the bridge to Rev. 2 — read it if you want to know *why* something changed, not just *that* it changed.

---

## 2. The Problem, in One Paragraph

Rev. 1 was unusually well-diligenced for a converted spreadsheet — real datasets, real license tracking, honest "no target defined" flags instead of invented numbers. But "honest about what's undecided" isn't the same as "ready to build from." Five of Rev. 1's open items weren't just undocumented, they were places where two different engineers, working independently on a tight part-time schedule, could reasonably build two incompatible things, or where a demo could end up measuring something other than what it claimed to measure. Those five items, plus a handful of smaller documentation gaps, are what Rev. 2 addresses.

---

## 3. The Initial Architecture (Rev. 1)

Before any changes, here's what Rev. 1's TRD specified (see `architecture_diagram.svg` for the visual version):

**Conceptual model — Sense → Reason → Act → Adapt**, one track and one owner per stage:

| Stage | Track | Owner | Layer |
|---|---|---|---|
| Sense | 1. Sensing & Simulation | Anshuman | Data + Simulation |
| Reason | 3. Anomaly Detection Model | Shashwat | Model |
| Act | 2. Flight Planning & Work Orders | Anshuman | Orchestration |
| Adapt | 4. Integration, Dashboard & Demo | Shashwat | Integration + Reporting |

**End-to-end flow:** public datasets → license audit/cleaning → synthetic flight-path generator → two-channel capture log → [RGB corrosion classifier | OGI/methane classifier] → shared output contract → [flight re-route trigger + risk-ranked work order] → dashboard.

**The two load-bearing interfaces:**
- The **classifier output contract** (`{ class, confidence, location }`) — the boundary between Reason and Act.
- The **work-order schema** (`{ work_order_id, location, anomaly_type, source_classifier, confidence, urgency, recommended_action }`) — the boundary between Act and Adapt.

**What Rev. 1 left genuinely open**, rather than just undocumented:
1. No train/held-out split methodology — a real risk for GasVid specifically, since it's video and a naive split leaks near-duplicate frames across train/test.
2. No rule for where "anomaly injection" demo samples come from — training data or held-out data wasn't specified, so a demo could unintentionally rig itself.
3. RGB corrosion taxonomy left undefined, despite the source data itself spanning 8 categories and OGI already being defined as binary — a real inconsistency, not just an unfilled field.
4. The GPL-3.0 / unresolved-license question was modeled as an engineering checklist item, with no escalation path or fallback if it doesn't clear.
5. "Adapt" was used as a stage name for what the architecture actually implements as one-way reporting, with no feedback path back into Sense.

Two smaller structural notes on the documents themselves, not the system: TR-IDs existed for §4/§6/§8 but not for §7 (model specs), §9 (work-order schema), or §10 (dashboard) — so some PRD requirements pointed at prose instead of a numbered technical requirement. And feasibility (32–48 hours per person against the full scope) was never checked against the calendar.

---

## 4. What Changed in Rev. 2

| # | Issue | Where it was fixed |
|---|---|---|
| 1 | No leakage-safe train/held-out split methodology (GasVid is video) | TRD TR-4.6, TR-7.1a, TR-7.2a; PRD §5, FR-3.3 |
| 2 | Anomaly-injection sourcing unspecified — demo could pull from training data | TRD TR-6.2a, TR-7.4; PRD FR-1.3 |
| 3 | RGB taxonomy undefined despite 8-category source data | TRD §3.1 note, TR-5.2; PRD §6 (assumption 8), FR-3.1 |
| 4 | License/legal risk modeled as a checklist item, no escalation or fallback | TRD TR-4.1 (now a gate), §11, §15; PRD §10 |
| 5 | "Adapt" labeling implies a feedback loop that isn't built | TRD §1 note; PRD FR-2.4 |
| — | No numeric success bar — a PoC that can't fail | PRD §5, §6 (assumption 7); TRD §7.4 |
| — | No fallback if Checkpoint 1 slips | PRD §9a (new) |
| — | No hour-budget sanity check against the 8-week calendar | PRD §3 (Goals caveat); TRD §13a (new) |
| — | Missing TR-IDs for §7/§9/§10 | TRD §7, §9, §10 |
| — | Class imbalance unaddressed | TRD TR-4.7, §7.4, §15 |
| — | Pretrained backbone license untracked | TRD §7.1, §11 |
| — | Similarly-named datasets (GasVid/Gas-DB/GasSeg) undefined outside the TRD | PRD §12 Glossary |

Everything above is marked **[Rev. 2]** inline in both documents, so it's visible in context rather than only listed here.

---

## 5. What Rev. 2 Resolves vs. What Still Needs a Decision

Rev. 2 gives every open item a recommended default so nobody's blocked waiting on a meeting — but "recommended default" is not the same as "stakeholder-confirmed." Four items still need an actual yes/no from whoever owns this PoC before Week 3:

1. **The numeric floor** (majority-class baseline + F1 ≥ 0.65 per classifier) — reasonable as a floor, not validated against this specific problem's difficulty.
2. **The RGB taxonomy default** (binary for MVP) — the technically simpler choice, not necessarily what stakeholders want to see demoed.
3. **The license audit outcome** — Rev. 2 defines the gate and the fallback; it doesn't and can't resolve the actual GPL-3.0/Kaggle-license question, which needs real legal review.
4. **The resourcing conflict** (possible parallel "LeRobot" commitment) — still unconfirmed; Rev. 2 just makes clearer where the pressure will show up first (Week 1, Weeks 6–7) if it's real.

Everything else in Rev. 2 is a specific, adoptable default — reversible if the team or stakeholders prefer a different answer, but not left blank.

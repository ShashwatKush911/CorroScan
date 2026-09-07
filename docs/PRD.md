# Product Requirements Document (PRD)
# Autonomous Drone Pipeline Inspection — Feasibility PoC
### *"The Pipeline Integrity Pilot" — Deloitte Energy & Chemicals, Physical AI Catalog, Use Case 2*

## Document Control

| Field | Value |
|---|---|
| **Project** | Autonomous Drone Pipeline Inspection PoC ("Pipeline Integrity Pilot") — Deloitte Energy & Chemicals |
| **Source material** | `Pipeline_Integrity_Timeline.xlsx` (tabs: Overview, Weekwise Plan, Dashboard, Weekly Goals) |
| **Team** | Anshuman — Tracks 1 & 2 · Shashwat — Tracks 3 & 4 |
| **Commitment** | Part-time, ~4–6 hrs/week per person |
| **Duration** | 8 weeks |
| **Start** | Week of Sept 7, 2026 (Week 1) |
| **Target completion** | Friday, Oct 30, 2026 (Week 8) |
| **Status at time of writing** | 0% started — no code, simulation, or trained models exist yet |
| **Document status** | **Rev. 2** — revised Sept 3, 2026 following an architecture/requirements review. Rev. 1 is the version originally derived from the source workbook; changes in this revision are marked **[Rev. 2]** inline. Still pending the confirmations in §6. |
| **Companion documents** | `TRD.md` — technical architecture, data specs, and model/interface requirements · `Review_Memo.md` — what was reviewed, what was found, and the initial (Rev. 1) architecture this revision changes |

---

## 1. Background & Problem Statement

Onshore pipelines are typically inspected only via in-line inspection (ILI) runs every 3–7 years, leaving long blind windows between checks. A single pipeline failure costs an estimated $50M–$500M. Use Case 2 in the Physical AI catalog proposes closing that gap with autonomous drone fleets that continuously patrol pipeline right-of-way using methane sensors, optical gas imaging (OGI), and LiDAR — detecting micro-leaks, ground movement, and encroachment 8–14 weeks before failure risk becomes acute.

This PoC targets the "Core PAI" (Physical AI) capability underlying that vision: an anomaly-detection model trained on real aerial sensor data, demonstrated end-to-end from (simulated) drone capture through AI classification to a risk-ranked work order — **not** the full production system.

> **Provenance note:** the original use-case business case lives in a separate document (`Use case 2.xlsx`) that the source workbook references but that was not itself supplied for this analysis. The framing above reflects the workbook's own summary of it — treat it as second-hand if the original document surfaces later and disagrees with anything here.

## 2. Why This Isn't Just a Speculative PoC

Where an initial read of Use Case 2 might assume no real training data is available, the team verified — source-by-source, via papers, GitHub repos, and dataset cards, not just search snippets — that real, usable public datasets exist for **two of the use case's four sensing modalities**:

- RGB corrosion imagery
- OGI / methane leak imagery

Acoustic emission has a real but lab-bench-scale dataset and is treated as an **optional bonus module**. No public dataset exists for LiDAR-based ground-movement/encroachment detection, so that modality is **explicitly out of scope**. Full dataset detail, including independent re-verification done alongside this PRD, is in the companion TRD (§3).

Net effect: this PoC can build **two independently-trained, real classifiers** instead of one speculative proxy model.

## 3. Goals

By the end of Week 8, demonstrate one continuous, runnable pipeline — on simulated flight data plus the two real public sensor datasets — that:

1. Simulates a drone patrol generating two-channel sensor readings (RGB + OGI) along a synthetic pipeline corridor.
2. Runs both readings through independently-trained classifiers, each flagging injected anomalies with **class + confidence + location**.
3. Triggers simulated flight re-routing when either classifier flags an anomaly.
4. Combines both classifiers' findings into one **risk-ranked work order**.
5. Surfaces those findings in a minimal dashboard/report view.
6. Documents limitations and a Phase 2 roadmap for leadership.

> **[Rev. 2] Feasibility caveat:** the above is ~32–48 hours of work per person across 8 weeks. That's workable but tight once Week 1's six tasks (dataset acquisition, license audit, cleaning, tooling setup) and the possible parallel "LeRobot" commitment (§6, #3) are accounted for. §9a below defines what ships if Checkpoint 1 slips, so scope has a fallback instead of just a schedule.

## 4. Non-Goals / Explicitly Out of Scope

| Item | Why |
|---|---|
| Production system or real fleet deployment | Feasibility PoC only — matches the source use case's own framing as a "PoC Direction," not a production build |
| Real drone flights or access to an actual pipeline asset | No physical drone or field access exists for this project |
| LiDAR-based ground-movement/encroachment detection | Confirmed: no public dataset exists |
| Training or validating on the client's own pipeline data | Public research datasets only |
| Guaranteed inclusion of GasSeg or the acoustic bonus module | Both conditional — see §6 and §10 |

## 5. Success Criteria

The source plan defines "done" qualitatively, week by week (§9), rather than against a fixed model-quality bar:

- Both classifiers trained and evaluated, with accuracy/F1 documented. *(No numeric target is specified in the source plan.)* **[Rev. 2 — recommended strawman, pending stakeholder confirmation]:** each classifier must (a) beat a majority-class baseline on held-out data, and (b) reach F1 ≥ 0.65. This is a floor to avoid a technically-passing demo that isn't actually distinguishing anomalies, not a claimed target of model quality — stakeholders should feel free to raise or lower it, but a PoC with no bar at all can't fail, which defeats the purpose of a checkpoint.
- **[Rev. 2]** "Held-out" means a split made *before* training that the model never sees — and for GasVid specifically, split by video, not by frame, since adjacent frames from the same clip are near-duplicates and a frame-level split would inflate the reported score. See TRD §7.1a.
- One runnable, end-to-end pipeline: simulated capture → dual classification → flight re-route trigger → risk-ranked work order → dashboard view.
- Checkpoint 1 (Week 5) and Checkpoint 2/final submission (Week 8) both demoed live to stakeholders.
- Limitations write-up and a one-page Phase 2 roadmap delivered at final submission.

## 6. Assumptions Requiring Stakeholder Confirmation

| # | Assumption | Needs confirmation? | Why it matters |
|---|---|---|---|
| 1 | **Dataset availability** — RGB corrosion + OGI/methane are real and usable; acoustic is optional; LiDAR is out | **No** | Unlocks two real classifiers instead of one speculative proxy — confirm the team should proceed on this basis, and that the plan doesn't wait on GasSeg's request-access step |
| 2 | **No physical drone or field access** — simulation + proxy imagery only | **Yes** | Sets the PoC's ceiling: it proves the AI/software concept, not flight hardware or field operations |
| 3 | **Time budget** — ~4–6 hrs/week/person, possibly the same two people already committed to a parallel "LeRobot" pilot | **Yes** | If the same two people run both pilots, combined load could reach ~8–12 hrs/week across two initiatives on top of client work |
| 4 | **Task split** — Anshuman: Sensing + Flight Planning; Shashwat: Anomaly Detection + Integration | No — adjustable | Mirrors the dependency chain used to split the LeRobot pilot |
| 5 | **PoC scope** — simulated patrols over proxy data, not a production system | No — matches source framing | The source use-case document itself frames this as a "PoC Direction" |
| 6 | **Training compute** — Google Colab, no local GPU assumed | No — standard assumption | Consistent with how the LeRobot pilot handles the same constraint |
| 7 **[Rev. 2]** | **Minimum quality bar** — recommended strawman is "beats a majority-class baseline" + "F1 ≥ 0.65" per classifier (§5) | **Yes** | Without *some* numeric floor, Checkpoint 1/Final can't actually fail — confirm this bar, or set a different one, before Week 4 evaluation |
| 8 **[Rev. 2]** | **RGB corrosion output taxonomy** — recommended default is binary (`corrosion` / `no_corrosion`) for the MVP, with the scraped set's 8-category labels available as a stretch multi-class extension | **Yes** | Blocks Week 3 work in Track 3 the same way the output-contract decision does (§8, FR-3.1) — needs an answer before training starts, not after |

## 7. Stakeholders & Roles

| Person | Tracks | Responsibility |
|---|---|---|
| **Anshuman** | 1. Sensing & Simulation · 2. Flight Planning & Work Orders | Owns the "Sense" and "Act" halves of the loop: simulation environment, sensor injection, flight re-routing, work-order format |
| **Shashwat** | 3. Anomaly Detection Model · 4. Integration, Dashboard & Demo | Owns the "Reason" and "Adapt" halves: both classifiers, packaging, end-to-end wiring, reporting |
| Leadership / stakeholders | — | Review Checkpoint 1 (Wk 5) and final submission (Wk 8); confirm §6 assumptions |

*Eventual end users (pipeline integrity engineers who would act on real work orders) are out of scope for direct interaction in this PoC — they're the persona a Phase 2 production build would serve.*

## 8. Functional Requirements by Track

### Track 1 — Sensing & Simulation (Anshuman)

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-1.1 | Build a synthetic pipeline-corridor flight path (waypoints along a mock route) | A flight path exists and stands in for a real drone GPS track |
| FR-1.2 | Attach RGB + OGI samples to synthetic waypoints/timestamps | Produces a mock two-channel "drone capture log" |
| FR-1.3 | Extend to multiple patrol routes + configurable anomaly-injection points, for both channels | Simulator supports >1 route and injectable anomalies per channel. **[Rev. 2]** Samples used for injection in any Checkpoint 1/Final demo must be drawn from each classifier's held-out split, never the training split — otherwise the demo is measuring memorization, not detection (see TRD §6.2a) |
| FR-1.4 | Model realistic sensor noise/false-positive behavior per modality (IR vs RGB) | Injected readings reflect each sensor's real-world characteristics |
| FR-1.5 | Wire the simulator to trigger a sensor reading (either channel) at each waypoint, in the agreed output format | Waypoint traversal reliably emits a reading conforming to the classifier output contract (TRD §5) |
| FR-1.6 | Visualize a simulated patrol run (map + readings, both channels) | A patrol run can be visually inspected end-to-end |

### Track 2 — Flight Planning & Work Orders (Anshuman, builds on Track 1)

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-2.1 | Flight-path re-routing logic: given a flagged anomaly from either classifier, simulate diverting for a closer look | A flagged anomaly visibly alters the simulated flight path |
| FR-2.2 | Risk-ranked work-order format carrying findings from both classifiers | Format includes location, anomaly type, source classifier, confidence, urgency, recommended action |
| FR-2.3 | Generate a full risk-ranked work-order report from a complete patrol run, combining both classifiers | Report reflects a real run, not fixture data |
| FR-2.4 | Document the "Adapt" loop and the acoustic/LiDAR scope decisions | Written up alongside Week 7 deliverables. **[Rev. 2]** As built, "Adapt" is reporting — Track 4's dashboard doesn't feed anything back into Sense (no retraining or patrol-strategy adjustment). Say so explicitly in the write-up so leadership doesn't read "Adapt" as a closed feedback loop |

### Track 3 — Anomaly Detection Model (Shashwat)

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-3.1 | Baseline RGB corrosion classifier (transfer learning on a pretrained CNN) on the cleaned dataset | Model trains and produces predictions in the agreed output contract. **[Rev. 2 default, pending confirmation]** binary (`corrosion`/`no_corrosion`) for MVP; the scraped set's 8 corrosion categories are a stretch multi-class extension, not the Week 3 baseline |
| FR-3.2 | Baseline OGI/methane classifier (binary leak/no-leak) on GasVid | Model trains and produces predictions in the agreed output contract |
| FR-3.3 | Train/evaluate both classifiers; document accuracy/F1 | Metrics recorded against held-out data. **[Rev. 2]** For GasVid, the held-out split must be made by video, not by frame — random frame-level splitting leaks near-duplicate frames across train/test and inflates the reported score. Each classifier must also beat a majority-class baseline and reach F1 ≥ 0.65 (§5), or the gap must be reported explicitly rather than omitted |
| FR-3.4 | *(Stretch)* Extend the OGI classifier with Gas-DB segmentation masks, or GasSeg if access has arrived | Must not slip the Checkpoint 1 date |
| FR-3.5 | *(Optional)* Acoustic bonus classifier on GPLA-12 | Only pursued if confirmed worthwhile at the Checkpoint 1 review |

### Track 4 — Integration, Dashboard & End-to-End Demo (Shashwat, builds on Track 3)

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-4.1 | Package both classifiers as callable functions/services behind one clean shared interface | Both classifiers callable through the same interface shape |
| FR-4.2 | Wire simulated capture → both classifiers into a single script | A single script runs both classifiers against simulated input |
| FR-4.3 | Finish end-to-end wiring: capture → classifiers → re-route trigger → work order, as one runnable pipeline | One command/script produces a complete run |
| FR-4.4 | Minimal report/dashboard view showing findings from both classifiers | Findings from a run are visible in one place |

## 9. Milestones & Timeline

| Week | Phase | Goal / Definition of Done | Checkpoint |
|---|---|---|---|
| 1 | Onboarding | Both understand the use case; both verified real dataset families (RGB corrosion, OGI/methane) downloaded/vetted | |
| 2 | Onboarding | Synthetic two-channel drone-patrol simulation exists, wired to real sensor data; each classifier's I/O contract agreed | |
| 3 | Independent work | First simulation extension (Anshuman) + both baseline classifiers (Shashwat) in place | |
| 4 | Independent work | Simulator produces realistic two-channel patrol runs; both classifiers' held-out accuracy measured | |
| 5 | Independent work | Full simulated patrol run works end-to-end on the sensing side; both classifiers reliably flag injected anomalies | **Checkpoint 1** |
| 6 | Independent work | Flight re-routing logic + dual-classifier work-order format exist; both classifiers packaged for integration | |
| 7 | Independent work | Full risk-ranked work-order report combining both classifiers generated from a patrol run, wired into one runnable pipeline | |
| 8 | Wrap-up | Full simulated PoC (patrol → dual classification → risk-ranked work order) runs end-to-end; limitations + Phase 2 roadmap documented | **Final submission** |

## 9a. Descope Plan If Checkpoint 1 Slips **[Rev. 2 — recommended]**

The source plan has no fallback if Week 5 doesn't land as scoped — worth naming one now rather than improvising under time pressure in Week 5:

| If, by Checkpoint 1... | Then descope to... |
|---|---|
| Only one classifier is reliably flagging anomalies | Ship that one classifier end-to-end for the final demo; document the other as "trained but not yet meeting the floor" rather than blocking the whole pipeline on it |
| Neither classifier is ready | Push Checkpoint 1's sensing-side demo back one week, compressing Weeks 6–7's integration work rather than dropping it — flag this to stakeholders at the Week 5 review, don't wait until Week 7 |
| The simulation/integration side (Tracks 1, 2, 4) is behind but classifiers are fine | Demo the classifiers standalone on held-out data at Checkpoint 1 in place of the full simulated patrol run |

This doesn't replace the Week 5 stakeholder review — it just means that review has an actual decision to make instead of only a status update.

## 10. Risks & Open Questions

- **Resourcing conflict:** the same two people may already be committed to a parallel "LeRobot" pilot at a similar part-time cadence — combined load could reach ~8–12 hrs/week if confirmed. *(Needs confirmation, §6.)*
- **Dataset licensing:** the Kaggle corrosion dataset's license is unspecified/unvetted; K-Pipelines is GPL-3.0 (copyleft). **[Rev. 2]** This is a legal/compliance question, not something the two engineers should resolve informally — route both through Deloitte's OSS/legal review process in Week 1. **Fallback if unresolved by end of Week 1:** proceed on K-Pipelines + the cleaned scraped set only, and exclude the Kaggle set until it clears.
- **GasSeg access-gating:** available on request, not direct download — the plan explicitly says not to block Week 3 on it arriving.
- **No numeric accuracy/F1 target defined.** **[Rev. 2 — resolved with a recommended default]** see §5's strawman floor (majority-class baseline + F1 ≥ 0.65); stakeholders should confirm or adjust it rather than leave the bar unset.
- **No real-world validation:** both classifiers train/evaluate on public datasets, not the client's actual pipeline — a limitation to state plainly, not a defect to fix within this PoC.
- **Two data-count discrepancies surfaced during independent verification** (frame/image counts for the scraped RGB set and for GasVid) — see TRD §3 for details; worth resolving before Week 2/3 effort estimates are finalized.

## 11. Deliverables

- Working simulation environment (multi-route, two-channel, anomaly injection)
- Two trained/evaluated classifiers (RGB corrosion, OGI/methane) + optional acoustic bonus
- Flight re-routing logic
- Risk-ranked work-order generator
- Minimal dashboard/report view
- One end-to-end runnable pipeline
- Limitations write-up + one-page Phase 2 roadmap for leadership

## 12. Glossary

| Term | Meaning |
|---|---|
| ILI | In-Line Inspection — periodic internal pipeline inspection (today's 3–7 year cycle baseline) |
| OGI | Optical Gas Imaging — infrared imaging used to visualize gas leaks |
| PAI | Physical AI — the catalog this use case belongs to |
| RGB | Standard color imagery (as opposed to IR/thermal) |
| LiDAR | Light Detection and Ranging — used for ground-movement/encroachment sensing (out of scope here) |
| CNN | Convolutional Neural Network |
| F1 score | Harmonic mean of precision and recall — a standard classifier evaluation metric |
| GPLA-12 | The acoustic leak dataset used for the optional bonus module |
| GasVid **[Rev. 2]** | The OGI baseline training video dataset (Stanford/METEC) — not to be confused with Gas-DB or GasSeg below |
| Gas-DB **[Rev. 2]** | The RGB-thermal OGI dataset (also called RT-CAN) usable for a stretch segmentation extension — distinct from GasVid and GasSeg despite the similar name |
| GasSeg **[Rev. 2]** | The largest OGI dataset, access-gated (request-only) — a stretch/optional source, not the Week 3 baseline |
| K-Pipelines **[Rev. 2]** | The GPL-3.0, Stable-Diffusion-synthetic RGB corrosion dataset — see licensing risk, §10 |
| mIoU / mF1 **[Rev. 2]** | Mean Intersection-over-Union / mean F1 — segmentation-quality metrics cited as GasSeg's own reported benchmark (TRD §7.2), not a target this PoC's classifiers are expected to hit |

## 13. Source Documents & Provenance

- `Pipeline_Integrity_Timeline.xlsx` — the workbook this PRD is derived from (Overview, Weekwise Plan, Dashboard, Weekly Goals tabs), last modified by Shashwat Kushwaha.
- `Use case 2.xlsx` — the original Physical AI catalog use-case document, referenced but not directly supplied for this analysis.
- `TRD.md` — companion technical requirements document, derived from the same workbook plus independent dataset re-verification (Sept 2, 2026).
- `Review_Memo.md` **[Rev. 2]** — architecture and requirements review (Sept 3, 2026) that produced the Rev. 2 changes in both this document and `TRD.md`; documents the initial (Rev. 1) architecture, what was checked, and what each change addresses.

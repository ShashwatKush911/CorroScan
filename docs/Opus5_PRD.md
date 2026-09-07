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
| **Document status** | **Rev. 3** — revised Sept 7, 2026 (Week 1, Day 1) following an architecture-level review of Rev. 2. Rev. 1 was derived from the source workbook; Rev. 2 closed requirement-level gaps; Rev. 3 closes architecture-level gaps. Changes in this revision are marked **[Rev. 3]** inline; **[Rev. 2]** markers are retained for revision history. Still pending the confirmations in §6. |
| **Companion documents** | `Opus5_TRD.md` — technical architecture, data specs, and model/interface requirements · `Opus5_Change_Notes.md` — what changed in Rev. 3 and why · `Review_Memo.md` — the Rev. 1 → Rev. 2 review record |
| **Timing note [Rev. 3]** | Rev. 3 is dated the first day of Week 1. Every deadline it introduces (contract freeze end of Week 2, risk model before Week 3) is ahead of the work it governs — none of it is retrofitted onto work already done. |

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

> **[Rev. 3] What "real datasets" does and doesn't buy us.** Real data means the classifiers learn from genuine corrosion and genuine methane plumes rather than invented proxies — that is a real advantage and the reason this PoC is worth running. It does **not** mean the resulting accuracy transfers to a client pipeline. Both sources are narrow: GasVid is a single camera at a single controlled-release facility, and one of the three RGB sources is Stable-Diffusion synthetic. See §5 and §10 for how this is measured and reported rather than glossed.

## 3. Goals

By the end of Week 8, demonstrate one continuous, runnable pipeline — on simulated flight data plus the two real public sensor datasets — that:

1. Simulates a drone patrol generating two-channel sensor readings (RGB + OGI) along a synthetic pipeline corridor.
2. Runs both readings through independently-trained classifiers, each flagging injected anomalies with **class + confidence + location**.
3. Triggers simulated flight re-routing when either classifier flags an anomaly.
4. Combines both classifiers' findings into one **risk-ranked work order**.
5. Surfaces those findings in a minimal dashboard/report view.
6. Documents limitations and a Phase 2 roadmap for leadership.

> **[Rev. 2] Feasibility caveat:** the above is ~32–48 hours of work per person across 8 weeks. That's workable but tight once Week 1's six tasks (dataset acquisition, license audit, cleaning, tooling setup) and the possible parallel "LeRobot" commitment (§6, #3) are accounted for. §9a below defines what ships if Checkpoint 1 slips, so scope has a fallback instead of just a schedule.

> **[Rev. 3] Goal 4 needs a definition, not just a name.** "Risk-ranked" is the goal most visible to stakeholders and, in Rev. 2, the only one with no logic behind it — the work-order schema had an `urgency` field but nothing that produced it. Rev. 3 defines the ranking rule explicitly (TRD §9.2) and makes it a Week-2/3 decision rather than something improvised in Week 6. Goal 4 is not met by emitting a list; it is met by emitting a list whose **order is defensible**.

## 4. Non-Goals / Explicitly Out of Scope

| Item | Why |
|---|---|
| Production system or real fleet deployment | Feasibility PoC only — matches the source use case's own framing as a "PoC Direction," not a production build |
| Real drone flights or access to an actual pipeline asset | No physical drone or field access exists for this project |
| LiDAR-based ground-movement/encroachment detection | Confirmed: no public dataset exists |
| Training or validating on the client's own pipeline data | Public research datasets only |
| Guaranteed inclusion of GasSeg or the acoustic bonus module | Both conditional — see §6 and §10 |
| **[Rev. 3]** Any claim of cross-camera / cross-site generalization | Both modalities train on a single capture setup each; the PoC measures in-distribution performance only, and says so (§5, §10) |
| **[Rev. 3]** Physically-simulated sensor synthesis | The simulator replays real labeled samples onto waypoints; it does not synthesise imagery from a physical sensor model (TRD §6.7) |

## 5. Success Criteria

The source plan defines "done" qualitatively, week by week (§9), rather than against a fixed model-quality bar:

- Both classifiers trained and evaluated, with accuracy/F1 documented. *(No numeric target is specified in the source plan.)* **[Rev. 2 — recommended strawman, pending stakeholder confirmation]:** each classifier must (a) beat a majority-class baseline on held-out data, and (b) reach F1 ≥ 0.65. This is a floor to avoid a technically-passing demo that isn't actually distinguishing anomalies, not a claimed target of model quality — stakeholders should feel free to raise or lower it, but a PoC with no bar at all can't fail, which defeats the purpose of a checkpoint.
- **[Rev. 2]** "Held-out" means a split made *before* training that the model never sees — and for GasVid specifically, split by video, not by frame, since adjacent frames from the same clip are near-duplicates and a frame-level split would inflate the reported score. See TRD §7.1a.
- One runnable, end-to-end pipeline: simulated capture → dual classification → flight re-route trigger → risk-ranked work order → dashboard view.
- Checkpoint 1 (Week 5) and Checkpoint 2/final submission (Week 8) both demoed live to stakeholders.
- Limitations write-up and a one-page Phase 2 roadmap delivered at final submission.

**[Rev. 3] Three criteria added, because Rev. 2 could be fully satisfied by a system whose headline output was still wrong:**

- **Ranking correctness.** Given a fixed set of findings, the work order's ordering matches the documented risk model (TRD §9.2). Verified by an assertion in the end-to-end smoke test, not by eyeballing the demo. *A correctly-ordered list of two items proves more than a beautifully-rendered list in arbitrary order.*
- **No cross-model confidence comparison.** Ordering must never depend on comparing a raw confidence score from the RGB model against one from the OGI model — these are separately-trained, separately-miscalibrated networks and the comparison is meaningless. Ranking is anchored to modality-level urgency first (TRD §7.5, §9.2).
- **Reported performance is scoped.** Every accuracy/F1 figure presented at Checkpoint 1 or Final is labelled with the capture setup it was measured on. A number without that label is not a reportable result.

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
| 9 **[Rev. 3]** | **Risk model** — recommended default ranks a methane leak above RGB corrosion at equal confidence, on the grounds that a leak is an acute safety/loss event (hours–days) while corrosion is a degradation signal (weeks–months). Full band table in TRD §9.2 | **Yes** | This is a *domain* judgement, not an engineering one — it encodes what the client would actually dispatch a crew for. Getting it wrong makes every demo work order subtly wrong. Needs an answer before Week 3 |
| 10 **[Rev. 3]** | **Recommended-action wording** — the `recommended_action` strings in the work order are plausible integrity-management language written by the team, not client-approved operational procedure | **Yes** | These strings are the most "real-looking" part of the demo and the easiest for a stakeholder to mistake for validated guidance. Confirm they read as illustrative, or have someone with domain authority supply the real wording |

## 7. Stakeholders & Roles

| Person | Tracks | Responsibility |
|---|---|---|
| **Anshuman** | 1. Sensing & Simulation · 2. Flight Planning & Work Orders | Owns the "Sense" and "Act" halves of the loop: simulation environment, sensor injection, flight re-routing, work-order format |
| **Shashwat** | 3. Anomaly Detection Model · 4. Integration, Dashboard & Demo | Owns the "Reason" and "Adapt" halves: both classifiers, packaging, end-to-end wiring, reporting |
| Leadership / stakeholders | — | Review Checkpoint 1 (Wk 5) and final submission (Wk 8); confirm §6 assumptions |

*Eventual end users (pipeline integrity engineers who would act on real work orders) are out of scope for direct interaction in this PoC — they're the persona a Phase 2 production build would serve.*

> **[Rev. 3] Shared ownership of the interface.** The classifier output contract (TRD §5) and the work-order schema (TRD §9) are the only two artifacts *neither* person solely owns — they are the seam between the two halves of the system. Rev. 3 makes them jointly-owned, version-controlled code rather than prose both sides interpret independently (FR-4.5, TRD §5.4). If there is one coordination failure this project is likely to have, it is here.

## 8. Functional Requirements by Track

### Track 1 — Sensing & Simulation (Anshuman)

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-1.1 | Build a synthetic pipeline-corridor flight path (waypoints along a mock route) | A flight path exists and stands in for a real drone GPS track |
| FR-1.2 | Attach RGB + OGI samples to synthetic waypoints/timestamps | Produces a mock two-channel "drone capture log". **[Rev. 3]** Every emitted reading carries a timestamp and a waypoint id, both required by the widened contract (TRD §5.3) |
| FR-1.3 | Extend to multiple patrol routes + configurable anomaly-injection points, for both channels | Simulator supports >1 route and injectable anomalies per channel. **[Rev. 2]** Samples used for injection in any Checkpoint 1/Final demo must be drawn from each classifier's held-out split, never the training split — otherwise the demo is measuring memorization, not detection (see TRD §6.2a) |
| FR-1.4 | Model realistic sensor noise/false-positive behavior per modality (IR vs RGB) | Injected readings reflect each sensor's real-world characteristics |
| FR-1.5 | Wire the simulator to trigger a sensor reading (either channel) at each waypoint, in the agreed output format | Waypoint traversal reliably emits a reading conforming to the classifier output contract (TRD §5). **[Rev. 3]** Conformance is enforced by importing and validating against the shared contract object — not by matching a documented shape by hand |
| FR-1.6 | Visualize a simulated patrol run (map + readings, both channels) | A patrol run can be visually inspected end-to-end |
| FR-1.7 **[Rev. 3]** | State plainly, in code comments and the Week-7 write-up, that the simulator *replays labeled dataset samples* onto waypoints rather than synthesising sensor output | The write-up contains an explicit sentence to this effect; no demo narration describes the simulator as generating sensor data (TRD §6.7) |

### Track 2 — Flight Planning & Work Orders (Anshuman, builds on Track 1)

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-2.1 | Flight-path re-routing logic: given a flagged anomaly from either classifier, simulate diverting for a closer look | A flagged anomaly visibly alters the simulated flight path |
| FR-2.2 | Risk-ranked work-order format carrying findings from both classifiers | Format includes location, anomaly type, source classifier, confidence, urgency, recommended action |
| FR-2.3 | Generate a full risk-ranked work-order report from a complete patrol run, combining both classifiers | Report reflects a real run, not fixture data |
| FR-2.4 | Document the "Adapt" loop and the acoustic/LiDAR scope decisions | Written up alongside Week 7 deliverables. **[Rev. 2]** As built, "Adapt" is reporting — Track 4's dashboard doesn't feed anything back into Sense (no retraining or patrol-strategy adjustment). Say so explicitly in the write-up so leadership doesn't read "Adapt" as a closed feedback loop. **[Rev. 3]** Extend the same honesty to "Act": re-routing is a deterministic confidence threshold firing on anomalies injected at known waypoints, demonstrating the wiring — not autonomous decision-making |
| FR-2.5 **[Rev. 3]** | Implement the risk model as an explicit, readable mapping — `(modality, class, confidence band) → urgency → recommended_action` — per TRD §9.2, rather than deriving urgency inline wherever the work order is built | The mapping exists as one reviewable table/function; changing the ranking policy means editing that one place. Ordering matches the table for a fixed fixture set (FR-4.6) |

### Track 3 — Anomaly Detection Model (Shashwat)

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-3.1 | Baseline RGB corrosion classifier (transfer learning on a pretrained CNN) on the cleaned dataset | Model trains and produces predictions in the agreed output contract. **[Rev. 2 default, pending confirmation]** binary (`corrosion`/`no_corrosion`) for MVP; the scraped set's 8 corrosion categories are a stretch multi-class extension, not the Week 3 baseline |
| FR-3.2 | Baseline OGI/methane classifier (binary leak/no-leak) on GasVid | Model trains and produces predictions in the agreed output contract |
| FR-3.3 | Train/evaluate both classifiers; document accuracy/F1 | Metrics recorded against held-out data. **[Rev. 2]** For GasVid, the held-out split must be made by video, not by frame — random frame-level splitting leaks near-duplicate frames across train/test and inflates the reported score. Each classifier must also beat a majority-class baseline and reach F1 ≥ 0.65 (§5), or the gap must be reported explicitly rather than omitted. **[Rev. 3]** Every reported figure is labelled with its capture setup (§5) |
| FR-3.4 | *(Stretch)* Extend the OGI classifier with Gas-DB segmentation masks, or GasSeg if access has arrived | Must not slip the Checkpoint 1 date. **[Rev. 3]** If Gas-DB is used at all, prefer holding it **entirely out of training** as a small cross-setup sanity check (TRD §7.2c) over adding it as extra training data — a second capture setup is worth more as a test than as 1,293 more training images |
| FR-3.5 | *(Optional)* Acoustic bonus classifier on GPLA-12 | Only pursued if confirmed worthwhile at the Checkpoint 1 review |
| FR-3.6 **[Rev. 3]** | *(Stretch)* Add a temporal feature to the OGI classifier — frame differencing against a short rolling background — as an additional input channel | Only attempted if the frame-level baseline underperforms its floor and time allows. If not attempted, the Week-7 write-up states that OGI was modelled frame-wise and that plume motion, the primary human cue, was therefore unavailable to the model (TRD §7.2b) |
| FR-3.7 **[Rev. 3]** | Publish each trained classifier as a versioned artifact — weights + config + metrics — at the agreed path, loadable without re-running the training notebook | Track 4 can load and run either classifier from a clean session without access to Track 3's notebook state (TRD §8.5) |

### Track 4 — Integration, Dashboard & End-to-End Demo (Shashwat, builds on Track 3)

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-4.1 | Package both classifiers as callable functions/services behind one clean shared interface | Both classifiers callable through the same interface shape |
| FR-4.2 | Wire simulated capture → both classifiers into a single script | A single script runs both classifiers against simulated input |
| FR-4.3 | Finish end-to-end wiring: capture → classifiers → re-route trigger → work order, as one runnable pipeline | One command/script produces a complete run |
| FR-4.4 | Minimal report/dashboard view showing findings from both classifiers | Findings from a run are visible in one place. **[Rev. 3]** The view displays urgency band and source modality per finding, not confidence alone — a bare confidence column invites exactly the cross-model comparison §5 forbids |
| FR-4.5 **[Rev. 3]** | Own the shared contract module: define the classifier output contract and work-order schema as validated objects in `models/interfaces.py`, imported by every producer and consumer | Both tracks import the same module; an invalid reading raises at construction rather than surfacing as a downstream bug. Contract frozen end of Week 2 (§9) |
| FR-4.6 **[Rev. 3]** | Extend the end-to-end smoke test with two assertions: a fixed fixture set ranks in the documented order, and each classifier's held-out F1 has not fallen below its recorded floor | Both assertions run as part of `run_e2e_demo.py`; a ranking regression or a model regression fails the run rather than passing silently (TRD §14) |

## 9. Milestones & Timeline

| Week | Phase | Goal / Definition of Done | Checkpoint |
|---|---|---|---|
| 1 | Onboarding | Both understand the use case; both verified real dataset families (RGB corrosion, OGI/methane) downloaded/vetted. **[Rev. 3]** Model-artifact handoff convention and data-persistence location agreed and written down (TRD §8.5, §4.8) — both are one-paragraph decisions that block Week 6 if deferred | |
| 2 | Onboarding | Synthetic two-channel drone-patrol simulation exists, wired to real sensor data; each classifier's I/O contract agreed. **[Rev. 3] Contract frozen as code by end of Week 2** — after this point it changes only by deliberate version bump, not ad-hoc edit (§9b) | |
| 3 | Independent work | First simulation extension (Anshuman) + both baseline classifiers (Shashwat) in place. **[Rev. 3]** Risk model (TRD §9.2) decided *before* training starts, alongside the taxonomy decision it depends on | |
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
| **[Rev. 3]** Time is short in Weeks 6–7 | Cut the dashboard to a sorted static table before cutting the risk model. A plain correctly-ranked list demonstrates the product thesis; a polished view over an arbitrary order does not |

This doesn't replace the Week 5 stakeholder review — it just means that review has an actual decision to make instead of only a status update.

## 9b. Interface Freeze **[Rev. 3]**

The output contract is the point where both people's work meets, and everything downstream — simulator wiring, integration, work order, dashboard — depends on its shape. In Rev. 2 it was "agreed in Week 2/3," which leaves it informally mutable for the whole project. Because this is a *chain* topology with a single fan-in, any late change to it costs both people rework simultaneously.

**Rule:** the contract is frozen at end of Week 2. After that:

- Additive, optional fields may be added at any time (they don't break existing consumers).
- Renames, removals, or type changes require both owners to agree and a version bump on the contract module.
- Nothing downstream reads a field the contract doesn't declare.

An early frozen interface is what allows two part-time people to work genuinely in parallel rather than serially. This is a cheap discipline that buys back the schedule risk identified in TRD §13a.

## 10. Risks & Open Questions

- **Resourcing conflict:** the same two people may already be committed to a parallel "LeRobot" pilot at a similar part-time cadence — combined load could reach ~8–12 hrs/week if confirmed. *(Needs confirmation, §6.)*
- **Dataset licensing:** the Kaggle corrosion dataset's license is unspecified/unvetted; K-Pipelines is GPL-3.0 (copyleft). **[Rev. 2]** This is a legal/compliance question, not something the two engineers should resolve informally — route both through Deloitte's OSS/legal review process in Week 1. **Fallback if unresolved by end of Week 1:** proceed on K-Pipelines + the cleaned scraped set only, and exclude the Kaggle set until it clears.
- **GasSeg access-gating:** available on request, not direct download — the plan explicitly says not to block Week 3 on it arriving.
- **No numeric accuracy/F1 target defined.** **[Rev. 2 — resolved with a recommended default]** see §5's strawman floor (majority-class baseline + F1 ≥ 0.65); stakeholders should confirm or adjust it rather than leave the bar unset.
- **No real-world validation:** both classifiers train/evaluate on public datasets, not the client's actual pipeline — a limitation to state plainly, not a defect to fix within this PoC.
- **Two data-count discrepancies surfaced during independent verification** (frame/image counts for the scraped RGB set and for GasVid) — see TRD §3 for details; worth resolving before Week 2/3 effort estimates are finalized.
- **[Rev. 3] Confidence scores are not comparable across the two models.** Two separately-trained networks are each miscalibrated in their own way, so ranking a combined work order by raw confidence produces a near-arbitrary order at exactly the moment stakeholders are looking at it. Mitigated by anchoring ranking to modality-level urgency bands (TRD §7.5, §9.2), not by trying to make the numbers comparable.
- **[Rev. 3] Single-capture-setup training limits generalization.** GasVid is one FLIR GF-320 on a tripod at one facility; K-Pipelines is Stable-Diffusion synthetic. A high held-out F1 may reflect the model learning the rig or the generator rather than the phenomenon. Not fixable within this PoC's data budget — mitigated by reporting scope honestly (§5) and, if cheap, a cross-setup sanity check on held-out Gas-DB (TRD §7.2c).
- **[Rev. 3] Model artifacts have no home by default.** Colab sessions are ephemeral; without an agreed artifact path and format, Week 6 integration begins with "can you re-run your notebook and send me the weights." Mitigated by the Week 1 handoff decision (TRD §8.5).
- **[Rev. 3] The most demo-visible output is the least specified.** `recommended_action` strings and urgency bands will look authoritative in a stakeholder demo regardless of how casually they were written. Mitigated by §6 assumption 10 and by labelling them illustrative in the write-up.

## 11. Deliverables

- Working simulation environment (multi-route, two-channel, anomaly injection)
- Two trained/evaluated classifiers (RGB corrosion, OGI/methane) + optional acoustic bonus
- **[Rev. 3]** Versioned model artifacts for both classifiers — weights, config, and metrics — loadable independently of the training notebooks
- Flight re-routing logic
- Risk-ranked work-order generator
- **[Rev. 3]** The documented risk model (band table) that produces the ranking
- **[Rev. 3]** A shared, validated contract module used by both tracks
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
| Calibration **[Rev. 3]** | Whether a model's stated confidence matches its actual accuracy — a "0.9" from a well-calibrated model is right ~90% of the time. Neural networks are typically overconfident, and two different models are miscalibrated differently, which is why their scores can't be compared directly |
| Capture setup **[Rev. 3]** | The specific camera, mounting, lighting and site a dataset was recorded with. Models often learn the setup rather than the phenomenon, so held-out performance within one setup overstates real-world performance |
| Model artifact **[Rev. 3]** | The saved output of a training run — weights plus the config and metrics needed to load and interpret it — as distinct from the notebook that produced it |
| Urgency band **[Rev. 3]** | The discrete risk level (`low`/`medium`/`high`) assigned to a finding by the risk model, derived from modality and confidence range rather than from raw confidence |
| Contract freeze **[Rev. 3]** | The end-of-Week-2 point after which the shared interface changes only by deliberate version bump (§9b) |

## 13. Source Documents & Provenance

- `Pipeline_Integrity_Timeline.xlsx` — the workbook this PRD is derived from (Overview, Weekwise Plan, Dashboard, Weekly Goals tabs), last modified by Shashwat Kushwaha.
- `Use case 2.xlsx` — the original Physical AI catalog use-case document, referenced but not directly supplied for this analysis.
- `Opus5_TRD.md` — companion technical requirements document (Rev. 3), derived from the same workbook plus independent dataset re-verification (Sept 2, 2026) and the Rev. 3 architecture review.
- `Review_Memo.md` — architecture and requirements review (Sept 3, 2026) that produced the Rev. 2 changes; documents the initial (Rev. 1) architecture and what each Rev. 2 change addresses.
- `Opus5_Change_Notes.md` **[Rev. 3]** — what changed between Rev. 2 and Rev. 3, why each change was made, and standing commentary on the project.
- `PRD.md` / `TRD.md` — the Rev. 2 documents this revision supersedes, retained for history.

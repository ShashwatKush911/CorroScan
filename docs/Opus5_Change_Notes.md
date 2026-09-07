# Change Notes: Rev. 2 → Rev. 3
### *What changed in `Opus5_PRD.md` and `Opus5_TRD.md`, why, and what I'd still flag*

| Field | Value |
|---|---|
| **Date** | Sept 7, 2026 — Week 1, Day 1 |
| **Supersedes** | `PRD.md` and `TRD.md` (Rev. 2, Sept 3, 2026), retained in the repo for history |
| **Produces** | `Opus5_PRD.md`, `Opus5_TRD.md` (both Rev. 3) |
| **Prior review** | `Review_Memo.md` — the Rev. 1 → Rev. 2 record |
| **Nature of this revision** | Architecture, not requirements. Rev. 2 was a requirements review and did that job well; this is the layer underneath it |

---

## 1. Why there was a Rev. 3 at all

Rev. 2 is a good document set. It caught the things a requirements review is supposed to catch — train/test leakage, an undefined taxonomy, a licensing question being treated as an engineering chore, and a PoC with no bar it could actually fail. It also did something rarer: it wrote down what it *hadn't* resolved, instead of papering over it.

But a requirements review asks *"is each requirement complete and unambiguous?"* It doesn't ask *"if two people build exactly this, what happens where their work meets?"* That second question is where Rev. 3 comes from, and it surfaces a different class of problem — ones that are invisible when you read each track's requirements in isolation, because they live in the space *between* the tracks.

Three of them stood out enough to justify a revision:

1. **The headline feature had no logic behind it.** Every document says "risk-ranked work order." Nothing anywhere defined the ranking. `urgency: low|medium|high` existed as an enum with no rule producing it.
2. **The one seam between the two engineers was prose.** The output contract — the thing both people build against — was a JSON block in a Markdown file, and it structurally could not carry what the work order downstream needed.
3. **Trained models had no way out of the notebook.** Colab is ephemeral. Nothing said how weights get from Shashwat's training notebook to the integration script in Week 6.

None of these are exotic. They're the ordinary failure modes of a two-person project with a single fan-in point — which is exactly what this architecture is.

**Timing matters here.** Rev. 3 is dated the first day of Week 1. Every deadline it introduces sits ahead of the work it governs. Nothing in it is retrofitted onto work already done, and nothing in it asks anyone to redo anything.

---

## 2. The changes, and the reasoning for each

### 2.1 Defined the risk model — *the most important change*

**Where:** TRD §9.2 (new band table) · PRD FR-2.5, §6 assumption 9, Goal 4 note

**What was there:** a work-order schema with `urgency: "low | medium | high"` and `recommended_action: "string"`, and a note that the urgency scale "is not defined in the source plan."

**Why it needed fixing:** this is the product. Everything upstream — two classifiers, a simulator, a re-route trigger — exists to produce a ranked list of things a pipeline engineer should go look at. The ranking *is* the value proposition, and it was the single least-specified thing in the document set. Left as-is, it gets invented in Week 6 by whoever writes `work_order.py` first, under time pressure, in the second-tightest week of the schedule.

**What Rev. 3 does:** replaces it with an explicit table mapping `(modality, class, confidence band) → urgency → anomaly_type → recommended_action`, plus a stated sort key.

The substantive judgement in that table is that **a methane leak outranks corrosion at equal confidence** — a leak is an acute safety and loss event on a timescale of hours to days, corrosion is a degradation signal on a timescale of weeks to months. That's a domain call, not an engineering one, which is why PRD §6 assumption 9 explicitly routes it to stakeholders rather than presenting it as settled.

I deliberately made it a lookup table rather than a scoring function. A table is reviewable by someone who doesn't read Python, is trivially adjustable when stakeholders disagree with a band, and can't accumulate subtle behaviour the way a formula can.

### 2.2 Fixed a real bug in how findings would be ranked

**Where:** TRD §7.5 (new) · PRD §5, §10

**The problem:** the RGB and OGI classifiers are separately-trained networks on unrelated datasets. Neural network confidence outputs are systematically overconfident, and each model is miscalibrated *differently*. A `0.80` from one and a `0.80` from the other are not the same quantity. Rev. 2 specified one combined work order, and the only sortable numeric field was `confidence` — so the natural implementation sorts by it and produces a near-arbitrary order in the artifact stakeholders scrutinise most.

I want to be precise about why this is a genuine defect rather than pedantry: it isn't that the ordering would be slightly suboptimal. It's that the ordering would carry no information, while *looking* authoritative.

**The fix, and why I chose the cheap one:** rank by modality-anchored urgency band first, then by confidence only *within* a band — where a model is compared against itself. The invalid comparison is removed rather than corrected.

I considered recommending temperature scaling as the primary fix and decided against it: it's real work, it needs a held-out set the team is already stretched to produce, and it would still leave two models' scores being compared across a boundary that means something. The band structure solves it for free and encodes a true domain fact — a leak and a rust patch aren't the same kind of event and shouldn't be interleaved because one number happens to be bigger. Temperature scaling is retained as an optional refinement for the within-band ordering.

### 2.3 Widened the output contract and made it executable

**Where:** TRD §5.3, §5.4, §5.5 · PRD FR-4.5, FR-1.5, §9b

**Two separate problems, same interface.**

*It couldn't carry the payload.* Rev. 2's `{class, confidence, location}` doesn't include `source_classifier`, `anomaly_type` or `urgency` — all required by the work order downstream. Track 2/4 would have to reconstruct provenance that was never handed to them. `location` was also one untyped string standing for two different concepts (which stop on the route, and where on Earth), and there was no timestamp — needed for patrol ordering and for any temporal OGI work.

Rev. 3 adds `reading_id`, `waypoint_id` + structured `coordinate`, `timestamp`, `source_modality`, `model_version`, and a nullable `bbox`. The classifier now stamps its own provenance, so nothing downstream guesses.

I deliberately kept `urgency` and `recommended_action` **out** of the reading. Those are policy, produced by the risk model; a classifier emits observations. Mixing them would push business rules into the model layer and make the risk policy impossible to change in one place.

*It was prose.* Two people building in parallel Colab notebooks against a Markdown JSON block will diverge — on field names, casing, null handling — and won't find out until integration week. Rev. 3 requires the contract to be a validated object in `models/interfaces.py` that both sides import, so an invalid reading raises at construction. This turns a coordination problem into an import statement.

*And it was informally mutable.* Rev. 2 had it "agreed in Week 2/3," which leaves it changeable for the project's whole life. Because this architecture is a chain with a single fan-in, a late change costs both people rework simultaneously. PRD §9b freezes it at end of Week 2 with a simple rule: additive optional fields anytime, breaking changes need both owners and a version bump.

### 2.4 Defined how a trained model leaves its notebook

**Where:** TRD §8.5 (new) · PRD FR-3.7, Week 1 milestone

**What was missing:** Rev. 2 said "package both classifiers as callable functions/services" and stopped. It never said in what format, to what location, or under what version. Track 3 trains in Colab — ephemeral storage, sessions that time out. Track 4 loads those models in Week 6.

The gap is invisible when you read the plan because each track looks self-contained. It only exists *between* them, which is exactly why a requirements review wouldn't catch it.

**What it would have cost:** Week 6 opening with *"can you re-run your notebook and send me the weights?"* That's the highest-probability schedule risk in the whole plan and simultaneously the cheapest to remove — the fix is one paragraph agreed in Week 1.

**What Rev. 3 specifies:** a versioned artifact directory (`weights` + `config.json` + `metrics.json` + `VERSION`) on mounted Drive. Three details in there are doing real work:

- **Class order lives in `config.json`.** A silently reordered class list ships an inverted classifier that still scores plausibly. This is a genuinely nasty bug and costs one line to prevent.
- **Preprocessing travels with the weights.** A resize or normalisation mismatch degrades accuracy quietly instead of raising.
- **`metrics.json` records the F1 floor** the smoke test later asserts against (§2.7).

### 2.5 Pinned data persistence

**Where:** TRD §2, TR-4.8, TR-4.9, TR-4.10

Rev. 2's NFRs acknowledged Colab's ephemeral storage but the pipeline never said where processed data *lives* between sessions. Re-downloading and re-cleaning GasVid every session is a silent tax on a 4–6 hr/week budget, and inconsistent processing between runs quietly undermines the reproducibility the same document asks for.

Rev. 3 makes a Drive-mounted `data/processed/` the single source of truth, written once by the Week-2 cleaning pass, with a `manifest.json` recording counts, class breakdown and split assignment. TR-4.9 additionally requires a fixed, seed-pinned GasVid frame subset rather than decoding ~670K frames per session.

A side benefit: this largely defuses the §3.4 frame-count discrepancy. Once training runs on a documented subset, whether the full set is 670K or 1M is a storage-planning question, not a modeling one.

### 2.6 Extended Rev. 2's honesty pass to two more places

**Where:** TRD §1 ("Act" note), TR-6.7 · PRD FR-1.7, FR-2.4, §2, §4

Rev. 2 did something I think was the best judgement call in the document: it conceded that "Adapt" is a label, not a loop, and required the team to say so. Rev. 3 applies the identical logic one box earlier and one box upstream, because the same reasoning applies and Rev. 2 stopped short.

- **"Act" is a trigger, not a decision.** Re-routing is a confidence threshold firing on anomalies injected at *known* waypoints, producing a scripted path change. It demonstrates that the wiring works. It is not autonomous decision-making.
- **The simulator is a replayer, not a sensor.** It staples labeled dataset samples onto waypoints. That's a sound design — a physically-accurate IR sensor model is nowhere near a 32–48 hour budget — but a stakeholder watching readings appear along an animated corridor may reasonably conclude the system is *sensing*.
- **Reported figures need their capture setup attached.** GasVid is one FLIR GF-320 at one facility; K-Pipelines is Stable-Diffusion synthetic. "F1 0.81" reads as a general capability claim; "F1 0.81 on held-out GasVid videos (single camera, METEC facility)" is the same result, honestly scoped. One clause per figure.

None of these change what gets built. They change what gets claimed, and they cost a few sentences. I'd argue this is the highest return-on-effort content in the revision: the failure mode they prevent — a stakeholder forming a wrong impression of maturity that later shapes Phase 2 planning — is expensive and hard to walk back.

### 2.7 Added two assertions to the existing smoke test

**Where:** TRD §14 · PRD FR-4.6

Rev. 2's testing validated contract shape and ran one end-to-end smoke test. Neither exercised the ranking logic nor guarded against model regressions — which are precisely the custom, unspecified, demo-critical parts. A ranking bug would pass the smoke test silently and be discovered on stage.

Rev. 3 adds exactly two assertions to the script that already exists: a fixed fixture set must rank in the documented order, and each classifier's held-out F1 must not drop below its recorded floor. No new framework, a few lines. I was deliberate about not proposing more — Rev. 2's judgement that a full CI suite is disproportionate here is correct, and I didn't want to erode it.

### 2.8 Smaller additions

| Change | Where | Why |
|---|---|---|
| Composition warning on mixed real/synthetic RGB sources | TRD §3.1, TR-7.1b | If K-Pipelines' synthetic images skew to one class, the model may learn "diffusion-generated?" as a shortcut for "corroded?". Detected by a per-source F1 breakdown |
| Source traceability metadata | TR-4.10 | Makes that per-source breakdown possible at all |
| Cross-setup sanity check on held-out Gas-DB | TR-7.2c | 1,293 extra training images barely move a model; a second independent capture setup is the only generalization evidence available here. Expect degradation — measure it, don't tune on it |
| Temporal OGI feature (stretch) | TR-7.2b, FR-3.6 | Plume motion is the primary human cue and frame-wise classification discards it. Offered as frame-differencing (preprocessing, not architecture). If skipped, say why, so a modest OGI F1 isn't read as failure |
| Dashboard shows urgency + modality, not bare confidence | TR-10.2 | A table sorted by a raw confidence column invites the exact cross-model comparison §7.5 prevents. The display should reinforce the risk model |
| Descope order: cut dashboard before risk model | PRD §9a, TR-10.3 | A plain correctly-ranked table proves the thesis; a polished view over an arbitrary order doesn't |
| `rank`, `reading_id`, `model_version` on work order | TRD §9.1 | Ordering becomes a recorded property, and findings become traceable to the artifact that produced them |
| Glossary: calibration, capture setup, model artifact, urgency band, contract freeze | PRD §12 | Rev. 2 added a glossary because similarly-named datasets were being confused; the same courtesy for Rev. 3's new concepts |
| Assumption 10 on `recommended_action` wording | PRD §6 | These strings are the most authoritative-looking text in the demo and are engineer-written, not client-approved |

---

## 3. What I deliberately did **not** change

Worth stating explicitly, because a reviewer who changes everything isn't reviewing:

- **The by-video split, held-out injection rule, and majority-class floor.** The leakage discipline is the strongest part of Rev. 2 and I didn't touch it.
- **Local Python callables over a network service layer.** Correct for this scale. Adding FastAPI would be pure ceremony.
- **The descope plan (§9a) and the license gate (TR-4.1).** Both well-judged. I added one row to the former and left the latter alone.
- **One smoke script instead of a CI suite.** Right call for the budget; §2.7 adds two lines to it rather than replacing it.
- **Binary taxonomy for the MVP.** Right default, still correctly flagged as needing confirmation.
- **The 8-week structure and track split.** No architectural reason to change them.
- **The honest "no target defined" flags.** Rev. 2's willingness to leave things visibly open is a feature. Rev. 3 fills gaps where a default is genuinely better than a blank, and preserves the open flag where the answer is genuinely a stakeholder's to give.

---

## 4. Schedule impact

Rev. 3 adds roughly **3–4 hours of Week 1–2 decision work**:

| Item | Rough cost |
|---|---|
| Contract as validated code | ~1–1.5 hrs |
| Artifact-handoff convention | ~30 min (a written decision, not code) |
| Risk-model band table | ~1 hr, mostly discussion |
| Persistence + manifest setup | ~1 hr, folded into Week 2 cleaning |

Against that it removes an unpredictable and larger amount of Week 6–7 work: interface renegotiation, re-running notebooks to recover weights, and inventing the ranking policy while trying to finish integration. It's a deliberate shift of effort into the weeks with the most slack, where decisions are cheapest.

The four Week 6–7 risks Rev. 2 flagged in §13a are all materially reduced, because by then the contract is frozen, the risk model is decided, and artifacts are already loadable. Those weeks become assembly rather than negotiation.

---

## 5. Standing commentary on the project

Things that aren't document changes but are worth someone holding in mind.

**The strongest thing about this project is its data honesty, and that's rarer than it sounds.** Most PoCs at this stage would have hand-waved "we'll find a dataset." This one verified sources against primary papers, caught a citation-drift error in a frame count that several published papers repeat, and tracked licenses per source. That discipline is worth protecting — it's the reason the eventual limitations write-up will be credible rather than defensive.

**The riskiest thing is not technical, it's narrative.** The classifiers will probably train fine; transfer learning on a binary image task is well-trodden. The genuine risk is the gap between what the demo *looks like* (an autonomous drone finding leaks) and what it *is* (a replayer, two image classifiers, a threshold, and a lookup table). That gap doesn't hurt anyone if it's named — and Rev. 2 already showed the team is willing to name it. Keep doing that; it's the project's real differentiator with a client.

**Two of the four "Sense → Reason → Act → Adapt" boxes contain real logic.** Reason (the classifiers) and the ranking inside Act. Sense is a replayer, Adapt is a report. That's a perfectly good PoC — it de-risks the software integration, which is genuinely the thing worth de-risking first — but if a Phase 2 proposal is built on this, the honest framing is *"we proved the pipeline plumbing and validated two detectors on public data,"* not *"we proved autonomous inspection."*

**The single-capture-setup ceiling is the finding most likely to be uncomfortable later.** If the OGI classifier posts a strong F1 on held-out GasVid videos, that number will get quoted. It is a real result and it deserves to be quoted — but it's measured on one camera at one facility with controlled releases, and there is currently no evidence about a different camera, weather, or standoff distance. The optional Gas-DB cross-setup check (TR-7.2c) is the cheapest way to have *something* to say about this, and I'd prioritise it over the segmentation stretch if there's time for exactly one.

**Watch the resourcing question — it's still the largest unquantified risk.** The possible parallel LeRobot commitment is flagged in both revisions and remains unconfirmed. It's the one risk that could invalidate the schedule regardless of how good the architecture is, and it's answerable with a single conversation. Worth having in Week 1.

**If only one thing from Rev. 3 gets adopted, make it the contract freeze.** Contract as code, frozen end of Week 2 (TRD §5.4/§5.5, PRD §9b). It's a few hours, it's the fan-in point of the whole architecture, and it's what allows two part-time people to work genuinely in parallel instead of serially. Everything else in this revision is an improvement; that one is structural.

---

## 6. Still needing a decision

Rev. 2 listed four items awaiting a stakeholder yes/no. Rev. 3 keeps all four and adds two:

| # | Item | Owner | Needed by |
|---|---|---|---|
| 1 | The numeric floor (majority baseline + F1 ≥ 0.65) | Stakeholders | Before Week 4 |
| 2 | RGB taxonomy (binary for MVP) | Stakeholders | Before Week 3 |
| 3 | License audit outcome — needs real legal review | Deloitte OSS/legal | End of Week 1 |
| 4 | The resourcing conflict (parallel LeRobot commitment) | Project owner | Week 1 |
| 5 **[Rev. 3]** | Risk model — does a methane leak outrank corrosion at equal confidence, are the band cut-offs right, and is there a minimum confidence below which no work order is raised at all? | Stakeholders / domain | Before Week 3 |
| 6 **[Rev. 3]** | `recommended_action` wording — illustrative, or should someone with domain authority supply real language? | Stakeholders / domain | Before Week 6 |

As in Rev. 2, every one of these has a recommended default in the documents, so nobody is blocked waiting on a meeting. A default is not a confirmation, and the documents keep saying so.

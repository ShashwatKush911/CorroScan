# Change Notes: Rev. 3 → Rev. 4 (Final)
### *What changed in `Final_PRD.md` and `Final_TRD.md`, why, and what I'd still flag*

| Field | Value |
|---|---|
| **Date** | Sept 7, 2026 — Week 1, Day 1 |
| **Supersedes** | `Opus5_PRD.md` and `Opus5_TRD.md` (Rev. 3, Sept 7, 2026), retained in the repo for history |
| **Produces** | `Final_PRD.md`, `Final_TRD.md` (both Rev. 4 / Final) |
| **Prior review** | `Opus5_Change_Notes.md` — the Rev. 2 → Rev. 3 record |
| **Nature of this revision** | Procedural, not architectural. Rev. 3 closed architecture gaps — the ranking logic, the interface contract, the model-artifact handoff, data persistence. This pass closes a different gap: *how the still-open human decisions actually get made*, plus one honesty note about the review chain that produced Rev. 2, Rev. 3, and this document itself |

---

## 1. Why there was a Rev. 4 at all

Asked directly whether Rev. 3 made the plan more robust, the honest answer was: yes, meaningfully — and with three caveats worth acting on rather than just stating. This revision acts on them.

1. **Rev. 3's single most consequential change is still unvalidated.** The risk-model band table (TRD §9.2) — the thing that makes "risk-ranked work order" mean something — encodes a domain judgement (a methane leak outranks corrosion at equal confidence) that no document revision can confirm. It has now sat across two revisions as a "recommended default, pending stakeholder confirmation," with the confirmation itself never actually scheduled.
2. **The review chain is not an independent check.** Rev. 2 reviewed Rev. 1; Rev. 3 reviewed Rev. 2; each pass was made by a reviewer of the same general kind as the one before it. That's a legitimate way to find missing rules and untyped fields — Rev. 3 found real ones — but it is not a substitute for the human sign-off the documents themselves keep asking for. Saying so plainly, once, is worth more than another pass that quietly repeats the pattern.
3. **The resourcing conflict is the largest unquantified risk in the plan and the only one with no fallback.** The license risk has a defined fallback (drop the Kaggle set) with a Week 1 deadline. The parallel LeRobot commitment — flagged in Rev. 2, kept in Rev. 3, still unconfirmed — has neither. A risk that could halve the team's effective capacity was being tracked with less rigor than a dataset license.

None of these are architecture problems, so none of them belong in another `[Rev. 5]` pass at that level. They're process problems, and the fix for a process problem is a process, not more specification.

## 2. The changes, and the reasoning for each

### 2.1 Consolidated the six open decisions into one Week 1 sync

**Where:** PRD §9c (new)

**What was there:** six items (PRD §6, items 2/3/7/8/9/10) each carrying a recommended default and a "needs confirmation" flag, scattered across §6, §9a, and TRD Appendix B, each with its own deadline (end of Week 1, before Week 3, before Week 4).

**Why it needed fixing:** scattered deadlines are individually easy to let slip one more week. A default that ships because nobody said otherwise reads, in the final artifact, exactly like a default that was confirmed — there's no way to tell the two apart after the fact, which defeats the purpose of flagging them at all.

**What Rev. 4 does:** one scheduled 30–45 minute conversation in Week 1, after tooling setup but before Week 2 commitments, with a named owner per item and a written outcome — confirmed, adjusted, or explicitly deferred with a new date. This is a scheduling change, not a content change: every recommended default in the table was already written down in Rev. 2 or Rev. 3. The only thing new is that "pending" now has to become something else on a specific day.

### 2.2 Gave the resourcing risk a fallback

**Where:** PRD §9d (new)

**The problem:** TR-4.1's license gate is a genuine go/no-go with a stated fallback. The resourcing risk (PRD §10) is at least as consequential — if the same two people are running a parallel pilot at a similar cadence, effective capacity could be roughly half of what the 8-week schedule assumes — and had no fallback at all. It was named as a risk in Rev. 2, named again in Rev. 3, and left exactly as open both times.

**What Rev. 4 does:** if resourcing isn't confirmed at the Week 1 sync (§9c) by end of Week 1, the plan explicitly re-baselines against the lower capacity bound already named in §6 (assumption 3) and re-checks the Week 3 / Week 6–7 pressure points (TRD §13a) against it at the Week 1 stakeholder review — rather than finding out the schedule was built on an optimistic assumption when Checkpoint 1 is already at risk.

I want to be clear about what this does and doesn't do: it does not resolve the resourcing conflict. Only the project owner's actual answer does that. It gives the plan a defined behavior for the case where that answer is late, instead of silently assuming the best case, which is the same thing the license gate already does for licensing.

### 2.3 Added a guardrail against Rev. 3's own rigor expanding

**Where:** TRD §11 ("Pattern discipline"), §14, §16

**The problem:** Rev. 3 introduced real, justified rigor — a validated contract, versioned artifacts, a persistence manifest, a risk-model table, two smoke-test assertions — each earning its cost against a 32–48 hour/person budget. That same budget doesn't stretch indefinitely, and having just established that "add a bit more structure" is the right instinct in five places, the natural next move for anyone building against these documents is to add a sixth, seventh, and eighth structural layer with the same good intentions and no additional budget to pay for them.

**What Rev. 4 does:** states plainly that Rev. 3's five additions are a ceiling for this PoC, not a floor, and that any further validation layer, metadata field, or test assertion goes through the same one-paragraph decision this document uses for its own additions — not into the codebase by default because the pattern is now established.

This is a preventative change — I'm not aware of scope creep having happened yet. It's cheap to state now and more expensive to say for the first time in Week 6 after it already has.

### 2.4 Named the review chain's own limits

**Where:** TRD §1, PRD Document Control, PRD §5, §10

**Why:** this document series has a strong, consistent habit of extending its own honesty pass — Rev. 2 said "Adapt is a label, not a loop"; Rev. 3 extended that to "Act is a trigger, not a decision" and "the simulator replays, it doesn't sense." The same logic applies one level up, to the documents themselves: four revisions in, produced by successive reviews of the revision before, none of which is a domain expert or a stakeholder. Saying this once, plainly, costs a few sentences and is consistent with everything this document series has already decided is worth saying out loud.

## 3. What I deliberately did **not** change

- **Every technical decision from Rev. 3** — the risk-model band table, the widened contract, the model-artifact convention, the persistence plan, the two smoke-test assertions. I did not re-derive or second-guess any of it. Independently verifying the architecture would mean actually building against these interfaces; that's Weeks 2–7, not a fourth document pass.
- **The specific recommended defaults in PRD §6.** I did not pick new answers for the quality bar, the taxonomy, or the risk-model bands. Doing so would repeat exactly the mistake this revision exists to fix — an AI review substituting its own judgement for a domain stakeholder's.
- **Rev. 2's leakage discipline, the descope plan, the license gate, the single smoke script, the 8-week structure.** Same reasoning Rev. 3 gave for leaving these alone: correct, and not this revision's job to touch.

## 4. What "Final" means here, and what it doesn't

"Final" describes the document, not the plan's validation status. Four things remain genuinely open after this revision — the quality bar, the taxonomy, the risk-model bands, and the `recommended_action` wording all need a stakeholder; the resourcing conflict needs the project owner; the license outcome needs Deloitte OSS/legal (all six listed in TRD Appendix C). None of them close by writing a fifth revision. They close by the Week 1 sync (PRD §9c) actually happening.

What changes starting Week 3 is that the plan can fail cleanly instead of ambiguously: either the sync happened and there's a recorded answer to build against, or it didn't and the resourcing gate (§9d) has already re-baselined the schedule rather than letting the gap surface for the first time at Checkpoint 1.

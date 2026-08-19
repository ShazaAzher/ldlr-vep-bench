# LDLR-VEP-Bench

**Mechanism-Stratified Benchmarking of Structural Variant Effect Predictors
Against Deep Mutational Scanning Data in LDLR**

## One-line pitch
Does any current AI variant-effect predictor actually distinguish *why*
an LDLR mutation is pathogenic — misfolded/ER-retained vs. binding-defective
vs. recycling-defective — or do they all just detect "something's wrong"
without capturing mechanism? We test this against the first near-complete
experimental map of LDLR variant function (published Feb 2026), which no one
has yet used to ask a mechanism-level question.

## Why this is worth doing (and why now)
- A Feb 2026 *Science* paper (Roth lab) deep-mutational-scanned ~17,000 —
  essentially all possible — LDLR missense variants, measuring both cell-surface
  abundance (trafficking/misfolding proxy) and LDL uptake (function). This is
  now the field's gold-standard ground truth, and it's brand new.
- A 2023 Mayo Clinic paper showed vanilla AlphaFold2 structures poorly predict
  LDLR variant pathogenicity, specifically because AF2 collapses LDLR's real
  open/closed pH-dependent conformational switch into a single guess — a named,
  citable, mechanistic failure mode, not a vague "AI struggles with this."
- General-purpose structure-based equivariant predictors (PreMode, HERMES) exist
  and are state-of-the-art, but have not been benchmarked against this new LDLR
  map, and were never evaluated for whether they separate *mechanism*, only
  whether they separate pathogenic from benign.
- Nobody has yet combined all three. That gap is the paper.

## What this is NOT
Not a new architecture. Not a cohort study. Not wet lab. This is systematic,
honest benchmarking — real science, modest scope, solo- and free-GPU-feasible.

## Two possible endings (both are a paper)
1. **Existing tools already work fine mechanism-wise** → useful negative/positive
   result for clinicians deciding which predictor to trust for LDLR variant
   triage, and a clean methods note.
2. **Existing tools conflate mechanisms** (plausible given none were trained on
   mechanism labels) → motivates a light fine-tune of PreMode's existing
   transfer-learning head on the new DMS abundance labels — a modest, defensible,
   *use of an existing framework as intended*, not a novel-architecture claim.

## Status
- [x] Project scaffold
- [x] Hand-curated mechanism-labeled variant set (29 variants, 2 papers,
      Class 2a/2b/3/4/5 + benign, with structural mechanism notes) —
      `data/ldlr_functional_variants_batch1.csv`
- [ ] Pull the Roth lab DMS dataset (MaveDB or Science supplement)
- [ ] Pull AlphaFold DB precomputed structure for LDLR (UniProt P01130)
- [ ] Score AlphaMissense, ESM, EVE, PreMode, HERMES on the full DMS variant set
- [ ] Domain- and mechanism-stratified correlation analysis
- [ ] Decide: write up as-is, or fine-tune PreMode first
- [ ] Draft manuscript

## Repo layout
- `data/` — DMS dataset, hand-curated mechanism labels, AlphaFold structure
- `src/` — scoring scripts (one per VEP tool), analysis/stats code
- `notebooks/` — exploratory analysis, figures
- `refs/` — key papers, PDB/AlphaFold structure files
- `paper/` — manuscript draft

---

## Task Breakdown

### Phase 0 — Data acquisition (no GPU needed)
1. Locate and download the Roth lab DMS dataset (check MaveDB first, then
   *Science* supplementary materials) — need per-variant abundance score
   and LDL-uptake score for all ~17,000 variants.
2. Download precomputed AlphaFold DB structure for LDLR (UniProt P01130) —
   no need to run AlphaFold locally.
3. Cross-reference the hand-curated 29-variant mechanism-labeled set against
   the DMS variant list by HGVS/position, so every hand-labeled variant has
   both a mechanism class *and* a DMS abundance/uptake score. This small,
   doubly-annotated set is what makes mechanism-level analysis possible later,
   since the DMS map alone only gives two continuous readouts, not a
   mechanism label.
4. Expand the hand-curated set opportunistically (Graça 2023 high-throughput
   panel is the next source) — not blocking, but strengthens the mechanism
   cross-reference.

### Phase 1 — Score existing predictors (mostly no GPU, PreMode/HERMES need one)
5. Pull AlphaMissense's precomputed score for every LDLR variant (whole-proteome
   predictions are downloadable — no inference needed).
6. Run ESM zero-shot scoring (ntranoslab/esm-variants repo) on the full variant
   set — CPU-feasible or light GPU.
7. Pull or reproduce EVE scores (evemodel.org has an existing LDLR model).
8. Set up free-GPU environment (Colab), clone PreMode (ShenLab/PreMode), run
   its pretrained model zero-shot on the LDLR variant set.
9. Same for HERMES — clone, run pretrained zero-shot.
10. Consolidate all scores into one table keyed by variant, alongside the two
    DMS readouts (abundance, uptake) and the mechanism label where available.

### Phase 2 — Benchmark analysis
11. Global correlation (Spearman) of each predictor's score against DMS
    abundance and DMS uptake, separately — replicates and extends the Mayo
    paper's approach but against the far larger, near-complete dataset.
12. Stratify correlations by structural domain (LBD, EGF-A, EGF-B,
    beta-propeller, EGF-C, O-linked, TM, cytoplasmic).
13. Mechanism-level test on the doubly-annotated subset: does any predictor's
    score distinguish low-abundance/low-uptake (misfolding/ER-retention
    signature) from normal-abundance/low-uptake (binding-defective signature)?
    This is the actual novel question — nobody has asked it against this data.
14. Targeted check on the AF2 hinge region the Mayo paper flagged (residues
    ~185–196 and ~394–399) — do AF2-derived predictors (PreMode, AlphaMissense)
    specifically underperform there relative to sequence-only models (ESM, EVE)?

### Phase 3 — Decision point
15. Review Phase 2 results together and decide: write up the benchmark as-is,
    or attempt a light PreMode fine-tune on DMS abundance labels first (their
    repo's transfer-learning workflow supports this directly).
16. If fine-tuning: train, re-run Phase 2 analysis on the fine-tuned model,
    compare.

### Phase 4 — Write-up
17. Figures: global + domain-stratified correlation heatmap, mechanism
    separation plot, hinge-region case study.
18. Draft manuscript, case-study/applications track framing (similar tier to
    [[drugsense]]).
19. Identify target journal/venue.

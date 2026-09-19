# COMP 577 — Midterm 1 Prep Notes

**From:** exam-prep discussion in the September 15, 2026 lecture ("Midterm exam prep — filtering, Harris corner detector, SIFT matching, and ransac").

## When

Next Tuesday (i.e. **September 22, 2026**, based on the Sep 15 lecture date).

## Format

- **4 questions, 10 points each (40 total).**
- Not multiple choice — proper written math questions, same style as the practice midterm and homework. Many questions have **subparts, graded separately**, so partial credit is available throughout.

### Question breakdown

| # | Topic | Difficulty |
|---|---|---|
| 1 | Filtering | Easy — covered in class + homework |
| 2 | Harris corner detector properties | Easy — covered in class + homework |
| 3 | SIFT feature matching: true positive rate, false positive rate, threshold selection, and RANSAC | Easy/medium — covered in class |
| 4 | "Thought problem" — partly maps to learned content, partly general reasoning/creativity | Hard — novel framing, not seen before |

**On Question 4:**
- Not testing new material to learn — one part maps to something already covered, the other part is closer to general/creative reasoning ("common sense"), not something you look up.
- Don't expect to fully solve it — last year only ~3–4 students solved it completely. Everyone will be in roughly the same boat.
- **Attempt it anyway** — even a partial/reasonable attempt earns some credit (e.g. 2–3 / 10), vs. zero for leaving it blank.
- Doesn't require pages of work — the solution itself is short once you see the right angle; most of the "work" is in reading/understanding the setup.

## Grading Philosophy

- **Partial credit is generous, but only if the professor can follow your steps.** A wrong final answer with correct/visible steps (e.g. an arithmetic slip) can still score very high (e.g. 9/10).
- **Show your steps clearly and write legibly** — illegible handwriting makes partial credit hard to award.
- Don't skip write-up to save time — the process matters more than just the final number.

## What to Study

- Follow the **"recap" sections in each lecture's slides** — these are explicitly the topics considered fair game. Anything the professor flagged as "mentioned for general knowledge, not exam material" is out of scope.
- The **practice midterm** (~30 questions) is the best guide for what's actually testable, and for gauging **how deep the math needs to be** vs. what's more conceptual (e.g. "given this scenario, which filter would you choose and why" vs. a full derivation).
- Homework problems and practice-midterm questions are a strong predictor of real exam questions — expect **interpolations** of these, not identical numbers/wording, but same underlying method.
- Not simply a copy of a prior semester's midterm, though there will be topical similarity.

### Math depth expected

- Basic linear algebra (matrix multiplication, etc.) is needed.
- You do **not** need to hand-derive eigenvalue/SVD computations in detail.
- You **do** need to understand *when* to apply which solution method conceptually — e.g., recognizing an `Ax = b` (least-squares) setup vs. an `Ax = 0` (null-space / smallest-eigenvector) setup, and why (see [2026-09-15-ransac-and-blending.md](2026-09-15-ransac-and-blending.md) for the affine-vs-homography version of this).
- Partial derivatives come up (e.g. Harris corner detector proofs) but nothing more advanced than what's already been used in class.

## Logistics / Rules

- **One cheat sheet allowed**, both sides, any content, handwritten or (micro)printed — use it, and use it well.
- **No electronics** of any kind: no phone, no smartwatch, no iPad/tablet. Electronics must stay at your seat even during a restroom break — you're not searched, but the expectation is explicit.
- Time budgeting suggestion: **~20 minutes each** for questions 1–3 (≈1 hour total), leaving the remaining time (plus any buffer) for question 4.

## Historical Score Distribution

- Not a bell curve — historically a **wide/uniform-ish spread**:
  - Some students score 35–39/40.
  - A large cluster lands around **22–28/40**.
  - Some students score below 10/40.
- Scores tend to **improve on the second midterm** once students know the format (average has historically moved from ~24–25 up to ~28–30 on exam 2).

## Study Strategy (professor's suggestion)

1. Solve the first 3 (easier) questions first; only move to Q4 after.
2. Work through the ~30 practice-midterm questions.
3. When checking your work with an AI tool, **verify your steps, not just the final answer** — it's possible to get a correct final number with flawed reasoning.
4. If stuck, ask an AI tool for a **hint** first rather than a full solution, and try again before asking for the complete answer.

## Topic Checklist by Question

*(My own mapping of the professor's question list onto the notes; check each item against the slides' recap sections.)*

- **Q1 — Filtering.** No lecture recordings exist before Sep 3, so these notes have no filtering coverage from class. See [filtering-primer.md](filtering-primer.md) (general background, not from lecture) and rely on the slides and the practice midterm. Related items that did come up: derivative-of-Gaussian filters for gradients (Sep 3), Gaussian vs. Laplacian pyramids (Sep 15), and "filtering changes intensity, warping changes coordinates" (Sep 8).
- **Q2 — Harris corner detector properties.** `H` and its eigenvalue interpretation; response function; the three proofs (photometric, translation, rotation); why scale breaks invariance. All in [2026-09-03-corner-detection-sift.md](2026-09-03-corner-detection-sift.md), including a step-by-step rotation proof and a cheat-sheet table.
- **Q3 — Matching evaluation and RANSAC.** TPR/FPR calculation; choosing a threshold for a target FPR; ROC/AUC; ratio test; RANSAC steps, sample sizes `s`, and the trial-count formula (derivation not required). See [2026-09-08-evaluation-and-image-warping.md](2026-09-08-evaluation-and-image-warping.md) and [2026-09-15-ransac-and-blending.md](2026-09-15-ransac-and-blending.md).
- **Also fair game (professor-flagged):** `Ax = b` vs. `Ax = 0` and how many point pairs each transform needs; why parallel lines are not preserved by a homography; homogeneous-coordinate facts (`w = 0` is a point at infinity, `(0,0,0)` is not allowed).

## Practice Problems With Worked Solutions

*(Written by me for practice, in the style the professor described. They are not the professor's questions.)*

**1. Photometric change.** `J = 2·I + 5`. Show how `H` changes, and by what factor the threshold on `R = det(H) − k·trace(H)²` must change for the detector to return the same corners.
- `Jx = 2·Ix`, `Jy = 2·Iy` (the `+5` vanishes under differentiation), so `H_J = 4·H_I`.
- Eigenvectors unchanged; eigenvalues × 4. `det` scales by `4² = 16` and `trace²` by `16`, so `R` scales by **16** (`a⁴` with `a = 2`). The threshold must be multiplied by 16.

**2. TPR / FPR / precision.** A matcher returns 1000 matches; the oracle says 800 are good and 200 are bad. After thresholding, 650 remain, of which 560 are good.
- `TP = 560`, `FP = 90`. `TPR = 560/800 = 0.70`, `FPR = 90/200 = 0.45`, precision `= 560/650 ≈ 0.86`.

**3. Choosing a threshold.** Ten matches, sorted by distance, with oracle labels: `0.1 G, 0.2 G, 0.3 G, 0.4 B, 0.5 G, 0.6 B, 0.7 G, 0.8 B, 0.9 B, 1.0 B` (G = good, B = bad; 5 of each). Which threshold gives the highest TPR while keeping FPR ≤ 0.2? (Keep matches with distance ≤ threshold.)
- `t = 0.3`: TP 3, FP 0 → TPR 0.6, FPR 0.
- `t = 0.5`: TP 4, FP 1 → TPR 0.8, FPR 0.2. ✔
- `t = 0.7`: TP 5, FP 2 → FPR 0.4, too high.
- Answer: any `t` in `[0.5, 0.6)`; TPR = 0.8.

**4. RANSAC trials.** A homography is fit to matches with a 30% outlier ratio. How many trials for 99% success?
- `s = 4`, `e = 0.3`: `n = ln(0.01) / ln(1 − 0.7⁴) = −4.605 / −0.2746 ≈ 16.8 → 17`.

**5. Minimum matches.** How many point pairs are needed for (a) translation, (b) affine, (c) homography? → **1, 3, 4** (2, 6, 8 degrees of freedom, 2 equations per pair).

**6. Composition.** Apply "rotate 90° counter-clockwise, then translate by (2, 0)" to `(1, 0)`. → `(2, 1)`. Doing it in the opposite order gives `(0, 3)`, so order matters (worked out in the Sep 8 notes).

**7. Parallel lines.** Explain why an affine map preserves parallelism but a homography does not. → Parallel lines meet at a point at infinity `(x, y, 0)`. An affine map keeps `w = 0`; a homography produces `w = gx + hy ≠ 0`, moving that point to a finite vanishing point.

## Known Gaps In These Notes

- **Filtering:** no lecture source (see above).
- **Question 4:** by design it is a new "story" problem, so there is nothing to study directly. Practice explaining, in writing and with steps, how a concept you know (e.g. thresholds, outliers, invariance) applies to an unfamiliar scenario.
- **Slides and practice midterm:** these notes come from audio transcripts, so equations the professor wrote on slides may be described imprecisely. Cross-check formulas against the slides.

## Related Class Material

Full lecture notes referencing this exam's topics:
- [2026-09-03-corner-detection-sift.md](2026-09-03-corner-detection-sift.md) — Harris corner detector, invariance/equivariance
- [2026-09-08-evaluation-and-image-warping.md](2026-09-08-evaluation-and-image-warping.md) — TP/FP rates, ROC/AUC, linear/affine/homography transforms
- [2026-09-15-ransac-and-blending.md](2026-09-15-ransac-and-blending.md) — solving for affine/homography transforms, RANSAC, panorama blending

**Not on Midterm 1:** the Sep 17 lecture ([2026-09-17-image-classification-fundamentals.md](2026-09-17-image-classification-fundamentals.md)) starts the next section; the professor said it is for the next exam. (Its ROC/TP/FP evaluation ideas do overlap with Question 3 material from Sep 8.)

## Support

- Professor's office hours: **Thursday**.
- TA's extra office hour: **Friday** (primarily for assignment help, but exam questions welcome too).

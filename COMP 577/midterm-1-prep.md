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

## Related Class Material

Full lecture notes referencing this exam's topics:
- [2026-09-03-corner-detection-sift.md](2026-09-03-corner-detection-sift.md) — Harris corner detector, invariance/equivariance
- [2026-09-08-evaluation-and-image-warping.md](2026-09-08-evaluation-and-image-warping.md) — TP/FP rates, ROC/AUC, linear/affine/homography transforms
- [2026-09-15-ransac-and-blending.md](2026-09-15-ransac-and-blending.md) — solving for affine/homography transforms, RANSAC, panorama blending

## Support

- Professor's office hours: **Thursday**.
- TA's extra office hour: **Friday** (primarily for assignment help, but exam questions welcome too).

# COMP 577 — Solving for Transformations, RANSAC, and Panorama Blending

**Date:** September 15, 2026

> Midterm 1 exam-prep/logistics discussion from this same class is kept separately in [midterm-1-prep.md](midterm-1-prep.md).

## Recap: Solving for Affine vs. Homography Transformations

Two ways of setting up the estimation problem, depending on transformation type:

### Affine transformation — least-squares (`Ax = b`)

- Affine transform has **6 unknowns** (degrees of freedom): `a, b, c, d, e, f`.
- Each matching point pair `(x, y) ↔ (x̂, ŷ)` gives **2 equations**.
- So you need **at least 3 point pairs** (3 × 2 = 6 equations) to solve for all 6 unknowns.
- Stack all equations into matrix form `Ax = b`, where `A` encodes the known left-image points, `b` encodes the known right-image points, and `x` is the unknown parameter vector.
- Solved via ordinary **linear least squares**: `x = (AᵀA)⁻¹Aᵀb`.

### Homography transformation — null-space solution (`Ax = 0`)

- Homography has **8 unknowns** — can't cleanly separate `x`/`y` onto opposite sides of the equation the way affine allows.
- Each point pair still gives 2 equations, so you need **at least 4 point pairs** (4 × 2 = 8 equations).
- Stack into one large matrix equation `Ax = 0`. The matrix `A` has rank 8 in the minimal case, and the solution is the **null space** of `A`.
- In practice: the optimal solution is the **singular vector corresponding to the smallest singular value of `A`** (via SVD), equivalently the **eigenvector corresponding to the smallest eigenvalue of `AᵀA`**. Eigen-decomposition is typically used in practice since it's faster.
- **You don't need to reproduce every step of this derivation** — what matters is recognizing *which* formulation (`Ax = b` vs `Ax = 0`) applies to a given problem, and why.

### When to use affine vs. homography

- **Homography** is the correct transformation between two images of the same **planar (2D) scene captured from different positions in 3D space** — i.e., when the camera itself moves/rotates such that its image plane is not the same between the two shots.
- **Affine** transformations suffice when the camera stays fixed in the same plane and the images only differ by rotation, translation, or scaling of the image itself — no true change in 3D viewpoint.
- If the scene being imaged is not consistent between the two shots (e.g. the physical thing photographed changes), homography does not apply either — it strictly models the mapping between two 2D planes embedded in 3D.

## Recap: Homogeneous Coordinates

- A homogeneous coordinate `(x, y, w)` maps back to the 2D point `(x/w, y/w)`.
- `(x, y, 0)` represents a **point at infinity**.
- `(0, 0, 0)` is **not a valid/allowed** homogeneous coordinate.
- Homography transformations do **not** preserve properties that affine transformations do (e.g. parallel lines remaining parallel) — proving this (a fairly simple linear-algebra argument) is expected exam material. See [midterm-1-prep.md](midterm-1-prep.md) for what's fair game.

## RANSAC (Random Sample Consensus)

A general statistical method for fitting a model in the presence of outliers — not specific to computer vision.

### Core algorithm

1. Randomly choose the **minimum number of samples `s`** needed to fit the model:
   - Line fit: `s = 2` points
   - Translation estimate: `s = 1` point pair (translation has 2 DOF, and one point pair gives 2 equations)
   - Affine transformation: `s = 3` point pairs
   - Homography transformation: `s = 4` point pairs
2. Fit the model to those `s` samples.
3. Count **inliers**: points that fit the model within some error threshold (a design choice).
4. Repeat steps 1–3 for `n` trials.
5. Keep the model/sample-set with the **largest inlier count**.
6. **Recompute the final model using all inliers** from the winning round (not just the original minimal sample) — this refines the estimate since it now uses more data than the minimal fitting set.

### Worked example (translation from noisy matches)

Given 7 matched point pairs (5 good, 2 bad, though we don't know which up front):

- Since translation only needs 1 point pair, take one pair, compute the implied translation, then check how many of the other pairs are consistent with (close to) that translation.
- A "lucky" draw from the good pairs yields 4 inliers + itself; a draw involving a bad pair yields few/no consistent inliers.
- After enough trials, the best-scoring sample set is kept, then the translation is recomputed using **all** inliers found in that round for the final answer.

### Inlier test for homography specifically

1. Estimate a homography from 4 randomly chosen point pairs (via the `Ax = 0` / smallest-eigenvector approach above).
2. For every other candidate point pair, apply the estimated homography to the left-image point and compare it to the actual matched right-image point.
3. Compute the **pixel distance** between the transformed point and the actual match.
4. A pair is an inlier if this distance is below a threshold (e.g., ~2–3 pixels for a 500×500 image — threshold choice is scene/resolution dependent).

### How many trials are needed?

There's a formula guaranteeing a given success probability `p`, based on:
- `s`: minimum sample size for the model (2 for line, 4 for homography, etc.)
- `e`: the outlier ratio in the data

Intuition for the derivation: to fail, *every* one of the `n` trials must include at least one outlier. Since trials are independent, the probability of failure is `(probability of ≥1 outlier in one trial)ⁿ`. The probability of drawing **zero** outliers in one trial (`s` independent draws) is `(1-e)ˢ`, so the probability of at least one outlier in a trial is `1 - (1-e)ˢ`. Setting overall failure probability ≤ `1-p` and solving for `n` gives the required number of trials. (Full derivation is **not exam material**, per the professor.)

**Example values (homography, `s = 4`, target 99% success):**
- 10% outlier ratio → only ~5 trials needed.
- 50% outlier ratio → ~72 trials needed.
- Required trials grow quickly with both outlier ratio and model complexity (`s`), and grow further if you require a higher success probability (e.g. 99.99%).

### Why RANSAC works: intuition

> "All good matches are alike; every bad match is bad in its own way."

Inliers all cluster around and support one consistent model (e.g. one true translation/homography), while outliers are essentially independently, randomly distributed with no shared structure. The proof relies on an assumption of **conditional independence** among outliers — there's no correlation between how different outliers are wrong. If outliers weren't independent (e.g. they formed their own consistent competing model), RANSAC could fail to find the correct answer.

### Pros / cons

- **Pros:** simple, very general — applicable well beyond computer vision — and works well in practice.
- **Cons:**
  - Requires guessing/tuning the number of trials, which technically depends on the (usually unknown) outlier ratio — in practice, people often just pick a large fixed number (e.g. 1000) without a principled basis.
  - **Breaks down when outlier ratio exceeds ~50%** — doesn't reliably converge.

## Panorama Blending

After aligning two images via homography, naive stitching produces a visible seam/line at the boundary.

### Simple weighted blending

- Define a **weight function** based on each pixel's distance from the stitch boundary:
  - Far into one image → trust that image ~100%.
  - Far into the other image → trust it ~100% (i.e. weight → 0 for the first image).
  - Near the boundary → blend the two images proportionally.
- The transition slope matters: too gradual → blurry combined image; too abrupt → visible seam remains.
- In the assignment, the weighting is driven by a provided **distance-threshold function** — you don't need to hand-design the weight curve, but you do need to understand what it's doing.

### Weight normalization

For each pixel, if `w1` and `w2` are the raw weights from image 1 and image 2, the **normalized weights must sum to 1**:

```
final_weight_1 = w1 / (w1 + w2)
final_weight_2 = w2 / (w1 + w2)
```

Skipping normalization distorts pixel intensity, since you'd be summing partial intensities that don't add up to a full-intensity blend.

### Stitching order for multiple images

When combining more than 2 images:

1. Arrange images in **capture order** (first, second, third, …).
2. Merge **adjacent pairs**: (1,2) → one image, (2,3) → one image, (3,4) → one image, etc. This reduces `N` images to `N-1`.
3. Repeat the pairwise-merge process until a single final image remains.

This keeps every homography estimate and blend between images that are visually/temporally close, which is far more stable than trying to directly match/blend the first image with the last — lighting and geometric change accumulate too much over a large image gap.

### Practical capture tips (for the assignment)

- The scene should be **roughly planar**, since homography assumes a planar scene.
- Keep **lighting consistent** across the full capture sequence.
- Avoid **large camera rotations/movements** between consecutive shots — small motions (some rotation is fine, just not drastic) give much more stable matching and blending results.
- Debugging loop: capture → check result → either **redo the capture** (if the scene/motion was the issue) or **tune thresholds/hyperparameters** (e.g. the RANSAC inlier threshold) if the capture itself was fine.

### Beyond the assignment: Laplacian pyramid blending (not required)

A more advanced blending approach used in practice:
- Build a **Gaussian pyramid** for each image (low-pass filtering at increasing scale).
- Derive the corresponding **Laplacian pyramid** (high-pass) by subtracting successive Gaussian pyramid levels.
- Blend the **low-frequency and high-frequency bands separately**, each with a potentially different weighting function, rather than one single blend weight for the whole image — this produces smoother, less visible seams than the simple linear-blend approach used in the assignment.
- Historically, manual multi-image compositing (e.g. classic Photoshop workflows — blending parts of different images together by hand) followed similar ideas; modern AI-based image blending has mostly superseded this manual approach.

## Assignment Logistics

- Due **Friday at midnight**.
- Late policy: a flat **~3-point deduction** for being late (not necessarily per day) — the professor's advice is to take the deduction rather than submit an incomplete/rushed assignment, since losing a few points is much better than a poor-quality submission.
- Office hours: professor on **Thursday** (exam prep / general questions); TA has an **extra office hour Friday** specifically for assignment help.

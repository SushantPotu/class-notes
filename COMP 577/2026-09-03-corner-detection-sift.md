# COMP 577 — Corner Detection, Invariance/Equivariance, and SIFT

**Date:** September 3, 2026

## Overview

Panorama stitching relies on several steps; this lecture (and the previous one) focus on the first two:

1. **Detection** — finding salient feature points ("corners")
2. **Description** — describing the local appearance around each point
3. **Matching** — matching points between images (only lightly covered)

Detection is the hardest of the three; description and matching are comparatively straightforward.

## Recap: Defining a Corner Mathematically

A **corner** is a location where sliding a small window `w` over the image produces a large change in image content *in every direction*.

- **Flat region:** no change when the window moves in any direction.
- **Edge:** change when moving perpendicular to the edge, but not when moving along it.
- **Corner:** change in *all* directions.

### Error function

Sliding the window by `(u, v)` gives an error function measuring how much the windowed image content changes. Computing this directly for every pixel and every shift is computationally prohibitive, so we approximate it with a **Taylor series expansion**, which expresses the error in terms of image gradients.

This lets us rewrite the error as a **quadratic form** in `(u, v)`:

```
E(u, v) ≈ [u v] · H · [u v]ᵀ
```

where `H` (the **second moment matrix**, sometimes called the Harris matrix) is built from sums of gradient products over the window:

- `a = Σ Ix²`
- `b = Σ Ix·Iy`
- `c = Σ Iy²`

### Geometric interpretation: generalized ellipse

The quadratic form `E(u,v) = a·u² + 2b·uv + c·v²` is the equation of a **generalized (rotated, stretched) ellipse** — as opposed to an axis-aligned ellipse (`x²/a² + y²/b² = 1`).

- **Eigenvectors** of `H` determine the **rotation** (axes) of the ellipse.
- **Eigenvalues** of `H` determine how much the ellipse is **stretched** along each axis (stretch ∝ `1/√λ`).

Interpreting the two eigenvalues `λ_max` (`λ1`) and `λ_min` (`λ2`):

| Case | Condition | Region type |
|---|---|---|
| Edge | one eigenvalue ≫ the other (e.g. ~100×) | edge |
| Flat | both eigenvalues small | flat region |
| Corner | both eigenvalues large and roughly similar | corner |

### Avoiding explicit eigenvalue computation

Computing actual eigenvalues per-pixel is expensive. Instead, use functions of the **determinant** (`det = λ1·λ2`) and **trace** (`trace = λ1 + λ2`) of `H`, both trivial to compute for a 2×2 matrix:

- `R = det(H) − k·trace(H)²` (Harris response), or
- `f = det(H) / trace(H)`

A pixel is a corner if the response exceeds a chosen **threshold**:
- High threshold → few but very reliable corners.
- Low threshold → many corners, more false positives.
This is a tunable hyperparameter depending on the application/scene.

**Reminder:** `H` always includes the summation over the window — don't drop it when doing proofs; the math becomes wrong (though the eigen-structure argument doesn't otherwise change) if you treat `H` as just `[Ix, Iy]` without the sum.

### Algorithm summary (Harris corner detection)

1. Compute image gradients `Ix`, `Iy` (via derivative-of-Gaussian filters).
2. Compute `Ix²`, `Iy²`, `Ix·Iy` at each pixel.
3. Sum these over a window (convolve with an all-ones filter) to get `a`, `b`, `c` → assemble `H`.
4. Compute `det(H)` and `trace(H)` (avoid explicit eigen-decomposition).
5. Compute response `R` (or `f`).
6. Threshold `R` to decide which points are corners.

## Properties: Invariance vs. Equivariance

- **Invariance:** the detected point stays at the *exact same location* after a transformation.
- **Equivariance (covariance):** the detected point *moves consistently* with the transformation (e.g., rotates by the same angle as the image).

Two transformation types:
- **Geometric transformation** — changes the *domain* (pixel coordinates), e.g. translation, rotation.
- **Photometric transformation** — changes the *range* (intensity values), e.g. contrast/brightness change.

### Results (Harris detector)

| Transformation | Property |
|---|---|
| Translation | Invariant |
| Rotation | Equivariant |
| Intensity change | Invariant *if threshold is scaled accordingly* |
| Scale | **Neither** invariant nor equivariant |

### Proof sketch: photometric invariance

Let `J(x,y) = a·I(x,y) + b` (contrast/brightness change).

- `Jx = a·Ix`, `Jy = a·Iy` (the additive constant `b` disappears under differentiation).
- Second moment matrix: `H_J = a² · H_I`.
- Eigenvectors of `H_J` = eigenvectors of `H_I` (unchanged); eigenvalues scale by `a²`.
- Since corner classification depends on the *relative* relationship between eigenvalues, the same points are still detected as corners — **as long as the threshold is also scaled by `a²`**. If the threshold is left unchanged, previously sub-threshold points may cross it and be falsely detected.

### Proof sketch: geometric equivariance

Let the geometric transform map `(x,y) → (u,v)`.

- **Translation:** `∂x/∂u = 1`, `∂y/∂u = 0` (and similarly for `v`) ⇒ `Ju = Ix`, `Jv = Iy` ⇒ `H_J = H_I` exactly. Fully invariant.
- **Rotation:** using the chain rule, `H_J = Rᵀ · H_I · R` (`R` = rotation matrix). Since `R` is orthogonal, this means the eigenvectors of `H_J` are the eigenvectors of `H_I` rotated by `R`, while the **eigenvalues are unchanged**. Since eigenvalues determine whether a point is a corner, and eigenvectors just rotate, corners rotate consistently with the image — i.e., **equivariant**, not invariant. (Full derivation is a good chain-rule practice problem; the rotation proof is longer than the translation proof and won't appear as-is on the exam, but similar reasoning will.)

### Scale: not invariant/equivariant

A "corner" depends on the window size relative to the feature. A curve viewed with a large window can look like a corner; the same curve stretched out (or viewed with a small window) can look like a straight edge. So Harris corner response at a **fixed scale** is not reliable across scale changes — the response depends on what window size is used.

## Scale-Space: Finding the Characteristic Scale

Instead of outputting just `(x, y)` corner locations, we want to output **`(x, y, scale)`** triplets — the scale at which each point is "most" a corner.

**Naive approach:** for each pixel, compute the Harris response across a full sweep of window sizes `w`, and pick the scale at which the response is maximized. This is correct but computationally very expensive.

### Laplacian of Gaussian (LoG)

A filter whose scale (`σ`) can be varied to compute this response efficiently, without recomputing the full Harris pipeline per scale.

### Difference of Gaussians (DoG)

Subtracting two Gaussian-blurred versions of the image (at different `σ`) numerically **approximates** the Laplacian of Gaussian, and is much cheaper to compute. (Note: "DoG" is *difference* of Gaussians, not a derivative.)

## SIFT (Scale-Invariant Feature Transform)

One of the most influential CV papers (~2004), still practically used today (e.g. on small robots/drones) due to its low compute cost relative to deep-learning-based features.

### Detection via DoG + Octaves

1. Apply a sequence of Gaussian blurs with increasing `σ` to the image.
2. Take differences between adjacent blurred images → Difference of Gaussian.
3. For each pixel, find the scale at which the DoG response is maximized (and above a threshold) → that's the point's **characteristic scale**.
4. Rather than sweeping `σ` over a huge range directly, SIFT **downsamples the image by 2×** periodically (down-sampling approximates increasing `σ`) and repeats the Gaussian sweep — each round is called an **octave**.

This lets you compute the same scale-response concept as brute-force multi-scale Harris, but much faster — a good example of approximating a theoretically-motivated but slow algorithm with a fast, numerically stable engineering solution (mirrors how modern ML algorithms are validated on small tractable cases and assumed to approximately generalize).

### SIFT Descriptor

For each detected keypoint:

1. Take a 16×16 patch around the point.
2. Compute gradients `Ix`, `Iy` at each pixel in the patch, and gradient orientation `atan(Iy / Ix)`.
3. Discard/downweight low-magnitude gradients (flat, low-contrast areas).
4. Build a **localized histogram of gradient orientations** over the patch.
5. Concatenate into a **128-dimensional feature vector** describing the region around the point.

**Why a histogram works well:** since it aggregates gradient orientation *distribution* rather than exact per-pixel values, it is:
- Invariant to translation/rotation/intensity shifts (same proof style as detector invariance).
- Robust to moderate 3D viewpoint (camera rotation) changes.
- Robust to moderate lighting changes (localized shadow movement doesn't change the overall distribution much).

### Output of SIFT

For an image, SIFT outputs `k` keypoints (k depends on threshold), each with:
- `(x, y)` location
- characteristic scale
- 128-dim descriptor vector

## Other descriptors

HOG (Histogram of Oriented Gradients) and similar methods follow similar ideas.

**Learned alternatives:** SuperPoint / SuperGlue (neural-network-based detection/matching) perform much better than SIFT under **large geometric changes** between images, but are computationally expensive (need a GPU). SIFT-like features remain preferred on small/embedded systems (drones, microcontrollers) where compute is limited, since they run fast on a CPU.

## Feature Matching

Given SIFT points + descriptors independently computed on two images, how do we match corresponding points?

### Brute-force matching

For each point's descriptor in image A, compute similarity/distance to every descriptor in image B; pick the best match. Simple but slow (exhaustive).

### k-Nearest-Neighbor + Ratio Test

Matching purely on "best similarity" is noisy in repetitive scenes (e.g. many similar-looking poles/pillars). Instead:

1. Find the **top 2** nearest matches (k=2) for each descriptor.
2. Apply the **ratio test**: only accept the match if the best match is at least ~25% better (smaller distance) than the second-best. (25% is a tunable choice — increase strictness for scenes with more repetitive structure.)
3. If the match is ambiguous (best and second-best are close), discard the point rather than keep an unreliable match.

### OpenCV workflow (as used in the assignment)

1. Read image.
2. Run SIFT detector → keypoints (`kp`) + descriptors (`des`), independently per image.
3. Create a matcher (e.g. brute-force `BFMatcher`).
4. Use k-NN matching (`k=2`).
5. Apply ratio test to filter ambiguous matches.
6. Visualize matches as connecting lines between corresponding points in the two images.

Goal isn't thousands of matches — a smaller set of *high-accuracy* matches is preferred, since they'll be used to compute geometric relationships between images.

## Next Class: Evaluating Matching Quality

Given ground truth correspondences, how do we score how good a matching algorithm/pipeline is (e.g. SIFT vs. another detector, brute-force vs. ratio-test matching)?

This leads into standard classification-evaluation concepts:
- True positive / false positive rate
- Accuracy, recall
- ROC curve (receiver operating characteristic) / area under curve

These are the same concepts used later in the course for ML/deep-learning classification evaluation — a distance threshold on match quality effectively turns this into a binary classification problem (correct match vs. incorrect match).

## Logistics

- Practice exam questions and worked homework/exam solutions will be provided — try problems using hints first before consulting the solution.
- Office hours today from 5.

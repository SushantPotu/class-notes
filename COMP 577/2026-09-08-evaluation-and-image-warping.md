# COMP 577 — Evaluating Matches (ROC/AUC) and Image Warping (Linear, Affine, Homography)

**Date:** September 8, 2026

## Recap: Corner Detection → SIFT → Matching

- Harris corner intuition: slide a window in all directions; large change in every direction ⇒ corner. Formalized via the second-moment matrix and its eigenvalues (`λmax`, `λmin`) — a generalized ellipse whose axes are the eigenvectors and whose stretch is governed by the eigenvalues. **Max eigenvalue direction = fastest-changing direction = minor axis of the ellipse** (perpendicular to an edge).
  - Corner: both eigenvalues large.
  - Edge: one eigenvalue much larger than the other.
- Properties/proofs (translation, rotation, photometric, scale) will reappear on the practice midterm — translation is the easiest proof, rotation the hardest (unlikely to appear in full on the exam), photometric proofs are easier since they don't need the chain rule.
- **Key limitation:** corner-ness is scale-dependent — a point is only a corner *at a particular scale*. This motivated computing a corner response across a stack of scales (via Difference-of-Gaussian / Laplacian-of-Gaussian) and keeping, for each point, the scale at which the response is maximal **and** above a threshold — this is the core idea behind the SIFT detector.
- SIFT descriptor: gradient orientation indicates edge direction (perpendicular to the edge); gradients with small magnitude are discarded (noise, not real edges); build a **histogram of gradient orientations** over a local region → robust to small local shifts and lighting changes, since only the *distribution* of orientations matters, not exact intensity.
- Matching: brute-force similarity is unreliable for repetitive scenes (matches everything similarly against repeated patterns). Fix: **ratio test** — keep a match only if the best candidate is at least ~25% better (lower distance) than the second-best; otherwise discard as ambiguous.

## Evaluating Matching Quality

Given a matcher's output (pairs of points + a distance, where **lower distance = better match**), how do we decide numerically which matches are "good"?

### Thresholding matches

- Choose a distance threshold: matches below it are kept ("good"), matches above it are discarded.
- The **matcher doesn't choose the threshold** — that's a system-design decision based on the application.
- Aggressive (low) threshold → fewer matches, but higher quality.
- Relaxed (high) threshold → more matches, but lower quality.

### Worked example

Matcher produces 1000 matches; an **oracle** (ground truth) says 800 are actually good and 200 are actually bad.

Choose threshold = 0.6 → 700 matches survive, of which 600 are truly good and 100 are truly bad.

- **True Positive (TP):** 600 (good matches that survived the threshold)
- **False Positive (FP):** 100 (bad matches that survived the threshold)
- **True Positive Rate (TPR / Recall):** `TP / total actual positives = 600 / 800 = 0.75`
- **False Positive Rate (FPR):** `FP / total actual negatives = 100 / 200 = 0.5`
- **Specificity** = `1 − FPR`

### Effect of threshold choice

- **Very aggressive threshold** (e.g. 0.001): few matches survive; ideally almost all survivors are true positives, few false positives. Good for **high-security use cases** (e.g. face recognition for a secure facility) — accept fewer people, but almost never let in the wrong person.
- **Very relaxed threshold** (e.g. 0.99, ~everything survives): both TP and FP counts rise close to their totals (≈800 TP, ≈200 FP here). Good for **low-stakes/high-recall use cases** (e.g. letting students into a lounge) — better to include some non-members than exclude nearly all real members.
- **The right threshold depends entirely on the application** — the algorithm doesn't determine this; the system designer does.
- Real facial recognition systems often target a **very specific operating point**: TPR at FPR ≈ 0.001% (roughly 1-in-a-million) — because at global scale (billions of people), even "1 in a million" false-positive rate is still a lot of absolute false matches.

### ROC Curve (Receiver Operating Characteristic)

Rather than committing to one threshold, plot **TPR (y-axis) vs. FPR (x-axis)** as the threshold is swept across its full range.

- At very low thresholds: low FPR, low TPR.
- At very high thresholds: both FPR and TPR rise toward 1.
- **Perfect system:** TPR = 1 and FPR = 0 simultaneously → a curve that hugs the top-left corner.
- **Area Under the Curve (AUC):** summarizes performance independent of any specific threshold — larger AUC = better/more general matcher.
- This lets you evaluate an algorithm's general quality without committing to one specific operating point, vs. the single-point measure (TPR at a fixed low FPR) used for a specific deployment requirement.
- In practice, real recognition papers report **both**: AUC (general quality) and TPR-at-fixed-FPR (specific deployment requirement).

**Terminology notes:**
- TPR = **Recall**
- `1 − FPR` = **Specificity**
- The exact same TP/FP/TPR/FPR/ROC/AUC framework reappears throughout ML/deep learning for classification evaluation — this is foundational and commonly asked in ML interviews.
- Historical origin: **"Receiver Operating Characteristic"** comes from signal processing / communication theory — originally about whether a receiver correctly detects a transmitted signal vs. noise.

**Likely exam question formats:**
1. Given a threshold, compute TPR and FPR.
2. Given a target FPR (e.g. < 0.1), determine what threshold achieves it.

### Aside: SuperPoint / SuperGlue vs. classical matching

Neural-network-based detection+matching (SuperPoint for detection, SuperGlue for matching) produces many more correct matches than classical methods (SIFT + brute-force/ratio test) when there is **large geometric change** between images (e.g. fast camera rotation). Classical methods still get a *few* good matches but very sparse — insufficient when large transformations are involved (e.g. panorama frames captured with significant rotation between shots). Practical implication for building a panorama: keep rotation between consecutive captures small (similar to how phone panorama modes require slow, steady motion) — too much geometric change between frames breaks matching.

## Image Warping vs. Image Filtering

- **Filtering** operates on the **range** of the image function — i.e., pixel *intensity* — while the pixel *coordinates* stay fixed (blurring, gradients, convolution in general).
- **Warping** operates on the **domain** — i.e., pixel *coordinates/location* — while intensity is simply copied from the source location to the new location (translation, rotation, resizing, aspect-ratio changes — the transformations phones/cameras apply under the hood).

Warping is represented as a matrix operation mapping a point `(x, y)` to a new point `(x̂, ŷ)`.

Three progressively larger classes of transformation, each a superset of the previous:

**Linear ⊂ Affine ⊂ Homography (Projective)**

Any linear transform can be expressed as an affine transform; any affine transform can be expressed as a homography; the reverse is not true.

## Linear Transformations (2×2 matrix)

`[x̂, ŷ]ᵀ = A · [x, y]ᵀ` where `A` is a 2×2 matrix.

Examples:
- **Rotation** by `θ`: `[[cosθ, −sinθ], [sinθ, cosθ]]` (provable via converting to polar coordinates and applying trig identities).
- **Identity**
- **Shear/scale**: scales `x` and `y` by potentially different factors (e.g. dragging an image in PowerPoint without preserving aspect ratio).
- **Flip** about an axis.

**Critical limitation:** a 2×2 linear transform **cannot represent translation**. Expanding the matrix multiplication `[ax+by, cx+dy]` — there's no way to introduce a constant offset term like `+tx`. This motivates moving to affine transformations.

## Homogeneous Coordinates & Affine Transformations

To represent translation, extend each 2D point `(x, y)` to a 3D vector `(x, y, 1)` — a **homogeneous coordinate**.

### Why homogeneous coordinates work (intuition)

An image lives in the 2D `(x, y)` plane, conceptually placed at height `z = 1` in a 3D space. Any point `(x, y)` can then be written as `(x, y, 1)`.

More generally, `(αx, αy, α)` for **any** `α ≠ 0` represents the *same* 2D point `(x, y)` — e.g. `(10, 10, 1)`, `(20, 20, 2)`, and `(100, 100, 10)` are all the same point. Geometrically, a 2D point maps not to a single 3D point but to an entire **line through the origin** in 3D (the line `{(αx, αy, α) : α ∈ ℝ}`); the point where that line crosses the `z=1` plane recovers the original 2D coordinate. This preserves the degrees of freedom: a 2D point (2 unknowns) maps to a 3D line (also 2 unknowns — e.g. slope and intercept), not to a full 3D point (3 unknowns).

The reason this representation is useful: matrix operations can't easily be constrained to always produce exactly `1` in the last coordinate, so instead we allow *any* scalar there and treat all scalar multiples as equivalent.

Coordinate convention used in class: **right-handed coordinate system**, drawn with `x` out of the page/ground, `y` horizontal, `z` vertical — chosen because in 3D vision, the `z`-axis usually represents camera depth, which is convenient to draw this way.

### Affine transformation

Using homogeneous coordinates, translation becomes linear again:

```
[x̂]   [1  0  tx] [x]
[ŷ] = [0  1  ty] [y]
[1 ]   [0  0  1 ] [1]
```

This generalizes: any linear 2×2 transform (rotation, scale, shear, flip, etc.) can be embedded in the top-left 2×2 block of a 3×3 matrix whose bottom row is fixed at `[0, 0, 1]`:

```
[a  b  c]
[d  e  f]
[0  0  1]
```

Setting `c = f = 0` recovers a pure linear transformation — **linear transforms are a special case of affine transforms.**

**Order matters:** composing transformations (e.g. rotate-then-translate vs. translate-then-rotate) generally gives *different* results — matrix multiplication is not commutative here. E.g. rotating a point then translating it ≠ translating first then rotating.

**Properties preserved by affine transformations:**
- Straight lines map to straight lines.
- Parallel lines remain parallel.
- **Note:** the origin does *not* necessarily map to the origin under general affine transforms (e.g. translation moves `(0,0)` away from the origin) — this differs from pure linear transforms, where the origin is always fixed.

## Homography (Projective) Transformation

Generalize affine further by allowing the **bottom row to be arbitrary** instead of fixed at `[0, 0, 1]`:

```
[a  b  c]
[d  e  f]
[g  h  1]
```

Applying this to homogeneous `(x, y, 1)` gives `(ax+by+c, dx+ey+f, gx+hy+1)`. Converting back to a 2D point (dividing by the third/homogeneous coordinate):

```
x̂ = (ax + by + c) / (gx + hy + 1)
ŷ = (dx + ey + f) / (gx + hy + 1)
```

- For affine transforms, `g = h = 0`, so the denominator is always `1` — giving a clean **linear** relationship between input and output coordinates.
- For general homography, the denominator depends on `x, y` — making the transformation **non-linear**. This will matter significantly when solving for these transformations later.
- The bottom-right entry doesn't need to be fixed at exactly `1` — any nonzero constant `k` there is equivalent (it factors out of both numerator and denominator), since only the *direction*/ratio of the homogeneous vector matters, not its absolute scale.

**Key property that breaks under homography:** parallel lines no longer remain parallel after transformation. Classic example: a photograph of railway tracks — physically parallel tracks converge toward a vanishing point in the 2D image, because the mapping from the 3D/ground plane to the image plane is a homography. (Some of these properties are provable in a couple of lines with the right approach, vs. pages of brute-force algebra — worth practicing efficient proofs, since this will likely be a homework/exam topic.)

## Next Class

Continuing with homography transformations — particularly the intuition behind the parallel-lines property and formal proofs of related properties.

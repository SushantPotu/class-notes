# COMP 577 — Filtering Primer (Supplementary Background)

> **Not from lecture.** Midterm 1 Question 1 is on filtering, but there are no recordings of the earlier filtering lectures, so this page is general background written to fill the gap. Check every item against the actual slides and the practice midterm; the professor's notation and emphasis may differ.

## The Core Idea

- **Filtering** computes each output pixel from a neighborhood of input pixels. It changes intensity values and leaves pixel coordinates alone (contrast with warping, which moves coordinates; see the Sep 8 notes).
- A filter (kernel) `h` is a small grid of weights. The most common filters are **linear and shift-invariant**: every output pixel is a weighted sum of its neighborhood, using the same weights everywhere.

## Correlation vs. Convolution

```
Correlation:  (h ⊗ I)(x, y) = Σᵢ Σⱼ h(i, j) · I(x + i, y + j)
Convolution:  (h * I)(x, y) = Σᵢ Σⱼ h(i, j) · I(x − i, y − j)     (kernel flipped)
```

For symmetric kernels (box, Gaussian) the two are identical. Convolution is commutative and associative, so filters can be combined by convolving the kernels first.

## Common Filters

| Filter | Kernel idea | Use |
|---|---|---|
| Box / mean | all weights equal (`1/9` for 3×3) | quick blur; crude |
| Gaussian | weights fall off as `exp(−(x²+y²)/2σ²)`; `σ` sets blur amount | smoothing, noise reduction, building pyramids |
| Derivative (central difference) | `[−1 0 1]` | horizontal/vertical gradient `Ix`, `Iy` |
| Sobel | `[[−1,0,1],[−2,0,2],[−1,0,1]]` (x direction) | gradient with a little smoothing built in |
| Derivative of Gaussian | Gaussian smoothing + differentiation in one kernel | gradients in Harris (Sep 3 notes) |
| Median | *non-linear*: output = median of the neighborhood | removes salt-and-pepper noise while preserving edges |

- **Gradient magnitude** `√(Ix² + Iy²)`; **orientation** `atan2(Iy, Ix)`. The gradient direction is perpendicular to the edge (used in SIFT descriptors).
- **Separability:** a 2D Gaussian equals two 1D Gaussians applied one after the other. For a `k × k` kernel that cuts the cost per pixel from `k²` to `2k`.
- **Gaussians compose:** blurring with `σ₁` then `σ₂` equals one blur with `σ = √(σ₁² + σ₂²)`.

## Practical Details

- **Borders:** the kernel hangs off the image edge, so choose a rule: zero-padding, replicating the edge pixel, or reflecting the image.
- **Output size:** "same" (padded to keep the size) vs. "valid" (only positions where the kernel fully fits; the output shrinks).
- **Larger σ** = more blur = lower frequencies kept (Gaussian is a low-pass filter). Subtracting a blurred image from the original keeps high frequencies (used in Laplacian pyramids and DoG).
- **Gaussian pyramid:** blur, then downsample by 2, repeatedly. **Laplacian pyramid:** the difference between a level and the blurred/upsampled next level; it holds the high-frequency detail (mentioned in the Sep 15 blending discussion).

## Worked Example

Patch:

```
1 2 3
4 5 6
7 8 9
```

- Box filter (3×3, weights 1/9) at the center: `(1+2+…+9) / 9 = 45 / 9 = 5`.
- Sobel-x at the center (correlation): `(−1·1 + 1·3) + (−2·4 + 2·6) + (−1·7 + 1·9) = 2 + 4 + 2 = 8`. Positive means intensity increases left to right, which matches the patch.

## "Which Filter Would You Choose, and Why?"

The professor said many filtering questions are conceptual. A reasonable template:

- **Noise removal (Gaussian noise):** Gaussian or box smoothing; the trade-off is blur of real detail.
- **Salt-and-pepper noise:** median filter, because it ignores outliers and preserves edges.
- **Edges/gradients:** derivative filters (Sobel, derivative of Gaussian); smooth first so noise is not amplified.
- **Blobs at a given scale:** Laplacian of Gaussian or DoG.
- **Speed on a large kernel:** use separability where the kernel allows it.

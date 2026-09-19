# COMP 577 — Image Recognition, Classification Metrics, and Linear/Neural Classifiers

**Date:** September 17, 2026

> This lecture starts the next section of the course. It is **not** Midterm 1 material (the professor said today's content is for the next exam). See [midterm-1-prep.md](midterm-1-prep.md) for what is on Midterm 1.

## Logistics

- Assignment due Friday at midnight; the TA has office hours for it. For software-installation problems, go to the TA/lab support rather than the professor.
- Light on math today; more conceptual.

## What Is Recognition?

A family of tasks, from simple to complex:

| Task | Question | Example |
|---|---|---|
| Verification | Yes/no about one thing | "Is this a lamp?" |
| Detection | Where are the objects? | Bounding boxes around people (self-driving cars) |
| Identification | Which specific place/instance? | "Where was this image taken?" |
| Object categorization | What kind of thing is it? | Mountain, tree, building |
| Scene understanding | What is happening overall? | Outdoor marketplace in a city with a mountain |
| Activity recognition | What are people doing? | Shopping, walking |

- Modern AI handles verification and categorization well. It struggles with **fine-grained understanding**, e.g. seeing someone cook and judging what dish it is or how skilled they are.

### Tasks studied in this course

1. **Classification** — which objects/labels are in the image (next two classes).
2. **Object detection** — classify *and* localize with bounding boxes (person, sheep, dog).
3. **Semantic segmentation** — a class label for every pixel (sheep / person / dog / none).
4. **Instance segmentation** — separate individual instances (five sheep get five different colors).

Everything builds on image classification architectures. The same ideas extend to depth prediction, structure prediction, and image description.

## Why Recognition Is Hard

- The naive approach is template matching: slide a template of the object over the image, compute a similarity map, and look for peaks. It works for the same chair in the same scene and breaks almost everywhere else.
- Sources of variation:
  - **Viewpoint** variation
  - **Lighting** variation
  - **Deformation** (cats are the classic example)
  - **Background clutter** (objects blend into the background)
  - **Occlusion**
- Object recognition research began in the early 1960s. It is reasonably well solved as of about 2024–2025 (e.g. "find my cat" in a photo library works ~99% of the time).
- Three ingredients drive progress:
  1. **Data** — historically the biggest bottleneck.
  2. **Representations** — from low-level hand-designed features (SIFT) to learned high-level features.
  3. **Learning techniques** — the focus of this part of the course.

## Measuring Classification Performance

### Classifier as a probability

- Conceptually a classifier is `f(image) → label` (cat/dog).
- In practice it outputs a **probability**: `P(y | x)`, the probability of label `y` given image `x` (`x` = data/input, `y` = label/output).
- Training means **maximizing the probability of the correct label**. This is **maximum likelihood estimation (MLE)**, a general statistical tool that predates computer vision and ML.

### From probability to a decision

- Choose a **threshold**: if `P(cat | x)` is above it, predict cat. Is 60% confidence enough? That is a design decision.
- This is the same idea as thresholding match distances in feature matching. There you had 1000 matches and kept ~700 under a distance threshold.
- Call cat the **positive** class and dog (not-cat) the **negative** class.

### Confusion table (binary)

| | Predicted positive | Predicted negative |
|---|---|---|
| **Actually positive** | True Positive (TP) | False Negative (FN) |
| **Actually negative** | False Positive (FP) | True Negative (TN) |

The diagonal (TP, TN) is what we want to maximize. The off-diagonal (FP, FN) is what we want to minimize.

### ROC curve and AUC

- Vary the threshold from 0 to 1, plot **TPR vs. FPR**, and you get the same ROC curve as in the feature-matching evaluation. **Area under the curve (AUC)** is the summary number.
- **Perfect classifier:** hugs the top-left corner.
- **Random classifier** (a coin toss): the diagonal line. We want to be well above it; more area is better.

### Precision, recall, F1

- **Precision** = `TP / (TP + FP)`. It is highest when FP = 0. Intuition: of the things I picked, how many were right (e.g. picking 10 objects from a bin and 9 are good).
- **Recall** = `TP / (TP + FN)` (same as TPR). It is highest when FN = 0. Intuition: did I recover as many true positives as possible, even at the cost of many false positives?
- **F1 score** = the **harmonic mean** of precision and recall, a single-number summary (like AUC).
- **Precision–recall curve:** the same idea as ROC with different axes. The curve looks different depending on which metrics you plot; both TPR/FPR and precision/recall are standard in papers.
- Which operating point you want depends on the application:
  - Nuclear facility face recognition → very low FPR (high precision).
  - Undergraduate lounge access → false positives matter little; favor recall.

### Worked example: all the binary metrics from one table

Reusing the feature-matching numbers: 800 actual positives, 200 actual negatives, and a threshold that leaves `TP = 600`, `FP = 100`, `FN = 200`, `TN = 100`.

| Metric | Formula | Value |
|---|---|---|
| Precision | `TP / (TP + FP)` | 600 / 700 = **0.857** |
| Recall / TPR | `TP / (TP + FN)` | 600 / 800 = **0.75** |
| FPR | `FP / (FP + TN)` | 100 / 200 = **0.50** |
| F1 | `2·P·R / (P + R)` | **0.80** |
| Accuracy | `(TP + TN) / total` | 700 / 1000 = **0.70** |

Accuracy can mislead on imbalanced data: with 99% negatives, a classifier that always says "negative" scores 99% accuracy but has recall 0. That is why the lecture stresses class balance and per-class metrics.

### Multi-class: confusion matrix

- Rows = **true class**, columns = **predicted class**.
- Example row for "truck" (1000 truck examples): 923 predicted truck, 49 predicted automobile, 20 predicted plane, etc. Each row sums to the number of examples of that class.
- **Diagonal = correct**; the goal is to minimize the off-diagonal spread.
- Not all errors are equally bad: confusing a truck with an automobile is much less bad than confusing a truck with a plane. Ordering classes by similarity (affinity) makes the matrix informative: errors should sit close to the diagonal.
- **Per-class metrics** (precision, recall, AUC) are computed by treating one class vs. the rest, e.g. *truck vs. not-truck*, which reduces to the binary case.

### Class balance

- With **balanced** classes (equal counts), you can divide each row by the class size to get percentages.
- With **heavily imbalanced** classes (e.g. 90/10), the classifier overfits to the majority class and metrics become misleading.
- One fix: make the **test set balanced** across classes. Whether this is right depends on the application. If you care about a rare event (e.g. flash floods, ~1% of the data), a balanced test set makes sure you actually measure detection of the rare event.
- Takeaway: models keep changing, but **how to measure them does not** — this is the most durable skill for industry work.

### Q&A: what is a "prior"?

- A prior is the accumulated experience/expectation about what data looks like. You can recognize a bobcat as "some kind of cat" without having seen one, because you have seen many cats.
- Similarly, a child shown many examples of "dog" builds a concept of "dog" that transfers to unseen dogs.
- An ML model trained on many examples encodes a prior in the same way; mathematically, a prior is a distribution over the data.

## Datasets (History)

- Early computer vision used very small datasets (the professor told stories of printing photos, scanning them, and mailing results). Data was scarce.
- The **PASCAL VOC** challenge: about 20 object categories (airplane to TV monitor). It started the era of benchmark challenges; about 20,000 images was considered large circa 2010.
- **ImageNet** (Fei-Fei Li and collaborators): **1000 object classes, ~1 million images**, a huge jump in scale and in fine-grained diversity (texture, color, shape).
- Later: Microsoft's large dataset (~2015–2016), stated in the lecture as roughly 10× ImageNet. *(The exact category and image counts in the transcript are unclear; verify against the slides.)*
- Building datasets is a huge effort (curation, benchmarking). Industry labs such as Meta's FAIR have shifted away from open-ended computer vision research toward building general models.

## Classic Pipeline (Pre-Deep-Learning)

1. Training images (with labels).
2. **Feature extraction** — e.g. a 128-dimensional feature vector per image (SIFT-style feature engineering dominated for ~20 years).
3. **Train** a classifier on (features `x`, labels `y`).
4. **Evaluate** on held-out test images (proper train/test splitting to be covered later).

Earlier, "eigenfaces" showed that plain eigen-decomposition could support recognition.

Classifiers covered: **k-nearest neighbors** (conceptual), **linear classifiers** (foundation of deep learning), then **neural networks**. Transformers get a high-level look later; graduate courses cover them in depth.

## k-Nearest Neighbors

- **Training:** just store all training examples and labels. No learning.
- **Testing:** find the closest training example(s) to the test point and copy the label.
- With 1 neighbor, the decision boundaries form a Voronoi-like partition of the space. On real, messy data this gives noisy "islands", where a slight move flips the label.
- **k-NN:** look at the `k` closest neighbors and take a **majority vote**. This smooths the boundaries. Ties (e.g. 2 red, 2 green, 1 blue) are ambiguous.

### Why it is impractical

- **Test-time cost:** each test point must be compared to all `N` training points, each comparison costing `d` (feature dimension). ImageNet has ~1M examples, so that is enormous per query.
- **Curse of dimensionality:** in high dimensions (e.g. 128-D) data is extremely sparse; there are large regions with no nearby training data, so predictions there are unreliable.
- It is still useful for building intuition about what classification means.

## Linear Classifier

- Learn a line/plane that separates the classes. At test time, check which side of it a point falls on.
- Model: `f(x) = Wx + b`
  - `x` — the input (an image flattened into a vector, or a feature vector).
  - `W` — the **weight matrix**, with **one row per class** (10 classes → 10 rows).
  - `b` — the **bias**.
- Each row defines a **hyperplane** (the n-D generalization of a line in 2D / plane in 3D) that splits the space into "this class" vs. "not this class".
- The outputs are raw class scores; the **softmax** function turns them into probabilities that sum to 1 (normalize the exponentiated scores). *(The transcript is garbled around the softmax explanation; it is standard: `softmax(s)ᵢ = exp(sᵢ) / Σⱼ exp(sⱼ)`.)*
- **Training goal:** find the model parameters `W` and `b` from the labeled data `(xᵢ, yᵢ)`.

### Training needs two things

1. A **loss function** — quantifies how unhappy we are with the model's predictions. For example, a cat image that gets an 82% "dog" prediction should produce a high loss.
2. An **optimization** procedure — uses the loss to update the parameters.

Total loss = the **average of the per-example loss** over all training data, comparing the prediction to the ground-truth label.

### Cross-entropy loss

- Apply softmax to the scores to get the probability of the correct class, take the log, and negate: the **negative log likelihood**.
- Minimizing this is equivalent to **maximizing the likelihood of the correct label** (the MLE idea from the start of the lecture). The log is taken for numerical convenience.
- Only the probability assigned to the **correct** class matters. Calling a cat a dog is penalized the same as calling it a ship. You could design a loss that penalizes plausible mistakes less, but the basic version does not.
- **Regularization:** add a term that discourages very large weights. Scaling `W` up does not change the hyperplane, so this keeps learning focused on the actual goal rather than inflating weight magnitudes.

### Worked example: softmax and cross-entropy

Suppose the linear classifier outputs raw scores for `[cat, dog, ship] = [3.2, 5.1, −1.7]`.

1. Exponentiate: `[e^3.2, e^5.1, e^−1.7] = [24.5, 164.0, 0.18]`; the sum is 188.7.
2. Normalize: probabilities `= [0.13, 0.87, 0.001]`, which sum to 1.
3. If the true label is **cat**, the cross-entropy loss is `−ln(0.13) ≈ 2.04` (large: the model was confident and wrong). If the model had given cat 0.9, the loss would be `−ln(0.9) ≈ 0.105`.
4. Check of the "cat vs. ship" remark: the loss depends only on the probability of the true class, so putting the leftover probability on dog or on ship costs the same.

**Why negative log likelihood:** for independent training samples the likelihood is `∏ P(yᵢ | xᵢ)`. Taking the log turns the product into a sum, `Σ log P(yᵢ | xᵢ)`; negating turns "maximize" into "minimize", and averaging over the data gives the cross-entropy loss. So minimizing cross-entropy *is* maximum likelihood estimation.

### Optimization for a linear classifier

- As presented in the lecture: take the loss's partial derivatives with respect to `W` and `b`, set them to zero, and solve for a closed-form solution. This is why linear classifiers are considered "easy".
- *(Note: for softmax cross-entropy in general, this is not truly closed-form and is usually solved iteratively with gradient descent. Check the slides for how the professor frames it.)*

## From Linear Classifiers to Neural Networks

- A neural network = **linear layers stacked with nonlinearities in between**.
- Example nonlinearity (**ReLU**): `y = max(0, x)`. Other activation functions exist.
- Structure: `x → W₁x → nonlinearity → W₂(·) → …` The **last layer is always linear**.
- **Why the nonlinearity is essential:** without it, stacked linear layers collapse into a single linear map (`W₃W₂W₁` is just one matrix), so depth adds nothing. The nonlinearity prevents this collapse.
- Example sizing from the lecture: a 3072-dimensional input → 100-D hidden layer → 10 outputs. `W₁` is 100 × 3072 and `W₂` is 10 × 100, so the parameter count is the sum of those entries.
- **Network diagram:** input nodes, hidden nodes, and arrows. Each arrow is one entry of a weight matrix (e.g. a 3-input → 4-hidden layer has a 3 × 4 `W₁`; one arrow might have weight 0.3).
- **Training** uses the same cross-entropy loss, but now the loss is a function of many weight matrices (`W₁`, `W₂`, …), so setting derivatives to zero no longer gives a neat solution. That is where **gradient descent** comes in.

## Worked Details

- **Why stacking linear layers without nonlinearity is pointless:** `W₂(W₁x) = (W₂W₁)x`, and `W₂W₁` is just one matrix, so two linear layers equal one linear layer. With a nonlinearity `σ`, `W₂·σ(W₁x)` cannot be collapsed.
- **Parameter count for the lecture's example** (3072-D input, 100 hidden units, 10 classes; 3072 = 32 × 32 × 3 pixels):
  - Layer 1: `100 × 3072 + 100 (biases) = 307,300`
  - Layer 2: `10 × 100 + 10 (biases) = 1,010`
  - Total: **308,310 parameters**
- **k-NN cost:** with `N = 1,000,000` training examples and `d = 128`, one test query needs about `N · d ≈ 1.28 × 10⁸` multiply-adds, and it must be paid for *every* test sample. A linear classifier needs only `classes × d` operations per sample.
- **Hyperplane geometry** *(standard background)*: the decision boundary between two classes is where their scores tie, `(wᵢ − wⱼ)·x + (bᵢ − bⱼ) = 0`; the weight vector is the normal to that hyperplane, and `|w·x + b| / ‖w‖` is the distance from `x` to it.

## Next Up

- **Next week:** gradient descent — how all neural networks (transformers included) are trained — and computing gradients.
- **Next Thursday:** a recap class (the midterm falls in between).
- Then: from simple neural networks to **convolutional neural networks**.

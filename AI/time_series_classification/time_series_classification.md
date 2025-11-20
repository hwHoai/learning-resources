# Time Series Classification

Short description

- Topic: Time Series Classification (TSC)
- Goal: collect concise notes, references, and practical commands for learning and experimenting with TSC algorithms and libraries.

---

## Overview

Time series classification applies supervised machine learning to labeled temporal data: models learn from examples of different classes and then predict the class of new series. This is essential for tasks like sensor monitoring or financial analysis, where correct, reliable predictions directly affect business decisions.

## Problem types / when to use

- Univariate vs multivariate time series
- Single-label classification (one label per series)
- Multi-label or sequence labeling (per-timestep labels)

``** Use time series classification when your input is ordered temporal data and the goal is to predict a discrete class for the sequence.``

## Types of time series classification

Below is a concise list of common classification types. Each line links to a short, detailed description further down.

1. [Distance-based](#distance-based) — compare series using a distance metric (e.g., DTW) with instance-based classifiers (k-NN).
2. [Shapelet](#shapelet) — discover discriminative subsequences (shapelets) that separate classes.
3. [Model ensembles](#model-ensembles) — combine multiple classifiers (often heterogeneous) for robust performance (e.g., HIVE‑COTE).
4. [Dictionary approach](#dictionary-approach) — convert series to symbolic words and classify using bag-of-words techniques (SAX, BOSS).
5. [Interval-based approach](#interval-based-approach) — extract features from random or selected intervals and classify using feature-based learners.
6. [Deep learning](#deep-learning) — end-to-end learning with CNNs/RNNs/Transformers to automatically extract discriminative features.

### Distance-based

Distance-based methods use a numeric **distance** to quantify how similar two time series are. The smaller the distance, the more similar the series. Common distance measures used in ML include **Euclidean**, **Manhattan**, **Minkowski**, and **Hamming** distances — but for time series we often need measures that tolerate time shifts and local misalignments, which is where **Dynamic Time Warping (DTW)** shines.

How distance-based classification works (typical **k‑NN** flow):

- **Normalize** series (usually **z-normalization**) to remove scale/offset differences.
- Compute **distances** between the test series and every training series using a chosen metric.
- Select the **k** training series with the smallest distances.
- Assign the test series the majority class among those **k** neighbors (**k=1** gives **1‑NN**).

Notes on distance metrics and algorithms:

- **Euclidean** / **Manhattan** / **Minkowski**: simple pointwise distances; efficient but sensitive to small time shifts.

- **Hamming**: for discrete/symbolic sequences; counts mismatches at aligned positions.

- **Kernel / SVM** relation: kernel methods (e.g., **SVM**) can incorporate similarity information — a kernel can be built from a distance or similarity measure so that the classifier uses pairwise similarities (think of an **SVM** with a custom kernel derived from a distance matrix).

Why **DTW** is special for time series:

- **DTW** aligns two sequences by warping the time axis to find the minimum cumulative distance (warping path) between them; this handles local speed/phase differences (e.g., the same pattern occurs slightly earlier/later).

- **DTW** computes a cost matrix and a minimal-cost path through it. Complexity is **O(n\*m)** for series lengths n and m; common speedups include limiting the **warping window** (Sakoe–Chiba / Itakura constraints) and using lower bounds (**LB_Keogh**) to prune comparisons.

- In practice, **DTW + 1‑NN** has been a strong baseline for many TSC tasks and is commonly used in benchmarks.
#### Example: Comparing two walkers

Imagine two people walking:
- **Sequence A (Person A)**: [Walk, Rest, Walk, Rest]
- **Sequence B (Person B)**: [Rest, Walk, Rest, Walk]

Essentially, both people exhibit the same behavior (2 steps of walking, 2 beats of resting), but Person B simply starts 1 beat later.

1. **Using Euclidean Distance (Pointwise Comparison):**
   - Euclidean compares rigidly, point-by-point:
     - T1: (A Walk) vs (B Rest) → Different!
     - T2: (A Rest) vs (B Walk) → Different!
     - T3: (A Walk) vs (B Rest) → Different!
     - T4: (A Rest) vs (B Walk) → Different!
   - **Conclusion by Euclidean**: These sequences are completely different (large distance).

2. **Using Hamming Distance (Pointwise Comparison):**

    * Hamming distance rigidly counts the mismatches at each corresponding position:

      * **T1:** (A `Walk`) vs (B `Rest`) $\rightarrow$ Mismatch! (Count = 1)
      * **T2:** (A `Rest`) vs (B `Walk`) $\rightarrow$ Mismatch! (Count = 2)
      * **T3:** (A `Walk`) vs (B `Rest`) $\rightarrow$ Mismatch! (Count = 3)
      * **T4:** (A `Rest`) vs (B `Walk`) $\rightarrow$ Mismatch! (Count = 4)

    - **Conclusion by Hamming:** These sequences are **completely different**. The Hamming distance is 4, which is the maximum possible distance for sequences of this length, implying zero similarity.

3. **Using DTW (Dynamic Time Warping):**
   - DTW is smarter. It "warps" the time axis to find the best alignment:
     - (A Walk at T1) aligns with (B Walk at T2).
     - (A Rest at T2) aligns with (B Rest at T3).
     - (A Walk at T3) aligns with (B Walk at T4).
   - **Conclusion by DTW**: These sequences are very similar, just slightly misaligned (phase-shifted).

This example highlights why DTW is particularly effective for time series data, as it can handle local shifts in time and still identify underlying similarities.

### 📊 Bảng so sánh các phương pháp đo lường

| Phương pháp | Loại dữ liệu chính | Cách hoạt động (Đơn giản) | Điểm mạnh / Điểm yếu |
| :--- | :--- | :--- | :--- |
| **Euclidean / Manhattan** | Dữ liệu số, liên tục | So sánh **1-đối-1** tại *cùng một thời điểm*. (Điểm `T1` của A vs. Điểm `T1` của B) | **(+)** Rất nhanh, hiệu quả.<br> **(-)** **Rất nhạy cảm** nếu dữ liệu bị lệch (shift) dù chỉ một chút. |
| **Hamming** | Dữ liệu rời rạc, ký hiệu | Đếm số lượng vị trí mà hai chuỗi *không khớp* nhau. (Ví dụ: `A,B,C` vs `A,B,D` có 1 điểm không khớp) | **(+)** Tốt cho dữ liệu dạng ký tự, gen, v.v.<br> **(-)** Không dùng được cho dữ liệu số liên tục (như nhiệt độ). |
| **DTW (Dynamic Time Warping)** | Dữ liệu số, liên tục | So sánh **1-đối-nhiều**. Nó "co giãn" thời gian để tìm cách căn chỉnh (align) tốt nhất. | **(+)** **Cực kỳ tốt** cho các chuỗi bị lệch pha/tốc độ.<br> **(-)** Tính toán chậm và tốn kém hơn (phức tạp O(n*m)). |
| **Kernel (dùng trong SVM)** | Bất kỳ (tùy kernel) | Đây là một *mô hình*, không phải phép đo. Nó dùng một "kernel" (có thể dựa trên DTW) để biến đổi dữ liệu. | **(+)** Rất mạnh mẽ để phân loại.<br> **(-)** Phức tạp hơn, không chỉ là một phép đo khoảng cách. |

Practical tips and pitfalls:

- Always **z-normalize** series before **DTW** or **Euclidean** comparisons — otherwise scale dominates the distance.

    * Z-normalization is a preprocessing step that standardizes a time series to have a mean of 0 and a standard deviation of 1. This allows algorithms like DTW to compare data based on shape rather than differences in scale or offset.
    $$z = \frac{x - \mu}{\sigma}$$

            **example:
                x1  x2  x3      x'1    x'2  x'3
            A: [26, 27, 26] -> [-0.33, 0.667, -0.33]
            B: [36, 37, 36] -> [-0.33, 0.667, -0.33]

                                        A     B
            * μ = (x1 + x2 + x3)/3 = [26.33,36,33]
            * σ = 1

- Use a **warping window** (e.g., 5–10% of series length) to avoid pathological warpings and speed up computation.

    * A "warping window" (e.g., 10% of series length) is a constraint added to DTW. Instead of calculating the entire $n \times m$ cost matrix, it only computes a small "band" around the diagonal.

        * **Benefit 1 (Speed):** Drastically speeds up computation from $O(n \times m)$ to $O(n \times r)$ (where $r$ is the window size).
        * **Benefit 2 (Accuracy):** Prevents "pathological warping" (unrealistic stretching/compressing of time) by forcing matches to stay relatively close. (ex: $|i - j| \le r$)

- For large datasets, compute and cache pairwise distances or use approximate methods / lower bounds (**LB_Keogh**) to prune.

    * Pruning is a "skip" strategy used to speed up search algorithms like KNN. Instead of performing the **slow, expensive** distance calculation (like DTW) for *every* item in the dataset, it uses a **fast, simpler** "proxy" calculation (a Lower Bound) to quickly eliminate candidates.

    * The logic is: If the *fast proxy distance* is already worse than the best match you've found so far, you can **"prune" (safely discard)** that candidate without ever running the slow calculation.

    * Key Advantages

        * **Massive Speedup:** It dramatically reduces the total number of expensive calculations required, often by over 99%. This makes KNN-DTW feasible on large datasets that would otherwise be computationally impossible.
        * **Guaranteed Accuracy:** When used correctly (with a true lower bound), pruning is an **exact** optimization, *not* an approximation. The final result (the nearest neighbor) is **guaranteed to be the same** as if you had run the full, slow brute-force search.
        
                Q = [1, 3, 2, 4]
                C = [3, 4, 1, 5]
                r = 1
            
        |  i | C[i] | L[i] = min(Q[i - r, i + r]) | U[i] = max(Q[i - r, i + r]) | Kiểm tra | Phí phạt (Cost) |
        | :--- | :--- | :--- | :--- | :--- | :--- |
        | **0** | **3** | `min(Q[0,1]) = min(1,3)` = **1** | `max(Q[0,1]) = max(1,3)` = **3** | `1 <= 3 <= 3` (Trong hầm) | 0 |
        | **1** | **4** | `min(Q[0..2]) = min(1,3,2)` = **1** | `max(Q[0..2]) = max(1,3,2)` = **3** | `4 > 3` (Vọt trần) | `(4 - 3)² = 1` |
        | **2** | **1** | `min(Q[1..3]) = min(3,2,4)` = **2** | `max(Q[1..3]) = max(3,2,4)` = **4** | `1 < 2` (Đâm sàn) | `(2 - 1)² = 1` |
        | **3** | **5** | `min(Q[2,3]) = min(2,4)` = **2** | `max(Q[2,3]) = max(2,4)` = **4** | `5 > 4` (Vọt trần) | `(5 - 4)² = 1` |
        | **Tổng** | | | | **Tổng Khoảng cách (LB\_Keogh)** | **0 + 1 + 1 + 1 = 3** |

- Consider **derivative DTW** or **weighted DTW** variants if amplitude vs. shape matters.

  1. Derivative DTW (DDTW)

  **Idea:**
Instead of comparing the absolute values `A[i]` vs. `B[j]`, this algorithm compares their *derivatives* (estimated slopes) at those points (e.g., `(A[i] - A[i-1])` vs. `(B[j] - B[j-1])`).

  **What it Solves:**
It completely ignores differences in "offset" (baseline shift). The two series `[1, 2, 3]` and `[101, 102, 103]` would have a DDTW distance of nearly 0, as they both have the same *slope* (a constant increase of 1).

  **When to Use:**
When **shape** and **trend** are the only things that matter, and the absolute amplitude (value) is irrelevant or misleading. (e.g., gesture recognition, signature matching).

  2. Weighted DTW (WDTW)

  **Idea:**
  This is standard DTW, but it adds a "weight" to the distance calculation at each point, often based on amplitude. The formula becomes: `Cost = Weight(i, j) * (A[i] - B[j])²`.

  **What it Solves:**
  It addresses the problem that "not all points are equally important." It heavily penalizes misalignments at peaks or valleys and cares less about errors along the flat baseline.

  **When to Use:**
  When **both shape and amplitude matter**, and you specifically want to ensure that high-amplitude events (like ECG peaks, stock price spikes) are aligned correctly.

Small runnable example (**DTW + 1‑NN** using `tslearn` + scikit-learn):

```powershell
python -m venv .venv; .\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install tslearn scikit-learn numpy
```

```python
import numpy as np
from tslearn.metrics import cdist_dtw
from sklearn.neighbors import KNeighborsClassifier

# Dummy example: 4 training series (2 classes), each series length=20
X_train = np.array([
  np.sin(np.linspace(0, 2*np.pi, 20)),
  np.sin(np.linspace(0, 2*np.pi, 20) + 0.5),
  np.cos(np.linspace(0, 2*np.pi, 20)),
  np.cos(np.linspace(0, 2*np.pi, 20) + 0.3)
])
X_train = X_train.reshape((4, 20, 1))  # tslearn expects shape (n_samples, sz, dim)
y_train = np.array([0, 0, 1, 1])

# One test series (shifted sine)
X_test = np.array([np.sin(np.linspace(0, 2*np.pi, 20) + 0.2)]).reshape((1, 20, 1))

# Compute distance matrices
D_train = cdist_dtw(X_train, X_train)   # shape (n_train, n_train)
D_test = cdist_dtw(X_test, X_train)     # shape (n_test, n_train)

# Train k-NN with precomputed distances
knn = KNeighborsClassifier(n_neighbors=1, metric='precomputed')
knn.fit(D_train, y_train)
pred = knn.predict(D_test)
print('Predicted class:', pred[0])
```

When to choose distance-based methods:

  * Distance-based approaches (typically **k-Nearest Neighbors**) classify time series by measuring the global similarity between a test sample and labeled training samples.

    * **Global Similarity Focus:** Use when the class is defined by the **overall shape or pattern** of the entire series, rather than specific small sub-patterns (shapelets) or complex abstract features.
    * **Low Data Regimes:** Highly effective when training data is scarce, as there are no complex parameters to learn (unlike Deep Learning).
    * **Interpretability:** Ideal when you need to explain *why* a classification was made by showing the "nearest neighbors" (similar historical examples).
    * **"Lazy" Learning:** Useful when you need to add new data to the training set frequently without retraining a model (since there is no training phase).

  * Metric Selection Guide:
    * **Euclidean:** Use for fixed-length, perfectly aligned continuous data (fastest).
    * **DTW:** Use for continuous data with phase shifts or speed variations (most accurate).
    * **Hamming / Edit Distance:** Use for **discrete** or symbolic sequences (e.g., DNA, text, log events).
    
### Shapelet

    Shapelet methods search for short, discriminative subsequences whose presence (or distance to them) predicts the class. Shapelets are interpretable (you can inspect the subsequence) and work well when classes are defined by local patterns. Finding shapelets can be computationally expensive; many algorithms use pruning or learning-based discovery.

### Model ensembles

    Ensemble approaches combine multiple classifiers (often heterogeneous: distance-based, interval-based, feature-based) to produce a robust final prediction. Examples: HIVE‑COTE and related ensembles. Pros: strong accuracy across varied problems. Cons: heavy computational and memory cost; more complex to interpret.

### Dictionary approach

    Dictionary (bag-of-patterns) methods transform time series into sequences of discrete symbols (e.g., SAX), build a vocabulary of "words" from subsequences, and then classify using frequency-based features (BOSS, Word Bag approaches). They are efficient and effective for pattern frequency differences, but may lose fine-grained temporal alignment.

### Interval-based approach

    Interval methods randomly or strategically select sub-intervals from series, extract summary features (mean, slope, std, etc.) per interval, and feed those features to traditional classifiers (e.g., Time Series Forest). These methods capture local behaviour with lower complexity than exhaustive subsequence search.

### Deep learning

    Deep models (1D CNNs, RNNs/LSTMs, and Transformers) learn hierarchical features directly from raw series, enabling powerful end-to-end classification. They scale well with data size and can capture complex temporal dependencies, but require more data and hyperparameter tuning, and are less interpretable by default.

## Libraries & tools

- Python:
  - `sktime` — unified framework for time series ML (classification, forecasting)
  - `tslearn` — algorithms and utilities (DTW, KMeans, classifiers)
  - `pyts` — transforms and classifiers for TSC
  - `tsfresh` — automated feature extraction
  - `sktime-dl` / `aeon` — deep learning helpers
- Research & datasets:
  - UCR / UEA time series archive (benchmark datasets)
- Useful packages:
  - `scikit-learn`, `pandas`, `numpy`, `torch` / `tensorflow` for DL

## Quick setup (Python)

Create a virtual environment and install common packages:

```powershell
python -m venv .venv; .\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install scikit-learn pandas numpy matplotlib
pip install sktime tslearn tsfresh pyts
# optional for deep learning
pip install torch torchvision
```

## Minimal example (sketch) — feature-based pipeline

```python
from sktime.datatypes._panel._convert import from_2d_array_to_nested
from sklearn.ensemble import RandomForestClassifier
from sktime.transformations.panel.compose import ColumnConcatenator

# X: pandas DataFrame of shape (n_instances, n_timesteps)
# y: labels

# Example pipeline: transform series -> extract simple features -> classifier
# (Detailed code depends on your dataset format)
```

## Notes & learning path

- Start: implement DTW + k-NN on a small UCR dataset to understand distance-based methods.
- Next: try feature extraction (`tsfresh`) + RandomForest.
- Then: test ROCKET / MiniROCKET for strong baseline performance.
- Finally: experiment with 1D-CNNs and Transformers for domain-specific improvements.

## References

- UCR Time Series Classification Archive: https://www.timeseriesclassification.com/
- ROCKET paper / implementation: https://github.com/angus924/rocket
- sktime: https://github.com/sktime/sktime
- tslearn: https://github.com/tslearn-team/tslearn
- tsfresh: https://github.com/blue-yonder/tsfresh

---

© 2025 hwHoai | [github.com/hwHoai](https://github.com/hwHoai) | License: MIT

# EEG Signal Specifications & Mental State Classification

**Source:** *Classification of Relaxation and Concentration Mental States with EEG (Information 2021, 12, 187)*.

## Part 1: EEG Frequency Band Specifications

This section outlines the technical frequency ranges for each wave band as defined in the study's introduction.

### 1\. Delta ($\delta$)

  * **Frequency Range:** $< 4$ Hz
  * **Technical Note:** Lowest frequency band defined.

### 2\. Theta ($\theta$)

  * **Frequency Range:** $4 - 7$ Hz
  * **Technical Note:** Frequency band immediately above Delta.

### 3\. Alpha ($\alpha$)

  * **Frequency Range:** $8 - 12$ Hz
  * **Technical Note:** Primary indicator for the **Relaxation** state in this study.

### 4\. Beta ($\beta$)

  * **Frequency Range:** $13 - 30$ Hz
  * **Technical Note:** Primary indicator for the **Concentration** state. The study emphasizes that boundaries (e.g., 12 vs 13 Hz) can vary in literature, so combining bands is recommended.

### 5\. Gamma ($\gamma$)

  * **Frequency Range:** $> 30$ Hz
  * **Technical Note:** High-frequency band. While often omitted in simple relaxation studies, it critical for improving accuracy in detecting **Concentration**.


## Part 2: Classification for Mental State

This section details the specific signals, experimental measurement methods, algorithms, and performance metrics used to identify mental states.

### 1\. Defining the States & Signal Characteristics

The study identifies states based on the dominance of specific energy bands.

| Target State | Cognitive Activity | Signal Characteristic (Key Indicator) |
| :--- | :--- | :--- |
| **Relaxation** | Mind idle, resting, no specific thought. | **High Alpha ($\alpha$) Energy**. |
| **Concentration** | Effortful mental work (e.g., reciting numbers backwards). | **High Beta ($\beta$) Energy**. <br> *Note: Combining Beta with Gamma ($\gamma$) yields better detection*. |

### 2\. Measurement Methodology (The Protocol)

To ensure accurate classification with low-cost hardware, the following protocol was strictly enforced:

  * **Hardware:** Neurosky Mindwave (Single channel `Fp1`, \< $300).
  * **Artifact Control:** Subjects must **close their eyes** to eliminate eye-blinking artifacts (EOG).
  * **Data Slicing:**
      * Recording duration: 10 seconds per sample.
      * **Active Analysis Window:** Only data from **second 6 to second 8** (3 seconds total) is used to ensure signal stability.

### 3\. Algorithm & Processing Configuration

The study compared multiple configurations to find the optimal classification pipeline.

  * **Feature Extraction Strategy:**
      * **Winning Feature Set:** $\alpha + \beta + \gamma$ (Alpha, Beta, and Gamma bands combined).
      * **Optimal Bandwidth:** **4 Hz**. (Slicing frequency bands into 4 Hz chunks performed better than 1 Hz or 2 Hz slices).
  * **Classifier Selection:**
      * **Primary Algorithm:** **SVM** (Support Vector Machine) with RBF Kernel (Cost=32, Gamma=8).
      * *Alternative:* BPNN (Back Propagation Neural Network) was tested but showed lower stability and accuracy.
  * **Model Type:**
      * **Individualized Model (Mandatory):** Models must be trained on the specific user's data.
      * *Generic Model:* Training one model for all users failed (Accuracy \~57%).

### 4\. Performance Metrics

Using the optimal configuration ($\alpha+\beta+\gamma$, 4 Hz bandwidth, Individualized SVM):

  * **Average Accuracy:** **\> 80%** across all subjects.
  * **Peak Performance:** Some subjects achieved accuracy **\> 90%**.
  * **Consistency:** SVM showed higher consistency (points above mean in Bland-Altman plot) compared to Neural Networks.
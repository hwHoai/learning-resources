# Machine Learning Basics: Introduction & Classification

## Table of Contents
* [1. Core Concept](#1-core-concept-ml-vs-traditional-programming) - Defines the fundamental shift from rule-based programming to data-driven pattern recognition.
* [2. Algorithm Comparison Matrix](#2-algorithm-comparison-matrix) - A quick lookup table comparing Supervised, Unsupervised, Semi-supervised, and Reinforcement learning.
* [3. Supervised Learning](#3-supervised-learning) - Details on training with labeled data (Regression & Classification).
* [4. Unsupervised Learning](#4-unsupervised-learning) - Details on finding patterns in unlabeled data (Clustering).
* [5. Semi-supervised Learning](#5-semi-supervised-learning) - Bridging the gap between supervised and unsupervised learning to save labeling costs.
* [6. Reinforcement Learning](#6-reinforcement-learning) - Learning through interaction and feedback loops.

---

## 1. Core Concept: ML vs. Traditional Programming

* **Traditional Programming:**
    * Process: `Data` + `Rules` (Code) -> `Output`
    * Logic: You explicitly write the logic (if/else) to process data.
* **Machine Learning:**
    * Process: `Data` + `Output` (Answers) -> `Rules` (Model)
    * Logic: The algorithm learns the logic (function) that maps inputs to outputs based on historical data.

---

## 2. Algorithm Comparison Matrix

| Learning Type | Data Input | Core Mechanism | Goal | Example |
| :--- | :--- | :--- | :--- | :--- |
| **Supervised** | **Labeled** (Input + Correct Output) | Mapping X to Y | Minimize prediction error | Spam Filtering, House Price Prediction |
| **Unsupervised** | **Unlabeled** (Raw Data) | Finding Structure | Discovery of patterns/groups | Customer Segmentation, Anomaly Detection |
| **Semi-supervised**| **Small Labeled + Large Unlabeled** | Pseudo-labeling | Improve accuracy with min. labeling cost | Medical Imaging, Google Photos labeling |
| **Reinforcement**| **State + Reward/Penalty** | Trial & Error | Maximize total reward | Game AI, Robot Navigation |
||||||

## 3. Supervised Learning
The most common type of ML in industry. The model learns from a "teacher" (the labeled dataset).

### A. Regression
1.  **Why "Regression"? (Historical Context)**
    * **Origin:** Coined by Francis Galton (19th Century) while studying human heights.
    * **Phenomenon:** "Regression to the mean". Extreme characteristics (e.g., very tall parents) tend to revert towards the average in the next generation.
    * **Modern Meaning:** In ML, it refers to statistical methods used to estimate the relationships between variables (finding the trend).

2.  **The Core Problem**
    * **Goal:** To predict a **continuous quantity** (a specific number), not a category.
    * **Mechanism:** It fills the gaps between known data points. By finding a mathematical function (curve/line) that fits the historical data, it allows us to estimate values for unseen inputs (Interpolation & Extrapolation).
    * **Contrast with Classification:**
        * Regression -> Output: 45.2, 1000, -5 (Infinite possibilities).
        * Classification -> Output: Yes/No, Cat/Dog (Finite set of options).

3.  **Real-world Applications of Output**
    The output is always a real number, used for quantitative decision making:
    * **Financial Forecasting:** Predicting exact stock prices or currency exchange rates.
    * **Demand Planning:** Estimating the number of product units to be sold next month based on marketing spend.
    * **Medical/Time Series:** Predicting the exact value of a biosignal (e.g., Blood Glucose level) in the next hour based on historical trends.
    * **Real Estate:** Automated Valuation Models (AVM) like Zillow/Batdongsan to estimate house prices.

    ### Linear Regression
    The "Hello World" of Machine Learning. It serves as the foundation for understanding how models learn from data.

    1. **Concept**
    * **Goal:** Find a straight line (or hyperplane) that best fits the data points.
    * **Input:** Feature vector $\mathbf{x}$.
    * **Output:** A continuous number $y$ (e.g., Price, Temperature, Speed).
    * **Core Idea:** $y \approx \text{Weight} \times \text{Input} + \text{Bias}$.

    2. **Mathematical Model**
    Instead of $y = ax + b$, we use Linear Algebra notation for efficiency:
    $$y \approx \mathbf{\bar{x}}\mathbf{w}$$

    * **The Weights ($\mathbf{w}$):** The "knobs" we need to tune. $\mathbf{w} = [w_0, w_1]^T$.
    * **The "Bias Trick" ($\mathbf{\bar{x}}$):** We add a **1** to the beginning of the input vector $\mathbf{x}$ to handle the bias term ($w_0$) within the matrix multiplication.

    3. **Matrix Operations Crash Course**
    * **$\mathbf{\bar{X}}$ (Extended Matrix):** Data matrix with an added column of **1s**.
    * **$\mathbf{X}^T$ (Transpose):** Flipping the matrix (Rows $\leftrightarrow$ Columns).
    * **Dot Product:** "Row times Column". Essential for calculating predictions efficiently.

    4. **Loss Function (MSE)**
    Used to measure how "wrong" the model is.
    $$\mathcal{L}(\mathbf{w}) = \frac{1}{2}\sum_{i=1}^{N} (y_{true} - \mathbf{\bar{x}}_i\mathbf{w})^2$$
    * **Objective:** Find $\mathbf{w}$ that minimizes this value (Bottom of the valley).

    5. **The Solution: Normal Equation**
    The exact mathematical formula to find the optimal $\mathbf{w}$ immediately (No iteration).
    $$\mathbf{w} = (\mathbf{\bar{X}}^T\mathbf{\bar{X}})^{-1} \mathbf{\bar{X}}^T\mathbf{y}$$

    6. **Python Implementation (Numpy)**
    ```python
    import numpy as np

    # 1. Prepare Data
    X = np.array([[147], [150], [153], [158], [163], [165], [168], [170], [173], [175], [178], [180], [183]])
    y = np.array([[ 49], [ 50], [ 51], [ 54], [ 58], [ 59], [ 60], [ 62], [ 63], [ 64], [ 66], [ 67], [ 68]])

    # 2. Bias Trick (Create X_bar)
    one = np.ones((X.shape[0], 1))
    Xbar = np.concatenate((one, X), axis = 1)

    # 3. Calculate w using Normal Equation
    A = np.dot(Xbar.T, Xbar)
    b = np.dot(Xbar.T, y)
    w = np.dot(np.linalg.pinv(A), b)

    # 4. Result & Prediction
    print(f"Model: y = {w[1][0]:.2f}*x + {w[0][0]:.2f}")
    
    ```
    ---

### B. Classification
* **Goal:** Predict a **categorical label** (Class).
* **Output:** A discrete category (e.g., Yes/No, Cat/Dog, Spam/Ham).
* **Key Use Cases:**
    * Medical diagnosis (Benign vs. Malignant).
    * Image recognition.

---

## 4. Unsupervised Learning
Used when data has no labels. The model must infer structure on its own.

### A. Clustering
* **Goal:** Group similar data points together.
* **Mechanism:** Measures similarity (usually distance) between data points.
* **Key Use Cases:**
    * Market segmentation (grouping users by behavior).
    * Grouping similar news articles.

---

## 5. Semi-supervised Learning
A hybrid approach used when labeling data is expensive or time-consuming.

* **Concept:** Uses a small amount of labeled data to train a basic model, which then predicts labels for the massive unlabeled dataset (Pseudo-labeling), and finally retrains on the combined data.
* **Key Use Cases:**
    * **Medical Analysis:** Using a few expert-diagnosed scans to help the AI learn from thousands of raw scans.
    * **Speech Analysis:** Labeling audio files is very time-consuming.

---

## 6. Reinforcement Learning (RL)
Focuses on training an **Agent** to make a sequence of decisions.

* **Core Components:**
    * **Agent:** The learner.
    * **Environment:** The world the agent interacts with.
    * **Action:** What the agent does.
    * **Reward/Penalty:** Feedback from the environment.
* **Mechanism:** The agent explores the environment and learns a **Policy** (strategy) to maximize cumulative rewards over time (Trial & Error).
* **Key Use Cases:**
    * Self-driving cars (Penalty for hitting an object, Reward for staying in lane).
    * Game playing AI (AlphaGo, Dota 2 Bots).
The goal is to create a minimum viable (runnable with kedro run) end to end workflow demonstrating time series tasks, specifically time series classification and clustering. More concretely, I would like to work in a Kedro framework (this is, leveraging Kedro and utilizing the concept of pipelines, namespaces, hooks, nodes, and kedro viz. I would like to have an ingestion pipeline, where the data is not actually ingested but created on the fly (dummy data, for example purposes). I need to also have a preprocessing pipeline creating the features for the model. Please keep in mind each pipeline does not need to be anything fancy or complex. On the contrary, anything simple with small pure functions as nodes will suffice. I need a modelling pipeline and finally an inference pipeline. There has to be some visualization somewhere. The whole point of this repo is to share with my team of software developers coming from Thoughtworks and working as contractors with us, unfamiliar with data science but proficient software developers, the main ideas behind time series classification and clustering. the scenario is as follows: we have production data for wells in an unconventional field (shale oil in midland basin). We have to identify, using machine learning, wells to be selected as candidates for acid treatment, which will improve their performance. We must identify said wells (poor performers, candidates for acid treatment). My original idea was to do classification (binary classification with two classes: good performer, poor performer) but someone suggested we just throw all wells data to a clustering algorithm, and if resulting groups/clusters are more than one, then there's a poor performing group and a good performing group. The goal for the session tomorrow is to demonstrate the two workflows, show differences, and contrast their applications. We will need to highlight the need for balanced samples when we do classification. We are currently using sktime, anything you can leverage from sktime is valuable. Lets first think deeply through how to organize the example, what to showcase at each step, and propose a plan to implement.



A conversation with Claude AI Chat led to this proposed README.md for the repository, please read through this as it proposes a detailed approach and highlights teaching points we want to be able to run.



\# Time Series Classification vs Clustering for Well Performance



A Kedro demonstration project comparing \*\*supervised\*\* and \*\*unsupervised\*\* machine learning approaches for identifying underperforming oil wells in unconventional (shale) reservoirs.



\## 🎯 Business Context



\*\*Problem\*\*: We have production data from oil wells in the Midland Basin (Permian). We need to identify wells that are candidates for \*\*acid treatment\*\* to improve their performance.



\*\*Two Approaches\*\*:

1\. \*\*Classification (Supervised)\*\*: Learn from labeled examples to predict "good" vs "poor" performers

2\. \*\*Clustering (Unsupervised)\*\*: Discover natural groupings and assess if they align with performance



\## 🏗️ Project Structure



```

timeseries-pipelines/

├── conf/

│   ├── base/

│   │   ├── catalog.yml          # Data definitions

│   │   └── parameters.yml       # Configurable parameters

│   └── local/

├── src/timeseries\_pipelines/

│   ├── pipelines/

│   │   ├── ingestion/           # Synthetic data generation

│   │   ├── preprocessing/       # Data preparation

│   │   ├── classification/      # Supervised learning

│   │   ├── clustering/          # Unsupervised learning

│   │   └── evaluation/          # Visualizations

│   └── pipeline\_registry.py     # Pipeline registration

├── data/                        # Generated data (created at runtime)

└── pyproject.toml

```



\## 🚀 Quick Start



```bash

\# Install dependencies

uv sync



\# Run the entire pipeline

uv run kedro run



\# Run specific pipelines

uv run kedro run --pipeline ingestion

uv run kedro run --pipeline classification

uv run kedro run --pipeline clustering



\# Visualize the pipeline graph

uv run kedro viz

```



\## 📊 The Pipelines



\### 1. Ingestion Pipeline

\*\*Purpose\*\*: Generate synthetic well production data



\*\*Key Features Generated\*\*:

\- Production time series (365 days of daily data)

\- "Ideal" production curves (what software predicts)

\- Reservoir assignments (categorical feature)

\- Ground truth labels (good/poor performer)



\*\*Teaching Point\*\*: We deliberately create \*\*80/20 class imbalance\*\* (80% good, 20% poor) to demonstrate real-world challenges.



\### 2. Preprocessing Pipeline

\*\*Purpose\*\*: Prepare data for machine learning



\*\*Transformations\*\*:

\- Compute oil variance (actual - ideal production)

\- Z-score normalize time series

\- Stratified train/test split



\*\*Teaching Point\*\*: Stratified splitting ensures class proportions are maintained in both train and test sets.



\### 3. Classification Pipeline

\*\*Purpose\*\*: Demonstrate supervised learning approach



\*\*Two Models\*\*:

1\. \*\*Naive Classifier\*\*: KNN without class balancing

&nbsp;  - High accuracy but poor minority class recall!

&nbsp;  

2\. \*\*Balanced Classifier\*\*: KNN with oversampling

&nbsp;  - Better recall for poor performers



\*\*Teaching Point\*\*: A model predicting "good" always would get 80% accuracy but miss ALL poor performers!



\### 4. Clustering Pipeline

\*\*Purpose\*\*: Demonstrate unsupervised learning approach



\*\*Process\*\*:

1\. Evaluate silhouette scores for k=2,3,4

2\. Train TimeSeriesKMeans with DTW distance

3\. Analyze cluster-label alignment



\*\*Teaching Point\*\*: Clusters may NOT correspond to performance categories. They might capture reservoir differences or other patterns.



\### 5. Evaluation Pipeline

\*\*Purpose\*\*: Generate visualizations



\*\*Outputs\*\* (in `data/08\_reporting/figures/`):

\- `sample\_timeseries.png` - What does production data look like?

\- `class\_distribution.png` - Visualizing the imbalance problem

\- `confusion\_matrix\_naive.png` - Naive classifier results

\- `confusion\_matrix\_balanced.png` - Balanced classifier results

\- `silhouette\_scores.png` - Choosing number of clusters

\- `cluster\_centroids.png` - What patterns did clustering find?

\- `cluster\_composition.png` - Do clusters match labels?

\- `comparison\_summary.png` - Side-by-side comparison



\## 🔑 Key Concepts Demonstrated



\### Time Series Classification

\- Using \*\*DTW (Dynamic Time Warping)\*\* for time series similarity

\- K-Nearest Neighbors for classification

\- \*\*Class imbalance problem\*\* and solutions

\- Proper evaluation metrics (recall, precision, F1)



\### Time Series Clustering

\- \*\*TimeSeriesKMeans\*\* with DTW distance

\- \*\*Silhouette score\*\* for choosing k

\- Post-hoc cluster validation

\- Why clusters ≠ your business objective



\### Kedro Framework

\- \*\*Pure functions\*\* as nodes (no side effects)

\- \*\*Data Catalog\*\* for data versioning

\- \*\*Parameters\*\* for configuration

\- \*\*Pipeline composition\*\* (pipelines can be combined)



\## ⚙️ Configuration



Key parameters in `conf/base/parameters.yml`:



```yaml

ingestion:

&nbsp; n\_wells: 100                 # Number of synthetic wells

&nbsp; good\_performer\_ratio: 0.8    # 80% good, 20% poor (imbalanced!)

&nbsp; series\_length: 365           # Days of production



classification:

&nbsp; knn:

&nbsp;   n\_neighbors: 5

&nbsp;   distance\_metric: "dtw"     # Dynamic Time Warping



clustering:

&nbsp; n\_clusters\_final: 2          # Final number of clusters

&nbsp; distance\_metric: "dtw"

```



\## 📈 Expected Results



\### Classification Results

| Metric | Naive | Balanced |

|--------|-------|----------|

| Accuracy | ~80% | ~75% |

| Recall (Poor) | ~30% | ~70% |

| Precision (Poor) | ~50% | ~45% |



\*\*Interpretation\*\*: Balanced model catches more poor performers (higher recall) even though overall accuracy is lower.



\### Clustering Results

\- Clusters may or may not align with labels

\- Often captures reservoir patterns instead of performance

\- Demonstrates why unsupervised ≠ business-aligned



\## 🎓 Discussion Questions for Session



1\. \*\*When to use classification vs clustering?\*\*

&nbsp;  - Classification: When you have reliable labels and a clear business objective

&nbsp;  - Clustering: When exploring, no labels, or validating hypotheses



2\. \*\*Why is class imbalance dangerous?\*\*

&nbsp;  - High accuracy can be deceiving

&nbsp;  - Model learns to predict majority class

&nbsp;  - Minority class (often the important one) gets ignored



3\. \*\*Why might clustering fail to find performance groups?\*\*

&nbsp;  - Data structure might be driven by other factors (reservoir, completion design)

&nbsp;  - Performance might not be the dominant pattern in the data

&nbsp;  - Clustering finds \*statistical\* patterns, not \*business\* patterns



4\. \*\*What's special about time series ML?\*\*

&nbsp;  - Need specialized distance metrics (DTW)

&nbsp;  - Must preserve temporal ordering

&nbsp;  - Traditional ML assumes independent samples



\## 📚 References



\- \[sktime Documentation](https://www.sktime.net/)

\- \[Kedro Documentation](https://docs.kedro.org/)

\- \[Dynamic Time Warping Explained](https://www.sktime.net/en/latest/examples/classification/feature\_based.html)



\## 👥 Authors



Created for the Assetization Team working session on Time Series Data Science.


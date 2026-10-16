# Bayesian Networks for Heart Disease Prediction

**An expert-designed network versus one learned from the data: which predicts better, and which can you trust?**

*EDAB 6808 Assignment 3 · B.Com Business Analytics (Hons), University of the Free State · Pair project, 2026*

---

## Problem
Build and compare two Bayesian networks that predict heart disease: one with a structure designed by hand from clinical reasoning (the **expert DAG**), and one with a structure learned automatically from the data using `bnlearn` (the **learned DAG**). Compare their structures, probability tables, inferences and predictive performance.

## Data
| Source | What | Size |
|---|---|---|
| [UCI Heart Disease (Cleveland)](https://archive.ics.uci.edu/dataset/45/heart+disease) | 13 clinical predictors (age, sex, chest-pain type, blood pressure, cholesterol, fasting blood sugar, resting ECG, max heart rate, exercise angina, ST depression, ST slope, vessels affected, thalassemia) plus a 0–4 disease score | 303 patients, 14 columns |

The 0–4 score was converted to a binary target, `heartdisease_bin` (45.9% positive). The data file is `data/heart_disease_cleveland.csv`.

## Tools
Python · Google Colab · `bnlearn` · `pgmpy` · `pandas` · `numpy` · `scikit-learn` · `networkx` · `matplotlib`

## Method
1. **Preparation:** a stratified 80/20 split (242 train, 61 test) with a fixed seed. The six missing values were treated as missing, not zero, and imputed using training data only.
2. **Discretisation:** age, resting blood pressure, cholesterol and maximum heart rate were split into Low, Medium and High bands using **training-set tertiles**. ST depression (`oldpeak`) was split into None, Moderate and High. All cut points come from the training set only.
3. **Expert DAG:** 14 nodes and **21 edges**, each justified clinically. Risk factors point into disease, and disease points out to symptoms and test findings.
4. **Learned DAG:** hill-climbing search scored by **BIC**, starting from an empty graph with no forced or forbidden edges (tabu list length 100). It found **11 edges**.
5. **Parameter learning:** conditional probability tables for all 14 nodes in each network, using a Bayesian estimator (BDeu prior, equivalent sample size 5). Both fitted networks are saved as `.pkl` files.
6. **Inference:** posterior probability of disease for different evidence combinations, and the effect of changing one variable at a time.
7. **Evaluation:** accuracy, precision, recall, F1, ROC-AUC and Brier score on the 61 held-out patients, against a majority-class baseline, with a paired bootstrap of the AUC difference.

## Key findings
| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | Brier |
|---|---|---|---|---|---|---|
| Expert network (21 edges) | 0.853 | 0.807 | **0.893** | 0.848 | **0.939** | **0.105** |
| Learned network (11 edges) | **0.869** | **0.857** | 0.857 | **0.857** | 0.918 | 0.108 |
| Majority-class baseline | 0.541 | 0 | 0 | 0 | 0.500 | 0.248 |

- **Neither network is clearly better.** The bootstrap 95% interval for the AUC difference (−0.098 to 0.039) includes zero.
- **Three variables dominate:** thalassemia result, chest-pain type and number of vessels affected. They form the learned network's entire Markov blanket.
- **Vessels affected moves the risk the most.** Changing it from 0 to 3 raises P(disease) from 3.2% to 37.9% in the learned network.
- **Asymptomatic chest pain** appears in 73.7% of diseased patients versus 25.9% of non-diseased patients.
- **The structure comparison:** 5 shared edges, 2 reversed, 14 only in the expert network and 4 only in the learned network.
- **Structure learning is not causal discovery.** Two learned edges, *max heart rate → age* and *thalassemia → sex*, are clinically impossible.
- **The expert network's target table has 108 rows**, and 39 of them have no supporting training patients. The learned network (57 parameters versus 194) is better supported by the data.

![Expert DAG and learned DAG](images/dag_comparison.png)

## How to run
1. Open the notebook in Google Colab: [Open in Colab](COLAB_LINK_HERE)
2. Upload `data/heart_disease_cleveland.csv` when prompted, or place it in the working folder.
3. **Runtime → Run all.** The notebook saves `expert_bayesian_network.pkl`, `learned_bayesian_network.pkl`, `preprocessing.pkl` and the CPT CSVs.

## Repository structure
```
bayesian-networks-heart-disease/
├── README.md
├── bayesian_networks_heart_disease.ipynb
├── data/
│   └── heart_disease_cleveland.csv
├── models/
│   ├── expert_bayesian_network.pkl
│   ├── learned_bayesian_network.pkl
│   └── preprocessing.pkl
├── cpts/
│   └── (one CSV per node, per network)
└── images/
    ├── expert_dag.png
    ├── learned_dag.png
    └── dag_comparison.png
```

## Report and presentation
- [Full report](REPORT_LINK_HERE) (view-only)
- [Presentation slides](SLIDES_LINK_HERE) (view-only)
- [Portfolio page](EPORTFOLIO_LINK_HERE)

## References
- Detrano, R., et al. (1989). International application of a new probability algorithm for the diagnosis of coronary artery disease. *American Journal of Cardiology, 64*(5), 304–310.
- Koller, D., & Friedman, N. (2009). *Probabilistic Graphical Models: Principles and Techniques*. MIT Press.
- Scutari, M. (2010). Learning Bayesian networks with the bnlearn R package. *Journal of Statistical Software, 35*(3), 1–22.
- Tanaka, H., Monahan, K. D., & Seals, D. R. (2001). Age-predicted maximal heart rate revisited. *Journal of the American College of Cardiology, 37*(1), 153–156.

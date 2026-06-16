# 🏢 Workplace Diversity & Salary Fairness Analysis
> **An end-to-end HR analytics project** analyzing internal employee data to detect compensation disparities and assess whether the company treats all employees fairly across gender, department, and seniority.

[![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)](https://python.org)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-ML%20Models-orange?logo=scikitlearn)](https://scikit-learn.org)
[![pandas](https://img.shields.io/badge/pandas-Data%20Analysis-150458?logo=pandas)](https://pandas.pydata.org)
[![Status](https://img.shields.io/badge/Status-Complete-brightgreen)]()

---

## 🧭 Project Overview

Workplace fairness is a data problem before it is a policy problem. This project analyzes internal company data to determine whether employees are being compensated equitably across gender, department, and seniority level. Using recursive tree traversal to classify employees into six organizational levels and aggregate total headcount managed, the analysis builds a salary prediction model and audits its features to identify whether protected attributes like gender have statistically significant influence on pay outcomes.

**Key questions this project answers:**
- How do you classify employees into organizational levels from raw hierarchy data alone?
- How do you calculate total indirect reports for every node in an org tree?
- Can salary be predicted accurately from available employee attributes?
- Is this company treating all employees fairly, and where are the biggest gaps?

---

## 🗂️ Project Structure

```
workplace-diversity-analysis/
│
├── notebooks/
│   └── diversity_analysis.ipynb    # Full analysis end to end
│
├── data/
│   ├── company_hierarchy.csv       # employee_id, boss_id, dept
│   └── employee.csv                # salary, sex, degree, experience, bonus
│
└── README.md
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Data Wrangling | Python, pandas |
| Hierarchy Analysis | NetworkX (recursive descendant traversal) |
| Encoding | pandas get_dummies, ordinal mapping |
| Modeling | scikit-learn (Linear Regression, Random Forest, Gradient Boosting) |
| Evaluation | RMSE, R2, practical 25% accuracy metric |
| Visualization | matplotlib |

---

## 🏗️ Architecture

```
> Two source tables (hierarchy + employee metadata) joined into a single
> modeling dataset. Org tree traversed bottom-up to assign levels and
> aggregate headcount. Three regression models trained and compared.
> Fairness audited via coefficients, feature importance, and visual analysis.
```

---

## 📦 Analysis Tasks

### Task 1 — Employee Level Classification
- Identified CEO as the single employee with null boss_id
- Classified employees bottom-up using set membership logic:
  - **IC**: employee_id never appears as a boss_id
  - **MM**: direct supervisors of ICs
  - **D**: direct supervisors of MMs
  - **VP**: direct supervisors of Directors
  - **E**: direct supervisors of VPs
  - **CEO**: null boss_id, top of hierarchy
- Assigned level labels via `.map()` to a new `employee_level` column

### Task 2 — Total People Managed
- Built a directed graph using NetworkX from boss_id to employee_id edges
- Used `nx.descendants()` to count all indirect reports per employee
- ICs return 0, each manager accumulates full downstream headcount automatically

### Task 3 — Salary Prediction
- Merged hierarchy and employee tables on employee_id
- Excluded single CEO record ($700K, zero variance) — distorted Linear Regression coefficients significantly
- Encoded features: ordinal for level and degree, one-hot for department, binary for sex
- Trained three models on 80/20 train/test split

### Task 4 — Fairness Analysis
- Visualized average salary by gender, department, level, and degree
- Cross-referenced with model coefficients (Linear Regression) and feature importance (Random Forest, Gradient Boosting)
- Identified department as the dominant salary driver and HR pay gap as the critical fairness concern

---

## 📊 Model Comparison

| Model | RMSE | R2 |
|---|---|---|
| Linear Regression | $73,675 | 0.35 |
| Random Forest | $76,989 | 0.29 |
| Gradient Boosting | $73,899 | 0.35 |

> All three models plateau at ~0.35 R2. The available features explain only 35% of salary variance. Remaining 65% is likely driven by performance ratings, tenure, and geographic location — not present in this dataset. Practical accuracy: **50% of predictions land within 25% of actual salary** (benchmark from solution: 49%).

---

## 🔑 Key Design Decisions

**Why NetworkX instead of a manual recursive function?**
NetworkX is purpose-built for graph traversal. `nx.descendants()` handles the full tree walk internally with no custom recursion needed. A manual recursive function would work but introduces more opportunity for bugs and is harder to read. For a DEA story, I can explain both approaches — NetworkX in code, recursive logic in the interview.

**Why exclude the CEO from modeling?**
One employee, one salary ($700K), zero variance. Standard deviation is NaN. Linear Regression tried to memorize a single data point and produced a dept_CEO coefficient of -919K, distorting every other coefficient in the model. The CEO record teaches the model nothing and actively misleads it. Both runs (with and without CEO) are shown in the notebook to demonstrate the impact.

**Why Linear Regression first despite tree models being more powerful?**
Coefficients are directly interpretable as dollar impacts. If sex_encoded carries a large negative coefficient after controlling for department and experience, that is a one-number fairness finding any HR executive can understand. Random Forest and Gradient Boosting give feature importance but not direction or magnitude. Linear Regression is the fairness lens; tree models are the accuracy benchmark.

**Why one-hot encoding for department but ordinal for level and degree?**
Department has no natural order. Engineering is not greater than or less than Marketing. One-hot treats each department independently. Level (IC through CEO) and degree (High School through PhD) both have meaningful natural order, so ordinal encoding preserves that signal without creating unnecessary columns.

---

## 🔍 Key Findings

**Finding 1: No Direct Gender Discrimination Detected**
Males average ~$200K vs females ~$172K, a $28K surface gap. However all three models assigned near zero importance to gender after controlling for other factors. The gap is likely explained by female employees being concentrated in HR, the lowest paid department — not direct pay discrimination.

**Finding 2: Severe Department Pay Gap (Critical)**
HR averages $84,560 vs Engineering at $243,525, a $160K gap. dept_HR was the dominant salary predictor across all models at 58% to 80% feature importance. Most urgent fairness concern in the dataset.

**Finding 3: Level Progression is Fair**
Salary increases consistently from IC through Executive. Seniority is being rewarded appropriately. No fairness concern here.

**Finding 4: Degree is Not Being Rewarded**
PhD and High School employees earn essentially the same salary. Inconsistent with standard compensation practices if the company values or requires advanced education in hiring.

---

## 📋 HR Recommendations

**1. Investigate HR Compensation Urgently**
Benchmark HR salaries against market rates and adjust if below industry standard.

**2. Audit Gender Distribution by Department**
Investigate whether female employees are disproportionately placed in lower paying departments and whether promotion practices are equitable.

**3. Revisit Degree Based Pay Policy**
If advanced degrees are required in hiring, compensation should reflect that investment.

**4. Collect Better Data**
65% of salary variance is unexplained. Add performance ratings, tenure, and location data to enable stronger future analysis.

---

## 🚀 What I'd Improve in Production

- Add statistical significance testing (t-test or regression p-values) to formalize the gender pay gap finding
- Include interaction terms (e.g. sex x department) to detect within-department gender gaps
- Pull in external market salary benchmarks via API for direct comparison
- Build a Tableau dashboard for HR to explore salary distributions interactively
- Add confidence intervals around salary predictions, not just point estimates

---

## 📈 Summary Results

- **10,000 employees** analyzed across 4 departments and 6 organizational levels
- **HR department earns $160K less** than Engineering on average — most critical finding
- **Gender pay gap disappears** after controlling for department (sex coefficient = +$1,311 in Linear Regression)
- **Degree level has no salary impact** — PhD and High School earn essentially the same
- **50% of salary predictions** land within 25% of actual salary across all models

---

## 🧑‍💻 About This Project

This project was built as part of the Data Engineering Academy curriculum to demonstrate end-to-end analytical thinking on a real-world HR fairness problem:
- **Hierarchy reasoning** — derived org levels from raw relational data using graph traversal, not lookup tables
- **Outlier handling** — identified and documented CEO record distortion with before/after model comparison
- **Fairness thinking** — separated surface-level gaps from controlled findings using model coefficients and feature importance
- **Business storytelling** — findings and recommendations written for a non-technical HR audience, not just a data team

**Author:** Talib Hussain
**GitHub:** [github.com/tbhatti211-wq](https://github.com/tbhatti211-wq)
**LinkedIn:** [linkedin.com/in/talhussain](https://linkedin.com/in/talhussain)

---

## 📄 License
MIT License — analysis code free to use and adapt.

---
*Last updated: June 2026*
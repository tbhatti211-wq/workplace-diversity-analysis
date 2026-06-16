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

| Decision | Why |
|---|---|
| **NetworkX for headcount** | Org chart is a graph. `nx.descendants()` returns full downstream reports in one line. Knew the recursive version too, but less to break this way. |
| **Dropped the CEO** | One row, fixed $700K, zero variance. Linear Regression latched onto it and threw a -919K coefficient that skewed everything. Kept both runs in the notebook to show the impact. |
| **Linear Regression first** | Coefficients are in dollars, so the sex coefficient answers the fairness question directly. Tree models predict better but only rank features, not direction or size. |
| **One-hot vs ordinal** | Departments have no order, so one-hot. Level and degree do (IC→CEO, HS→PhD), so ordinal keeps the ranking in one column. |

---

## 🔍 Key Findings

| Finding | Detail |
|---|---|
| **Gender gap fades after controls** | Raw gap is ~$28K (men $200K, women $172K), but all models drop gender to near-zero weight once department, level, and experience are in. Likely a distribution issue: more women in HR, the lowest paid dept. |
| **Department gap is the real problem** | HR $84,560 vs Engineering $243,525, a $160K spread. dept_HR was the top predictor in every model (58–80% importance). This is the one to act on. |
| **Level progression is healthy** | Pay rises cleanly IC → Executive. Seniority is rewarded as expected. |
| **Degree barely moves pay** | PhD and high school land at roughly the same salary. Out of step if degrees are a hiring expectation. |

---

## 📋 HR Recommendations

1. **Fix HR pay first.** Benchmark HR roles against market and adjust. The $160K gap is the headline.
2. **Check why women cluster in HR.** The gender gap comes from placement, not equal-role pay. Audit hiring and promotion.
3. **Decide if degrees should matter.** They move pay by nothing right now. Align the pay structure with hiring expectations.
4. **Collect richer data.** Two thirds of salary variance is unexplained. Add performance, tenure, and location.

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

Built as part of the Data Engineering Academy curriculum. The interesting part was not the modeling but the reasoning around it: deriving org levels from raw relational data instead of a lookup column, catching the CEO record before it wrecked the regression, and being careful to separate the surface gender gap from what actually holds up once you control for department. The findings and recommendations are written for an HR reader, not a data team, since that is who would actually act on them.

**Author:** Talib Hussain
**GitHub:** [github.com/tbhatti211-wq](https://github.com/tbhatti211-wq)
**LinkedIn:** [linkedin.com/in/talhussain](https://linkedin.com/in/talhussain)

---

## 📄 License
MIT License — analysis code free to use and adapt.

---
*Last updated: June 2026*

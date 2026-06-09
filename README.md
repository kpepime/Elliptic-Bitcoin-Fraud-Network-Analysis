# Elliptic Bitcoin Fraud Network Analysis

**Uncovering Fraud Networks in a Blockchain Payment System**  
Blockchain Data Analytics  

---

## Overview

End-to-end fraud detection pipeline built on the [Elliptic Bitcoin Transaction Dataset](https://www.kaggle.com/datasets/ellipticco/elliptic-data-set), a real-world graph of 203,769 Bitcoin transactions used in published academic research. The project models transaction flows as a directed weighted network, computes structural risk scores for every node, detects two laundering patterns (layering and structuring), and delivers a regulatory-grade compliance report and interactive dashboard.

---

## Dataset

| File | Contents |
|---|---|
| `elliptic_txs_features.csv` | 203,769 transactions. 166 anonymised financial features. Time step |
| `elliptic_txs_edgelist.csv` | Directed payment flows (source → destination transaction) |
| `elliptic_txs_classes.csv` | Labels: `illicit`, `licit`, or `unknown` |

The three files are joined on `txId`. Labels cover only 12.8% of transactions, 4,545 illicit and 21,602 licit, with 177,622 unreviewed.

---

## Project Structure

```
Network Analysis.ipynb # Main notebook
fraud_intelligence_dashboard.html # Self-contained interactive dashboard
compliance_report.pdf # Regulatory compliance report
```

---

## Tasks

### Data Preparation
- Loaded all three source files independently before merging to isolate any structural issues early
- Renamed features file columns (`txId`, `time_step`, `lf_1`–`lf_165`), the file ships with no headers
- Left-joined features + classes to retain all 203,769 transactions including the 77% with unknown labels
- Validated edge list against known transaction IDs and removed orphaned edges
- Enriched each edge with source and destination class labels and time steps
- Documented class distribution, time step coverage, and missing value status

### Network Construction & Subgraph Selection
- Built a directed weighted graph (`G_full`), each node is a transaction, each edge is a payment flow, edge weight is connection frequency
- Sampled 20,000 nodes proportionally across weakly connected components (avoids landing in sparse isolated regions)
- Computed betweenness centrality on the sample to identify structural gatekeeper nodes
- Selected top 50 nodes by betweenness as seed nodes; applied a 3-tier ego network expansion with LCC fallback to reach the 5,000-node minimum
- Filtered to the largest weakly connected component before centrality computation so all scores are directly comparable
- Computed four centrality metrics for every node:
  - **In-degree**: how many transactions feed into this node (layering signal)
  - **Out-degree**: how many destinations it reaches (fan-out signal)
  - **Betweenness**: structural gatekeeper role in payment routing
  - **PageRank**: recursive influence from high-traffic connected nodes
- Combined into a **weighted composite risk score** (betweenness 40%, PageRank 30%, in-degree 20%, out-degree 10%) after MinMaxScaler normalisation

**Key finding:** All 17 confirmed illicit nodes in the LCC subgraph ranked below position 2,493 out of 7,880, deliberately peripheral. The highest-risk nodes by composite score are `unknown`, consistent with sophisticated actors avoiding structural visibility.

### Fraud Pattern Detection
The working subgraph was rebuilt to span all 49 time steps after discovering the LCC was restricted to time step 1 only, making temporal detection meaningless. The rebuilt graph (`G_multi`) contains 19,973 nodes across the full dataset.

**Layering** (16 nodes flagged): Nodes receiving from ≥2 distinct sources and redistributing to ≥1 destination across ≥2 time steps. Replicates the layering stage of the three-phase money laundering model, transaction complexity deliberately created to obscure fund origin.

**Fan-out/Structuring** (21 nodes flagged): Nodes sending to ≥3 distinct destinations within a 2-step window. Consistent with smurfing, fragmenting large transfers to stay below CBN reporting thresholds (₦5M individuals/₦10M corporates under SCUML guidelines).

**Tier 1 node (Transaction 8856425):** The only transaction flagged by both patterns simultaneously. Active at time step 4, received from multiple sources, sent to 3 destinations within a 2-step window. No prior investigator review. Highest-priority escalation.

18 unknown nodes were flagged by either pattern with no investigator label, their behaviour alone warrants regulatory review.

### Compliance Report
400 - 500 word report to the Head of Compliance covering:
- Three highest-risk nodes with risk scores and plain-English behavioural explanations
- Most suspicious cluster (7,880-node early-time-step component with 9,164 payment flows)
- Two actionable recommendations with named owners and regulatory references (MLPA 2022, Section 6, NFIU, SCUML)
- One data limitation: the LCC/G_multi graph split and what it means for finding reliability

### Interactive HTML Dashboard
Self-contained HTML file, opens in any browser, no server required, no external data files.

Components:
- **Network graph**: Canvas-rendered, top 300 nodes by risk score, zoomable and pannable, hover tooltips, gold nodes = top 10
- **Risk ranking table**: Top 20 nodes with class pills and pattern badges
- **Time series**: Transaction activity by class across all 49 time steps (dual Y-axis so illicit/licit are visible alongside unknown)
- **Scatter chart**: Risk score vs PageRank, coloured by class

Filter bar updates all four components simultaneously on every click.

---

## Stack

| Tool | Purpose |
|---|---|
| `pandas`, `numpy` | Data loading, cleaning, transformation |
| `networkx` | Graph construction, centrality computation |
| `sklearn` | MinMaxScaler for composite risk score |
| `plotly` | Dashboard charts |
| `matplotlib`, `seaborn` | Static visualisations for notebook export |
| `pyvis` | (Initial network exploration) |

---

## Key Numbers

| Metric | Value |
|---|---|
| Total transactions | 203,769 |
| Working subgraph (G_multi) | 19,973 nodes. 2,399 edges |
| Time steps covered | 49 |
| Confirmed illicit in subgraph | 421 |
| Layering flags | 16 nodes |
| Fan-out flags | 21 nodes |
| Tier 1 (both patterns) | 1 node, TxID 8856425 |
| Unknown nodes flagged | 18 |
| Top composite risk score | 0.636, TxID 111575373 |

---

## How to Run

1. Download the [Elliptic Bitcoin Dataset](https://www.kaggle.com/datasets/ellipticco/elliptic-data-set) from Kaggle
2. Place the three CSV files in the same directory as the notebook
3. Install dependencies:
   ```bash
   pip install pandas numpy networkx scikit-learn plotly matplotlib seaborn pyvis
   ```
4. Run `elliptic_fraud_analysis.ipynb` top to bottom, all outputs are generated in order
5. Open `fraud_intelligence_dashboard.html` by double-clicking it in File Explorer (not through Jupyter)

---

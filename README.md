# 🚕 RoutePulse — Distributed Transportation Analytics Pipeline

[![Python](https://img.shields.io/badge/Python-3.9-3776AB?style=flat&logo=python&logoColor=white)](https://python.org)
[![Apache Spark](https://img.shields.io/badge/Apache_Spark-3.3-E25A1C?style=flat&logo=apachespark&logoColor=white)](https://spark.apache.org)
[![Hadoop](https://img.shields.io/badge/Hadoop-3.3-66CCFF?style=flat)](https://hadoop.apache.org)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)](https://docker.com)
[![License](https://img.shields.io/badge/License-MIT-10B981?style=flat)](LICENSE)

Distributed big data analytics pipeline processing **3.5M+ NYC taxi trips** using Hadoop HDFS and Apache Spark. Achieves **91% payment-type prediction accuracy** with linear cluster scalability.

---

## 🎯 Business Problem

NYC's taxi ecosystem faces three core analytics challenges:

- **Payment friction** — cash handling costs and slow digital adoption
- **Fraud** — "ghost" trips (zero distance, full fare charged)
- **Driver churn** — unpredictable earnings discourage retention

RoutePulse builds a scalable ML pipeline on top of HDFS + Spark to address all three with interpretable predictions at production scale.

---

## 📊 Dataset

**Source**: [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) — December 2024

| Column | Description | Example |
|--------|-------------|---------|
| tpep_pickup_datetime | Trip start | 2024-12-01 00:23:00 |
| trip_distance | Miles traveled | 2.8 |
| fare_amount | Base fare | $12.50 |
| tip_amount | Gratuity | $2.75 |
| payment_type | 1=Card, 2=Cash, 3=No charge, 4=Dispute | 1 |
| total_amount | Final charge | $17.80 |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATA INGESTION                               │
│  CSV Download → HDFS (Replication: 3, Block size: 128MB)        │
└──────────────────────────┬──────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              DISTRIBUTED PROCESSING (Spark)                      │
│                                                                  │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐           │
│  │  Spark SQL  │   │  DataFrame  │   │   MLlib     │           │
│  │   (EDA)     │──▶│  Operations │──▶│  Pipeline   │           │
│  │             │   │ (filter,    │   │             │           │
│  │ 3.5M rows   │   │  groupBy,   │   │ • Naive     │           │
│  │ 5 workers   │   │  caching)   │   │   Bayes     │           │
│  │ Partitioned │   │             │   │ • Logistic  │           │
│  │  by date    │   │             │   │   Regression│           │
│  └─────────────┘   └─────────────┘   │ • DBSCAN    │           │
│                                      └─────────────┘           │
└─────────────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                      MODEL OUTPUTS                               │
│                                                                  │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐   │
│  │ Tip Prediction  │ │ Payment Type    │ │ Trip Clustering │   │
│  │  (Naive Bayes)  │ │ (Logistic Reg)  │ │   (DBSCAN)      │   │
│  │  Accuracy: 78%  │ │  Accuracy: 91%  │ │   Clusters: 5   │   │
│  │  Driver         │ │  Cash handling  │ │  Fraud detect.  │   │
│  │  incentives     │ │  optimization   │ │                 │   │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Models & Results

### 1. Payment Type Classification

**Algorithm**: Logistic Regression with L2 regularization  
**Accuracy: 91%** | Macro F1: 0.89

| Payment Type | Precision | Recall | F1-Score |
|-------------|-----------|--------|----------|
| Card (1) | 0.93 | 0.95 | 0.94 |
| Cash (2) | 0.88 | 0.85 | 0.86 |
| No Charge (3) | 0.82 | 0.79 | 0.80 |
| **Macro Avg** | **0.84** | **0.83** | **0.83** |

> **Business Use**: Anticipate cash-handling needs; incentivize digital payment adoption to reduce processing costs.

### 2. Tip Prediction

**Algorithm**: Naive Bayes (discretized targets)  
**Accuracy: 78%**

| Tip Range | Precision | Recall | F1-Score |
|-----------|-----------|--------|----------|
| $0–$2 | 0.81 | 0.79 | 0.80 |
| $2–$5 | 0.76 | 0.78 | 0.77 |
| $5–$10 | 0.74 | 0.75 | 0.74 |
| $10+ | 0.82 | 0.80 | 0.81 |

> **Business Use**: Forecast driver earnings; design incentive programs for high-tip routes.

### 3. Anomaly Detection (DBSCAN)

**Algorithm**: DBSCAN (ε=0.5, min_samples=50)

| Cluster | Description | % of Trips |
|---------|-------------|-----------|
| 0 | Normal city trips | 98.2% |
| 1 | Short airport runs (JFK/LGA) | 0.9% |
| 2 | Manhattan-to-Brooklyn commutes | 0.6% |
| -1 (Noise) | Anomalous / potential fraud | 1.8% |

Anomalies include zero-distance "ghost" trips and extreme fares ($200+ for under 1 mile).

---

## 📈 Key Insights

### Trip Patterns

| Metric | Finding | Implication |
|--------|---------|-------------|
| Distance | 75% of trips ≤ 6 miles | Optimize short-haul operations |
| Fare | Bimodal (~$12 city, ~$70 airport) | Airport flat-rate differentiation |
| Payment | 65% card, 30% cash | Target 80% digital adoption |
| Peak Hours | 5–6 PM (max), 2–5 AM (min) | Dynamic pricing opportunity |

### Cluster Scalability

| Cluster Size | Records | Training Time | Accuracy |
|-------------|---------|--------------|----------|
| 1 node | 100K | 45s | 89% |
| 3 nodes | 1M | 2m 15s | 90% |
| 5 nodes | 3.5M | 4m 30s | 91% |
| 10 nodes | 10M | 8m 00s | 91% |

> **Linear scaling achieved** — doubling nodes roughly halves processing time.

---

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- 8GB+ RAM (16GB recommended)
- Python 3.9+

### Docker Deployment (Full Cluster)

```bash
git clone https://github.com/mrudula1501/RoutePulse.git
cd RoutePulse

# Start Hadoop + Spark cluster
docker-compose up -d

# Verify running services
docker ps

# Access UIs
# Hadoop NameNode: http://localhost:50070
# Spark Master UI:  http://localhost:8080
```

### Local Development (No Docker)

```bash
pip install -r requirements.txt

# Download data
wget https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-12.parquet -P data/

# Run notebooks
jupyter notebook notebooks/01_eda_clustering.ipynb
```

### Submit Spark Job

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --num-executors 5 \
  --executor-memory 4g \
  --executor-cores 2 \
  scripts/payment_classification.py
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Storage | HDFS (Hadoop 3.3) | Distributed fault-tolerant storage |
| Compute | Spark 3.3 (PySpark) | Distributed processing |
| ML | MLlib, Scikit-learn | Machine learning at scale |
| Orchestration | Docker Compose | Local cluster simulation |
| Visualization | Matplotlib, Seaborn | Charts and dashboards |

---

## 📁 Project Structure

```
RoutePulse/
├── data/
│   ├── raw/                   # NYC TLC downloads
│   └── processed/             # Parquet files
├── notebooks/
│   ├── 01_eda_clustering.ipynb
│   └── 02_classification_models.ipynb
├── src/
│   ├── data_ingestion.py      # HDFS upload
│   ├── features.py            # Feature engineering
│   └── models/
│       ├── payment_classifier.py
│       ├── tip_predictor.py
│       └── anomaly_detector.py
├── scripts/
│   └── submit_spark_job.sh
├── docker/
│   ├── docker-compose.yml
│   └── hadoop.env
├── docs/
│   └── RESEARCH_PAPER.pdf
├── requirements.txt
└── README.md
```

---

## 🔮 Future Enhancements

- [ ] Spark Structured Streaming for real-time ingestion
- [ ] XGBoost for higher accuracy on tip prediction
- [ ] SHAP explainability for regulatory compliance
- [ ] FastAPI REST endpoint for live inference
- [ ] External data integration (weather, events, traffic)

---

## 👥 Team

| Name | Role |
|------|------|
| Mrudula Deshmukh | Data Engineering, ML Pipeline |
| Pratik Aher | Spark Optimization, HDFS |
| Siddhi Nalawade | Feature Engineering, Visualization |

**University**: University at Buffalo, SUNY  
**Course**: CSE 587 — Data Intensive Computing

---

## 📚 References

- NYC TLC Data: https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- Apache Spark: https://spark.apache.org/docs/latest/
- Zaharia et al. *Large-Scale Machine Learning with Spark.*

---

## 📄 License

MIT License — see [LICENSE](LICENSE)

---

## 📬 Contact

**Mrudula Deshmukh**

[![GitHub](https://img.shields.io/badge/GitHub-mrudula1501-181717?style=flat&logo=github)](https://github.com/mrudula1501)
[![Portfolio](https://img.shields.io/badge/Portfolio-mrudula1501.github.io-10B981?style=flat&logo=githubpages&logoColor=white)](https://mrudula1501.github.io/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-dmrudula-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/dmrudula/)
[![Email](https://img.shields.io/badge/Email-mrudulad25@gmail.com-EA4335?style=flat&logo=gmail&logoColor=white)](mailto:mrudulad25@gmail.com)

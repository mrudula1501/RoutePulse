# RoutePulse: Scalable Transportation Analytics Pipeline

Distributed big data pipeline analyzing 3.5M+ NYC taxi trips using Hadoop, Spark, and ML for payment-type prediction.

## 🎯 Results

| Task | Algorithm | Result |
|------|-----------|--------|
| Tip Prediction | Naive Bayes | 78% Accuracy |
| Payment Type | Logistic Regression | 91% Accuracy |
| Trip Clustering | DBSCAN | Anomaly Detection |

## 📊 Dataset

- **Source:** NYC Yellow Taxi December 2024
- **Records:** 3,519,975 trips
- **Size:** 1.2 GB
- **Features:** 19 columns (distance, fare, tip, payment_type, etc.)

## 🏗️ Architecture

**Infrastructure:**
- Docker-Compose Hadoop cluster (5 containers)
- HDFS for distributed storage
- Apache Spark for distributed computation
- PySpark-ML for machine learning

**Pipeline:**
1. Data ingestion to HDFS
2. Exploratory data analysis (Spark SQL)
3. Feature engineering & preprocessing
4. Train 3 ML models (Naive Bayes, Logistic Regression, DBSCAN)
5. Evaluate & persist models to HDFS

## 🔬 Models

### Naive Bayes - Tip Prediction
- Predicts tip category (discrete: $0-$2, $2-$5, $5-$10, $10+)
- Test Accuracy: 78%
- Use Case: Forecast driver earnings, design incentive programs

### Logistic Regression - Payment Type Classification
- Predicts payment method (card, cash, no-charge, dispute, unknown)
- Test Accuracy: 91% (Macro F1)
- Use Case: Anticipate cash-handling needs, encourage digital payments

### DBSCAN - Trip Pattern Clustering
- Identifies anomalous rides (zero-distance "ghost" trips)
- Detects trip archetypes (short-haul, airport runs)
- Use Case: Fraud detection, special pricing strategies

## 💻 Tech Stack

- Python 3.x
- Apache Hadoop + HDFS
- Apache Spark + PySpark
- PySpark MLlib
- Docker & Docker Compose
- Jupyter Notebooks

## 📁 Project Structure
```
RoutePulse/
├── README.md (this file)
├── requirements.txt
├── data/README.md
├── notebooks/
│   ├── 01_eda_clustering.ipynb
│   └── 02_classification_models.ipynb
└── docs/
    ├── RESEARCH_PAPER.pdf
    └── PRESENTATION_SCRIPT.pdf
```

## 🚀 Quick Start

### Prerequisites
```bash
pip install -r requirements.txt
```

### Run Notebooks Locally
```bash
jupyter notebook notebooks/01_eda_clustering.ipynb
```

### Run Full Hadoop/Spark Cluster
```bash
docker-compose up -d
# Access Hadoop UI: http://localhost:50070
```

## 📈 Key Insights

- **Trip Distance:** 75% of trips ≤ 6 miles (right-skewed distribution)
- **Fare Distribution:** Bimodal with spike at ~$70 (airport flat rate)
- **Payment Split:** 65% card, 30% cash, 5% other
- **Hourly Patterns:** Peak 5-6 PM (commute), low 2-5 AM
- **Model Scalability:** Linear scaling across Spark cluster

## 🔮 Future Enhancements

- Real-time Spark Structured Streaming
- XGBoost for regression
- SHAP values for explainability
- FastAPI REST endpoint
- Dashboard visualization
- External data integration (weather, traffic)

## 📚 References

- [Apache Hadoop](https://hadoop.apache.org/)
- [Apache Spark](https://spark.apache.org/)
- [PySpark MLlib](https://spark.apache.org/docs/latest/ml-guide.html)
- [NYC TLC Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

---

**Status:** ✅ Complete  
**Authors:** Mrudula Deshmukh, Pratik Aher, Siddhi Nalawade  
**University:** University at Buffalo, SUNY

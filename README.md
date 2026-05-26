# UK Housing Prices: Data Analysis, Visualization & Prediction

An end-to-end Big Data Analytics and Machine Learning project focused on processing, analyzing, and predicting property transaction trends in England and Wales using cloud infrastructure and modern Business Intelligence tools.

---

## 📌 Project Overview
This project processes the complete **Price Paid Data** provided by **His Majesty's (HM) Land Registry**, covering over **27 million records** (approx. 2.8 GB) of full-market value property sales from 1995 onwards. Due to the scale of the dataset, processing and analytics were offloaded to a distributed cloud computing cluster, followed by interactive visual engineering and predictive modeling.

### 👥 Project Team (C-DAC Kharghar)
* **Ranjit Kailasrao More**
* **Prathmesh Shivaji Dalve**
* **Jayraj Shrikant Maghade**
* **Nayan Surendra Dhoble**
* **Sonali Ashok Waman**
* **Shivaraj Hiraman Shelar**
* **Project Guide:** Parag Thakur | **Course Co-ordinator:** Vineeta Singh

---

## 🛠️ Tech Stack & Architecture

* **Cloud Infrastructure (AWS):**
  * **Amazon S3:** Scalable object storage holding the raw 2.8 GB `.csv` data cluster.
  * **Amazon EC2:** Created a `t2.large` Windows Instance to ingest raw registry transactions via AWS CLI.
  * **Amazon EMR (Elastic MapReduce):** Provisioned a Hadoop cluster comprised of 1 Master Node (`m4.xlarge`) and 2 Slave Nodes (`m4.large`).
* **Big Data Frameworks:** Spark SQL, PySpark RDDs/DataFrames, and Apache Hive.
* **Business Intelligence:** Microsoft Power BI Desktop for data modeling and interactive dashboard tracking.
* **Machine Learning Environment:** AWS SageMaker running a Python Jupyter Lab notebook ecosystem.

---

## 📂 Repository Structure
```text
├── notebooks/            # Jupyter Notebooks containing SageMaker EDA & ML models
├── dashboards/           # Power BI dashboard screenshots (.png formats)
└── docs/                 # Detailed project dissertation report (PDF)

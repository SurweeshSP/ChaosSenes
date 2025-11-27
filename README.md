# ChaosSense 🔥  
*A Real-Time AI Engine for Pattern Discovery in System-Level Chaos*

ChaosSense is an experimental backend-only project that listens to **chaotic system signals** (CPU usage, RAM usage, and network latency/ping jitter) and tries to extract **hidden patterns, anomalies, and "emerging order"** using streaming pipelines and machine learning.

This is my **first AWS project**, designed to help me learn:

- AWS streaming (Kinesis)
- Data lakes (S3)
- Basic ETL (Glue-style jobs)
- Simple ML models on time-series (PyTorch/TF or Scikit-learn)
- Serverless APIs (Lambda + API Gateway)

---

## 🎯 Project Goals

- Collect high-frequency **CPU, RAM, and network jitter** from a system.
- Stream that data to AWS using **Kinesis Data Streams**.
- Store it in an **S3-based data lake** for analysis.
- Run **batch jobs** to compute chaos features like:
  - entropy
  - volatility
  - autocorrelation
- Train simple models to:
  - forecast short-term behavior
  - detect anomalies
- Expose **backend APIs** like `/chaos/forecast` and `/chaos/insights`.

No frontend.  
This is 100% **backend + data + ML + AWS**.

---

## 🧱 High-Level Architecture (v1)

**Version 1 focuses on:**

1. **Local collector**
   - Python script that reads:
     - CPU usage (%)
     - RAM usage (%)
     - Network latency (ping a known host)
   - Emits events like:

     ```json
     {
       "timestamp": "2025-01-01T12:00:00Z",
       "host_id": "local-machine-1",
       "cpu_percent": 37.2,
       "ram_percent": 61.4,
       "ping_ms": 23.7
     }
     ```

2. **Streaming to AWS (Kinesis)**
   - Collector sends each event to a **Kinesis Data Stream**.
   - Partition key examples: `"cpu"`, `"ram"`, `"network"` or `"host_id"`.

3. **Data Lake (S3)**
   - Downstream consumer (or Kinesis Firehose later) writes streaming data to S3.
   - Folder layout (example):

     ```
     s3://chaossense-data/
       chaos-raw/year=2025/month=01/day=01/...
     ```

4. **Batch Analysis (Local or Glue-style)**
   - Use a Jupyter notebook to:
     - load data from S3 (or local file during early stages)
     - compute rolling averages, entropy, volatility
     - visualize CPU/RAM/latency time-series

5. **Basic ML (Offline First)**
   - Use Python notebook to:
     - train a simple forecasting model (e.g. LSTM, ARIMA, or even baseline models)
     - train an anomaly detector (e.g. IsolationForest)
   - Save results and notes inside `ml/` folder.

6. **APIs (Later in v1.5 / v2)**
   - Wrap minimal inference logic in **AWS Lambda**.
   - Expose via **API Gateway** as:
     - `GET /chaos/forecast`
     - `GET /chaos/insights`

---

## 🧩 Tech Stack

- **Language:** Python 3.x
- **Local Metrics:** `psutil`, `subprocess` (for ping)
- **Core AWS Services (v1):**
  - Amazon Kinesis Data Streams
  - Amazon S3
- **Optional (later versions):**
  - AWS Glue / Glue ETL scripts
  - Amazon SageMaker (for training)
  - AWS Lambda + API Gateway
  - Terraform or AWS CDK for infrastructure

---

## 📂 Repository Structure

```bash
chaossense/
├─ README.md
├─ requirements.txt              # Python deps for local + Lambda code
├─ config/
│  ├─ config.example.yaml        # Sample config for streams, regions, etc.
│  └─ logging.conf               # Optional logging configuration
├─ collectors/                   # Local/system agents that collect chaos signals
│  ├─ cpu_ram_network_agent.py   # Collect CPU, RAM, ping jitter & send to Kinesis
│  └─ utils_system_metrics.py    # Helper funcs (psutil, ping, etc.)
├─ streaming/                    # Real-time data pipeline components
│  ├─ kinesis_producer/          # Simple producer logic (if separate from collector)
│  │  └─ producer.py
│  └─ kinesis_consumer/          # For local debugging / testing consumers
│     └─ consumer.py
├─ data_lake/                    # S3-related logic and schemas
│  ├─ schemas/
│  │  └─ chaos_event_schema.json # JSON schema for each event
│  └─ examples/
│     └─ sample_events.json      # Example of raw events
├─ batch/                        # Batch/ETL jobs
│  ├─ glue_jobs/
│  │  └─ chaos_feature_etl.py    # Glue-style ETL script (can run locally first)
│  └─ notebooks/
│     └─ chaos_exploration.ipynb # Jupyter: entropy, plots, first analysis
├─ ml/
│  ├─ feature_engineering/
│  │  └─ chaos_features.py       # Functions to compute entropy, etc.
│  ├─ models/
│  │  ├─ forecasting.py          # Simple LSTM / baseline forecasting
│  │  └─ anomaly_detection.py    # IsolationForest / RandomCutForest (offline first)
│  └─ experiments/
│     └─ chaos_signatures.ipynb  # Notebooks for experimenting
├─ api/
│  ├─ lambda/
│  │  ├─ chaos_forecast_handler.py   # Lambda entrypoint for /chaos/forecast
│  │  └─ chaos_insights_handler.py   # Lambda entrypoint for /chaos/insights
│  └─ openapi/
│     └─ chaosense_api.yaml      # Optional: API spec for API Gateway
├─ infra/                        # AWS infrastructure as code (add gradually)
│  ├─ terraform/                 # OR use cdk/ if you prefer CDK
│  │  └─ main.tf                 # Define Kinesis, S3, IAM, Lambda later
│  └─ diagrams/
│     └─ architecture.png        # Exported architecture diagram (optional)
├─ scripts/
│  ├─ setup_venv.sh              # Shell script to create venv and install deps
│  └─ local_run_demo.sh          # Run collector locally & write to file or stdout
└─ docs/
   ├─ architecture.md            # High-level description of architecture
   ├─ milestones.md              # Roadmap v1, v2, v3
   └─ chaos_metrics.md           # Notes on entropy, Lyapunov, etc.


# ![Bordeaux Traffic Jam ](architecture/de_bordeaux_traffic_jam_data.webp "Architecture Picture")

The project consists of creating a data pipeline to study traffic jams in and around Bordeaux. The architecture is based on a **Docker** container composed by :

> - **Kafka** for real-time data streaming,
> - **Snowflake** for storage and transformation,
> - **Power BI/Streamlit** for interactive visualization.
> - **Grafana & Prometheuse** for performance tracking

```mermaid
flowchart TB
    subgraph Sources["Bordeaux Metropole - Traffic"]
        S1["<a href='https://datahub.bordeaux-metropole.fr/explore/dataset/ci_trafi_l/information/'><b>Bordeaux Métropole Open Data</b></a>"]
        S2["<a href='https://opendata.bordeaux-metropole.fr/explore/dataset/pc_capte_ponct_p/api/?disjunctive.insee'><b>Capteurs de trafic routier ponctuel</b></a>"]
        S3["<a href='https://www.data.gouv.fr/fr/datasets/comptage-du-trafic-2023-bordeaux-metropole/'><b>Comptage du trafic 2023</b></a> "]
    end

    subgraph Ingestion["Real-time Ingestion"]
        direction TB
        K1["Kafka Broker 1"]
        K2["Kafka Broker 2"]
        K3["Kafka Broker 3"]
        ZK["Zookeeper"]
    end

    subgraph Storage["Data Storage & Processing"]
        direction TB
        SF["Snowflake"]
        subgraph WH["Warehouses"]
            W1["Raw Data"]
            W2["Transformed Data"]
            W3["Analytics"]
        end
    end

    subgraph Orchestration["Workflow Orchestration"]
        direction TB
        AF["Apache Airflow"]
        subgraph DAGs["DAG Types"]
            D1["Data Load"]
            D2["Transformations"]
            D3["Export"]
        end
    end

    subgraph Visualization["Data Visualization"]
        direction TB
        PBI["Power BI"]
        ST["Streamlit"]
    end

    subgraph Monitoring["System Monitoring"]
        direction LR
        P["Prometheus"]
        G["Grafana"]
        AM["Alert Manager"]
    end

    subgraph Containerization["Docker Environment"]
        direction TB
        DC["Docker Compose"]
        subgraph Networks["Docker Networks"]
            N1["Ingestion Network"]
            N2["Storage Network"]
            N3["Viz Network"]
            N4["Monitoring Network"]
        end
    end

    Sources --> K1 & K2 & K3
    K1 & K2 & K3 <--> ZK
    K1 & K2 & K3 --> SF
    SF --> W1 --> W2 --> W3
    AF --> D1 & D2 & D3
    D1 & D2 & D3 --> SF
    W3 --> PBI & ST
    P --> G
    P --> AM
    DC --> Networks

    %% Component monitoring connections
    K1 & K2 & K3 & SF & AF --> P

```

During this upskilling journey we will :

### 📌 **1. Data sources**

- Real-time data\*\*: Live traffic API (Bordeaux Métropole Open Data).
- Aggregated data\*\*: Historical traffic counts (data.gouv.fr).
- Enriched data\*\*: OpenWeather API (weather), event data (disruptions).

---

### 🖀 **2. data ingestion and storage**: real time

#### **Real time: Kafka + DLT**

- **Kafka**: Captures real-time data from road traffic API.
- Kafka Connect + Delta Live Tables (DLT)\*\*: Continuous loading into Snowflake.
- Snowflake Storage\*\*: Separate tables for raw and transformed data.

#### **Historical and aggregated data: Airflow + DLT** \*\*Airflow DAGs

- Airflow DAGs\*\*: Scheduled execution to retrieve aggregated data.
- DLT\*\*: Initial storage in raw tables before transformation.

---

### 🔧 **3. Transformation and Modeling (DBT)**

- DBT transforms and cleans\*\* raw data.
- Creation of models\*\* for :
  - Traffic flows by zone and time.
  - Traffic/weather/events correlation.
  - Anomaly detection and prediction.

---

### 🚦 **4. Orchestration (Apache Airflow)**

- Triggers Airflow DAGs\*\* to :
  - Check API availability.
  - Launch Kafka/DLT pipelines.
  - Execute DBT transformations.
  - Refresh dashboards.

---

### 📊 **5. Visualization and Analytics**

- **Power BI**: Interactive dashboards with heat maps and trends.
- **Streamlit**: Web application with dynamic filters and simulations.

---

### 🔄 **6. Automation & Deployment**

- GitHub Actions\*\*: CI/CD to validate DBT models and Airflow workflows.
- Automated testing\*\* :
  - DBT tests\*\*: Check for constraints and anomalies.
  - Unit tests\*\* on Airflow DAGs.
  - Kafka monitoring\*\* with Prometheus/Grafana.

---

### 🏧 **Architecture diagram**

📌 **Real Time Flow**  
1⃣ API Bordeaux Métropole → 2⃣ Kafka → 3⃣ Snowflake (DLT) → 4⃣ DBT → 5⃣ Power BI/Streamlit

📌 **Flux Aggregates**  
1⃣ API Data.gouv.fr → 2⃣ Airflow → 3⃣ Snowflake → 4⃣ DBT → 5⃣ Power BI/Streamlit

📌 **Orchestration & Automation**

- Airflow orchestrates everything.
- GitHub Actions provides CI/CD.

---

**✅ Highlights:**

- ✔ Optimal management of real-time and historical data.
- ✔ Scalable architecture.
- ✔ Clear separation between ingestion, storage, transformation and visualization.
- ✔ Robust orchestration and automation.

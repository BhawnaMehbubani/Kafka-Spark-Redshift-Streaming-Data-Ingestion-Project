# **Kafka-Spark-Redshift Streaming Data Pipeline**

## **Table of Contents**
1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Repository Structure](#repository-structure)
4. [Pipeline Components](#pipeline-components)
   - [Kafka](#1-kafka)
   - [Spark Streaming](#2-spark-streaming)
   - [Redshift](#3-redshift)
   - [Docker Compose](#4-docker-compose)
5. [Step-by-Step Setup](#step-by-step-setup)
   - [Prerequisites](#prerequisites)
   - [Installation and Execution](#installation-and-execution)
6. [Detailed Workflow](#detailed-workflow)
7. [Troubleshooting](#troubleshooting)
8. [Future Enhancements](#future-enhancements)
9. [Contributions](#contributions)



## **Overview**
This project is a real-time data pipeline designed for ingesting, processing, and storing telecom call records. It integrates **Apache Kafka**, **Apache Spark Streaming**, and **AWS Redshift** to handle large volumes of streaming data in near real-time. The pipeline is containerized with **Docker Compose**, enabling easy deployment, scalability, and modularity.



## **Architecture**

Below is the architectural pipeline workflow, represented as a data flow diagram:

```
+-------------------+         +--------------------+          +--------------------+         +----------------------+
| Kafka Producer    |         | Kafka Broker       |          | Spark Streaming    |         | Redshift Database    |
| (Python Script)   |         | (Docker Container) |          | (Data Processing)  |         | (Data Storage)       |
+-------------------+         +--------------------+          +--------------------+         +----------------------+
        |                             |                                |                                 |
        |  1. Produces telecom data   |                                |                                 |
        +----------------------------->                                |                                 |
                                      | 2. Handles message delivery    |                                 |
                                      +------------------------------->                                |
                                                                       | 3. Reads data, processes,       |
                                                                       |    and performs transformations |
                                                                       +------------------------------->|
                                                                                                       | 4. Stores enriched data          |
                                                                                                       +----------------------------------+
```

## **Repository Structure**

The repository has the following file structure to organize the pipeline components effectively:

```
Kafka-Spark-Redshift-Streaming-Data-Ingestion-Project/
|
├── docker-compose.yml           # Docker configurations for all services
├── kafka_producer.py            # Generates mock telecom data and sends to Kafka
├── redshift_connect.py          # Script to test Redshift connectivity
├── redshift_create_table.txt    # SQL script to create a table in Redshift
├── redshift-jdbc42-2.1.0.12.jar # Redshift JDBC driver
├── spark_redshift_stream.py     # Spark Streaming application script
└── README.md                    # Project documentation (this file)
```


## **Pipeline Components**

### 1. **Kafka**
Apache Kafka acts as the message broker, managing real-time streams of data between the producer and the consumer.

#### Kafka Components:
- **Producer**: Generates mock telecom data using the `kafka_producer.py` script.
- **Broker**: A Kafka broker is hosted in a Docker container to manage topics and distribute messages.
- **Topic**: The data is streamed to the `telecom-data` topic, created during the pipeline initialization.
- **Schema Registry**: Ensures that the producer’s data conforms to the required schema for quality checks.

#### Example Kafka Data:
```json
{
  "caller_name": "John Doe",
  "receiver_name": "Jane Smith",
  "caller_id": "+123456789",
  "receiver_id": "+987654321",
  "start_datetime": "2025-01-25 10:00:00",
  "end_datetime": "2025-01-25 10:05:00",
  "call_duration": 300,
  "network_provider": "AT&T",
  "total_amount": 0.50
}
```



### 2. **Spark Streaming**
Apache Spark processes data from Kafka in real-time, performs transformations, and writes it to AWS Redshift.

#### Key Features:
- Reads data from the `telecom-data` Kafka topic.
- Validates and filters out invalid records (e.g., negative call durations).
- Writes enriched and transformed data to Redshift.

#### Spark Job Workflow:
1. **Read Stream**: Subscribes to the Kafka topic and parses incoming JSON data.
2. **Transform**:
   - Ensures valid call durations.
   - Adds metadata like processing timestamps.
3. **Write**: Uses the JDBC driver to insert data into the Redshift `telecom_data` table.



### 3. **Redshift**
AWS Redshift acts as the persistent data store for processed telecom records, allowing for further analytics.

#### Table Schema:
The Redshift table is created using the `redshift_create_table.txt` script with the following schema:

```sql
CREATE TABLE telecom.telecom_data (
  caller_name VARCHAR(50),
  receiver_name VARCHAR(50),
  caller_id VARCHAR(20),
  receiver_id VARCHAR(20),
  start_datetime TIMESTAMP,
  end_datetime TIMESTAMP,
  call_duration INTEGER,
  network_provider VARCHAR(30),
  total_amount DECIMAL(10, 2)
);
```

#### Example Query:
```sql
SELECT * FROM telecom.telecom_data LIMIT 10;
```



### 4. **Docker Compose**
The `docker-compose.yml` file orchestrates the deployment of the entire pipeline, including Kafka, Spark, and auxiliary services.

#### Key Services:
- **Kafka Broker**: Hosts the `telecom-data` topic.
- **Zookeeper**: Coordinates Kafka brokers.
- **Schema Registry**: Ensures data schema validation.
- **Spark Master & Worker**: Executes Spark jobs for real-time processing.



## **Step-by-Step Setup**

### **Prerequisites**
- Docker and Docker Compose installed.
- Python 3.8+ installed with the following dependencies:
  ```bash
  pip install kafka-python faker psycopg2
  ```
- AWS Redshift cluster configured with proper credentials and network access.

### **Installation and Execution**
1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd project-directory
   ```

2. **Start Docker Services**:
   ```bash
   docker-compose up -d
   ```

3. **Run the Kafka Producer**:
   ```bash
   python kafka_producer.py
   ```

4. **Submit the Spark Job**:
   ```bash
   spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.1.1,io.github.spark-redshift-community:spark-redshift_2.12:6.2.0-spark_3.5 spark_redshift_stream.py
   ```

5. **Verify Data in Redshift**:
   - Connect to your Redshift cluster using a SQL client or `psql`.
   - Run the following query:
     ```sql
     SELECT * FROM telecom.telecom_data LIMIT 10;
     ```



## **Detailed Workflow**

1. **Data Generation**:
   - The `kafka_producer.py` script generates telecom data with realistic fields using the `Faker` library.
   - Data is pushed to the Kafka topic `telecom-data`.

2. **Kafka Message Broker**:
   - Manages the data stream and ensures delivery to consumers (Spark Streaming).

3. **Real-Time Processing**:
   - Spark Streaming reads data from the Kafka topic.
   - Invalid records (e.g., negative durations) are filtered out.
   - Valid data is enriched and transformed.

4. **Data Storage**:
   - Processed data is written to the `telecom.telecom_data` table in Redshift using the JDBC driver.



## **Future Enhancements**
- Add monitoring tools like **Prometheus** and **Grafana**.
- Automate schema validation with **AWS Glue**.
- Enable fault tolerance with Kafka partition replication.



## **Contributions**

Contributions are welcome! To contribute to this project:

1. Fork the repository.
2. Create a feature branch (`git checkout -b feature-name`).
3. Commit your changes (`git commit -m 'Add new feature'`).
4. Push the branch (`git push origin feature-name`).
5. Open a pull request for review.



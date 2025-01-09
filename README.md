# Smart Home Energy Consumption Analysis 

## Objective

This project showcases the development of a scalable data pipeline capable of processing streaming data by combining Kafka, Apache Flink, and SQL for real-time analytics. The key objectives are to:

1. Generate synthetic datasets using Avro schema definitions.
2. Stream data into Kafka topics for further processing.
3. Leverage Apache Flink and SQL for real-time data analysis.
4. Create enriched data views to derive actionable insights.
5. Set the foundation for future data visualization with Elasticsearch and Kibana.

## Dataset Description

The dataset provides synthetic data on smart home energy usage, detailing electricity consumption, appliance usage, temperature settings, and occupancy status. It is designed to help analyze energy consumption patterns, identify energy-saving opportunities, optimize energy usage, and develop smart home automation solutions. The dataset can be used to identify trends in energy consumption, forecast energy demand, enhance the efficiency of smart home systems, and support the development of automated energy-saving solutions.

Columns:

1. **Datetime:** Date and time of the recorded data.
2. **home_id:** Unique identifier for each home.
3. **energy_consumption_kWh:** Energy consumption in kilowatt-hours.
4. **temperature_setting_C:** Temperature setting in degrees Celsius.
5. **occupancy_status:** Whether the home is occupied or unoccupied.
6. **appliance:** Type of appliance in use (e.g., HVAC, Washing Machine, Dishwasher).
7. **usage_duration_minutes:** Duration of appliance usage in minutes.
8. **season:** Season during the recording (e.g., Winter, Spring, Summer, Autumn).
9. **day_of_week:** Day of the week (e.g., Monday, Tuesday).
10. **holiday:** Indicator of whether the day is a holiday (1 if holiday, 0 otherwise).
    

<img width="650" alt="Screenshot 2025-01-10 at 3 11 22 AM" src="https://github.com/user-attachments/assets/4d5d20b2-c324-4b32-b71d-d1fd8530cd74" />

## Data Pipeline Process

**1. Environment Setup**
   
- Apache Flink, Kafka, and Elasticsearch were configured in the environment for data streaming, processing, and storage.
- Required shell scripts and Python scripts were prepared for automation:
  
a. `start_flink_nodatagen.sh`: To start Apache Flink without data generation.

b. `convert.py`: To transform data into a Kafka-compatible format.

c. `gen_sample.sh`: To generate and stream data to Kafka topics.

d. `consumer.sh`: To verify and monitor Kafka topic data consumption.
   
**2. Data Preparation**

- **Source Data:** A synthetic energy consumption dataset in JSON format, `rev_energy_data.json`, was prepared.
- **Transformation:** The JSON file was converted into key-value pairs using `convert.py` for Kafka ingestion.
- **Commands Executed:**
  
       python $HOME/Documents/fake/convert.py
  
       chmod +x *.sh
  
**3. Data Ingestion into Kafka**

- Data was streamed into a Kafka topic named energy_data using `gen_sample.sh`.
- Each record contained details such as energy consumption, appliance usage, and contextual attributes like season and occupancy status.
- **Command Executed:**
  
      ./gen_sample.sh /home/ashok/Documents/gendata/rev_energy_data.json 500 100 | kafkacat -b localhost:9092 -t energy_data -K: -P
  
**4. Real-Time Stream Processing with Apache Flink**
   
- Apache Flink consumed data from the `energy_data` Kafka topic for real-time processing and transformation.
- A Flink SQL table `energy_data` was created to structure the data for downstream usage:
  
      CREATE TABLE energy_data (
      id BIGINT PRIMARY KEY,
      datetime TIMESTAMP,
      home_id INT,
      energy_consumption_kWh FLOAT,
      temperature_setting_C FLOAT,
      occupancy_status STRING,
      appliance STRING,
      usage_duration_minutes INT,
      season STRING,
      day_of_week STRING,
      holiday INT,
      WATERMARK FOR datetime AS datetime - INTERVAL '5' SECOND
      ) WITH (
      'connector' = 'kafka',
      'topic' = 'energy_data',
      'scan.startup.mode' = 'earliest-offset',
      'properties.bootstrap.servers' = 'kafka:9094',
      'format' = 'json',
      'json.timestamp-format.standard' = 'ISO-8601'
       );

**5. Data Storage in Elasticsearch**
- Processed data was stored in an Elasticsearch index `energy_index` for efficient querying and visualization.
- A Flink SQL table `energy_index` was created to map the processed data into Elasticsearch:
  
       CREATE TABLE energy_index (
       id BIGINT PRIMARY KEY,
       datetime TIMESTAMP,
       home_id INT,
       energy_consumption_kWh FLOAT,
       temperature_setting_C FLOAT,
       occupancy_status STRING,
       appliance STRING,
       usage_duration_minutes INT,
       season STRING,
       day_of_week STRING,
       holiday INT
       ) WITH (
       'connector' = 'elasticsearch-7',
       'hosts' = 'http://elasticsearch:9200',
       'index' = 'energy_index',
       'format' = 'json',
       'json.fail-on-missing-field' = 'false',
       'json.ignore-parse-errors' = 'true',
       'json.timestamp-format.standard' = 'ISO-8601'
        );

- Data was inserted into energy_index:
  
        INSERT INTO energy_index
        SELECT *
        FROM energy_data;

**6. Data Visualization with Kibana**
   
- Kibana Integration: Elasticsearch was integrated with Kibana to enable real-time data visualization.
- A dashboard was created on `localhost:5061` with visualizations for:
  
**a. Energy Trends:** Energy consumption trends by appliances and time.
**b. Occupancy Impact:** How occupancy affects energy usage.
**c. Seasonal Analysis:** Patterns across different seasons.
**d. Efficiency Metrics:** Appliance efficiency and usage duration analysis.

**7. Validation and Monitoring**
- Kafka consumer script `consumer.sh` was executed to monitor and validate data flowing through Kafka topics:
  
      ./consumer.sh energy_data

- The Flink SQL interface was accessed to confirm correct data processing.

**8. Key Highlights**

a. Real-Time Data Pipeline: Built a robust pipeline using Kafka, Flink, Elasticsearch, and Kibana.

b. Automated Data Streaming: Used shell scripts and Python for seamless automation.

c. Actionable Insights: Enabled real-time visualization of energy consumption patterns to drive meaningful insights.

# Dashboard Analysis

![image](https://github.com/user-attachments/assets/4d98c73e-5ac3-4705-bad8-975fd1aea0ba)

## video

https://github.com/user-attachments/assets/9155bbd7-1697-4c9c-8618-6036ce979596












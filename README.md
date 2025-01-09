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
    

<img width="850" alt="Screenshot 2025-01-10 at 3 11 22 AM" src="https://github.com/user-attachments/assets/4d5d20b2-c324-4b32-b71d-d1fd8530cd74" />

## Data Pipeline Process

1. Data Generation

Synthetic datasets for energy consumption were generated using Avro schemas and the gendata.sh script:

         ./gendata.sh energy_consumption.avro energy_data.json 10000

2. Data Transformation

The generated JSON file was transformed into Kafka-compatible formats using a Python script:

python $HOME/Documents/convert.py

3. Kafka Ingestion

The transformed data was streamed into Kafka topics using the gen_sample.sh script:

./gen_sample.sh /home/user/Documents/gendata/rev_energy_data.json | kafkacat -b localhost:9092 -t energy_data -K: -P
4. Real-Time Analysis with Apache Flink

Flink SQL was used to create real-time analytics tables for the dataset.

Energy Consumption Table (energy_data)
CREATE TABLE energy_data (
    datetime TIMESTAMP(3),
    home_id BIGINT,
    energy_consumption_kWh DOUBLE,
    temperature_setting_C DOUBLE,
    occupancy_status STRING,
    appliance STRING,
    usage_duration_minutes INT,
    season STRING,
    day_of_week STRING,
    holiday BOOLEAN
) WITH (
    'connector' = 'kafka',
    'topic' = 'energy_data',
    'scan.startup.mode' = 'earliest-offset',
    'properties.bootstrap.servers' = 'kafka:9094',
    'format' = 'json',
    'json.timestamp-format.standard' = 'ISO-8601'
);
5. Data Enrichment

A consolidated view was created to enrich the energy consumption data with derived metrics, such as energy usage per minute and appliance efficiency:

CREATE VIEW enriched_energy_data AS
SELECT 
    datetime,
    home_id,
    energy_consumption_kWh,
    temperature_setting_C,
    occupancy_status,
    appliance,
    usage_duration_minutes,
    energy_consumption_kWh / usage_duration_minutes AS energy_usage_per_minute,
    CASE
        WHEN occupancy_status = 'Occupied' THEN 'High'
        ELSE 'Low'
    END AS usage_priority,
    season,
    day_of_week,
    holiday
FROM energy_data;
6. Dashboard Creation

The data in the enriched_energy_dashboard index was visualized using Kibana.

An index pattern for enriched_energy_dashboard was created in Kibana.
A comprehensive dashboard was designed to showcase:
Energy Consumption Trends: Consumption by appliance and time.
Occupancy Insights: Impact of occupancy status on energy usage.
Seasonal Effects: Seasonal variations in energy usage.
Efficiency Metrics: Energy usage per minute and appliance-specific insights.

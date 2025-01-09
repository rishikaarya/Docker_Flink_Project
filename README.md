# Smart Home Energy Consumption Analysis 

## Objective

This project showcases the development of a scalable data pipeline capable of processing streaming data by combining Kafka, Apache Flink, and SQL for real-time analytics. The key objectives are to:

1. Generate synthetic datasets using Avro schema definitions.
2. Stream data into Kafka topics for further processing.
3. Leverage Apache Flink and SQL for real-time data analysis.
4. Create enriched data views to derive actionable insights.
5. Set the foundation for future data visualization with Elasticsearch and Kibana.

## Dataset Description

The dataset provides synthetic data on smart home energy usage, detailing electricity consumption, appliance usage, temperature settings, and occupancy status. It is designed to help analyze energy consumption patterns, identify energy-saving opportunities, optimize energy usage, and develop smart home automation solutions.

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
    
The dataset can be used to identify trends in energy consumption, forecast energy demand, enhance the efficiency of smart home systems, and support the development of automated energy-saving solutions.

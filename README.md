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

### 1. Environment Setup
   
- Apache Flink, Kafka, and Elasticsearch were configured in the environment for data streaming, processing, and storage.
- Required shell scripts and Python scripts were prepared for automation:
  
a. `start_flink_nodatagen.sh`: To start Apache Flink without data generation.

b. `convert.py`: To transform data into a Kafka-compatible format.

c. `gen_sample.sh`: To generate and stream data to Kafka topics.

d. `consumer.sh`: To verify and monitor Kafka topic data consumption.
   
 ### 2. Data Preparation

- **Source Data:** A synthetic energy consumption dataset in JSON format, `rev_energy_data.json`, was prepared.
- **Transformation:** The JSON file was converted into key-value pairs using `convert.py` for Kafka ingestion.
- **Commands Executed:**
  
       python $HOME/Documents/fake/convert.py
  
       chmod +x *.sh
  
### 3. Data Ingestion into Kafka

- Data was streamed into a Kafka topic named energy_data using `gen_sample.sh`.
- Each record contained details such as energy consumption, appliance usage, and contextual attributes like season and occupancy status.
- **Command Executed:**
  
      ./gen_sample.sh /home/ashok/Documents/gendata/rev_energy_data.json 500 100 | kafkacat -b localhost:9092 -t energy_data -K: -P
  
### 4. Real-Time Stream Processing with Apache Flink
   
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

### 5. Data Storage in Elasticsearch
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

### 6. Data Visualization with Kibana
   
- Kibana Integration: Elasticsearch was integrated with Kibana to enable real-time data visualization.
- A dashboard was created on `localhost:5061` with visualizations for:
  
**a. Energy Trends:** Energy consumption trends by appliances and time.

**b. Occupancy Impact:** How occupancy affects energy usage.

**c. Seasonal Analysis:** Patterns across different seasons.

**d. Efficiency Metrics:** Appliance efficiency and usage duration analysis.

### 7. Validation and Monitoring
- Kafka consumer script `consumer.sh` was executed to monitor and validate data flowing through Kafka topics:
  
      ./consumer.sh energy_data

- The Flink SQL interface was accessed to confirm correct data processing.

### 8. Key Highlights

**a. Real-Time Data Pipeline:** Built a robust pipeline using Kafka, Flink, Elasticsearch, and Kibana.

**b. Automated Data Streaming:** Used shell scripts and Python for seamless automation.

**c. Actionable Insights:** Enabled real-time visualization of energy consumption patterns to drive meaningful insights.


# Dashboard Analysis

![image](https://github.com/user-attachments/assets/d38767ea-8d7c-40a9-91eb-004f2e37f762)

### Video

https://github.com/user-attachments/assets/9155bbd7-1697-4c9c-8618-6036ce979596

### Charts 

1. How does the average energy consumption differ by occupancy status? 

**Average Energy Consumption by Occupancy Status**

![image](https://github.com/user-attachments/assets/c09c5608-e359-4bab-b153-4aa59b26034f)

**Observations:** 
The chart shows that the average energy consumption during "Occupied" and "Unoccupied" periods is comparable, with only a slight difference.

**Insights:**
The small difference in energy consumption suggests that human presence has minimal impact on energy usage, potentially indicating reliance on devices or systems that operate regardless of occupancy.

2. How does energy consumption vary between different types of appliances used?

**Energy Consumption by Each Appliance**

![image](https://github.com/user-attachments/assets/d99a548a-f23c-41bc-84a0-76319ffd3b8f)

**Observations:** 
The chart illustrates the average energy consumption for various appliances, showing that dishwashers and washing machines are the most energy-intensive, while lighting consumes the least energy.

**Insights:**
Dishwashers and washing machines should be the primary focus for energy efficiency measures. Potential interventions could include promoting energy-efficient models or optimizing their usage patterns to achieve better energy management.

3. How does the average energy consumption trend vary across the days in January?

**Energy Consumption Trend over the Month** 

![image](https://github.com/user-attachments/assets/fa100c38-2008-4c9b-88f9-0c88f929cfcd)

**Observations:** 
The trend indicates a steady increase in average energy consumption over the month. This could be due to cumulative usage patterns, increasing reliance on heating or appliances as the month progresses. 

**Insights:**
January is typically associated with winter in many regions. The increasing trend reflects growing energy demands during colder days for the use of ppliances like heaters or HVAC systems for longer durations as the month progresses, contributing to higher consumption..

4. How does the energy usage duration differ between holidays and non-holidays across the days of the week?

**Weekly Energy Usage Duration(mins): Holidays vs Non - Holidays**

![image](https://github.com/user-attachments/assets/8469581e-42c9-45e1-903b-c68996122fc3)

**Observations:** 
It shows that energy usage on holidays is evenly distributed across the week, with no significant peaks or dips. On non-holidays, energy usage is relatively consistent but shows a slight increase on Thursdays. The highest energy usage duration (72–90 minutes) is predominantly observed during weekends and holidays, indicating increased usage during these times. 

**Insights:**
The higher energy usage during weekends and holidays suggests that people spend more time at home, likely engaging in activities requiring greater appliance usage, such as cooking or entertainment.

5. How does energy consumption vary across different seasons?

**Energy Consumption By Season**

![image](https://github.com/user-attachments/assets/3d5bd22d-f4af-4874-9060-fb9cb66df0b4)

**Observations:** 
The distribution is relatively even, with Spring showing the highest energy consumption and Winter the lowest. The differences between seasons are marginal, with a variation of less than 3%.

**Insights:**
The higher energy consumption in Spring and Summer could be attributed to the increased use of cooling appliances like air conditioners. In contrast, Winter shows the lowest consumption, possibly due to more efficient heating systems or reduced reliance on appliances compared to cooling needs in warmer seasons.

6. How does the temperature setting affect average energy consumption?

**Effect of Temperature on Energy Consumption**

![image](https://github.com/user-attachments/assets/b9bcd7d1-00f4-49d7-a0d8-8ca20fec28d3)

**Observations:** 
The chart demonstrates a clear upward trend in average energy consumption as the temperature setting increases. At lower temperature settings, energy consumption is minimal but rises steadily with higher temperature settings, peaking at around 22.7°C and above. It shows that as the temperature setting increases, energy consumption also increases and vice varsa.  

**Insights:**
It indicates that energy consumption increases as the temperature setting rises and decreases as the temperature setting lowers. This relationship suggests that higher energy usage is primarily influenced by heating or cooling systems working to maintain the set temperature.

7.How does seasonal energy consumption vary across different appliance types?

**Seasonal Energy consumption by Appliance Type** 

![image](https://github.com/user-attachments/assets/89c03f94-e808-4aa7-89c1-42545f8456a6)


**Observations:** 
The heatmap illustrates seasonal variations in energy consumption for various appliances. Appliances such as HVAC and refrigerators show consistently higher energy usage, particularly in summer and winter, likely due to increased cooling and heating demands. In contrast, appliances like dishwashers and washing machines exhibit relatively stable energy consumption across seasons, with minimal seasonal variation.

**Insights:**
The high energy consumption of HVAC systems and refrigerators during extreme weather conditions suggests that these appliances are the primary drivers of seasonal energy spikes. Energy-saving measures could include optimizing HVAC usage with programmable thermostats and using energy-efficient refrigerators.

8. Which appliances contribute the most to energy consumption in the top 5 homes with maximum energy usage?
   
**Top 5 homes with Max Energy Consumption by Appliance Type**

![image](https://github.com/user-attachments/assets/4897b477-31e6-4aa4-864e-fb95622652ef)

**Observations:** 
The bar chart shows that HVAC systems and refrigerators are the largest contributors to energy consumption in most of these homes. Homes 56 and 86 exhibit the highest overall energy usage, with HVAC accounting for a significant share in both. Other appliances like lighting, washing machines, and electronics contribute comparatively less but vary in significance across homes.

**Insights:**
The dominance of HVAC and refrigerators in energy consumption highlights their role in driving high energy usage in these homes. This suggests that targeted strategies, such as energy-efficient HVAC systems, better insulation to reduce HVAC workload, and upgrading refrigerators to more efficient models, could significantly reduce energy usage in these households. 

9. Which days of the week have the highest energy consumption?
    
**Week Days with Higher Energy Consumption**

![image](https://github.com/user-attachments/assets/bd236ed0-7b3f-40f4-9dae-f24a526abb60)

**Observations:** 
The chart reveals that Saturday, Friday, and Sunday stand out as the days with the highest energy consumption. In contrast, midweek days, such as Wednesday, show significantly lower energy usage. This suggests a notable difference in consumption patterns between weekends and weekdays, likely influenced by lifestyle or activity changes.

**Insights:**
Higher energy consumption on weekends and near-weekend days may be due to increased household activities such as laundry, cleaning, and cooking, as more people stay at home during these times. On weekdays, particularly midweek, energy usage appears to be lower, possibly because people spend more time at work or outside the home.

10. Which homes have the longest appliance usage durations?

**Top 10 homes with Max usage duration(mins)**

![image](https://github.com/user-attachments/assets/f221b2a4-9d63-4b68-b226-49512b031635)

**Observations:** 
The table lists the top 10 homes with the highest appliance usage durations. Home IDs 8, 11, 19, 53, 87, and 94 share the longest recorded duration of 119 minutes, while the remaining homes (20, 29, 40, and 76) have slightly lower durations of 118 minutes. 

**Insights:**
The high appliance usage durations in these homes suggest heavy reliance on specific energy-intensive appliances or consistent usage patterns. These durations might be linked to different activities carried throughout the day. 

# Managerial Insights 

**1. Occupancy and Automation:**
Energy usage shows minimal variation between occupied and unoccupied periods, indicating a reliance on always-on or automated systems. Efforts should focus on optimizing these systems for efficiency rather than solely addressing user behavior.

**2. Appliance Energy Optimization:**
Dishwashers, washing machines, and HVAC systems are the most energy-intensive appliances. Managers should prioritize promoting energy-efficient models and educating homeowners about optimal usage practices, such as scheduling appliances during off-peak hours.

**3. Seasonal Consumption Trends:**
HVAC systems drive higher energy usage in summer and winter, while spring exhibits the highest overall consumption, potentially due to transitional weather demands. Insulation improvements and smart thermostats can help reduce seasonal spikes.

**4. Behavioral Patterns and Weekends:**
Energy consumption is highest during weekends, Fridays, and holidays, reflecting increased household activities. Managers can implement time-of-use pricing or energy-saving campaigns targeting these high-demand periods to manage consumption more effectively.

**5. Temperature-Driven Usage:**
Energy consumption rises with increased temperature settings, emphasizing the need for smart thermostats and homeowner education on maintaining efficient temperature settings.

**6. Energy Usage in High-Consumption Homes:**
A small subset of homes exhibits consistently long appliance usage durations and higher energy consumption, driven by HVAC and refrigerators. Personalized energy audits or targeted interventions in these homes can yield significant savings.

**7. Seasonal and Appliance Insights:**
HVAC and refrigerators are the primary drivers of energy consumption across all seasons. Managers should consider rebate programs for energy-efficient HVAC systems and refrigerators to reduce long-term energy costs.

# Key Learnings

**1. Understanding Data Streaming with Apache Flink:**

- Gained hands-on experience with Apache Flink, a powerful framework for real-time data processing.
- Learned how to efficiently handle streaming data and implement transformations to derive meaningful insights.

**2. Using Flink SQL for Data Analysis:**

- Explored the capabilities of Flink SQL for querying and processing streaming data in real-time.
- Understood the importance of Flink SQL's ability to support complex aggregations, filtering, and joins, making it an efficient tool for data manipulation.
  
**3. Data Visualization with Kibana:**

- Developed skills in creating interactive dashboards using Kibana to visualize streaming data.
- Learned how to use various visualizations like bar charts, pie charts, and timelines to interpret key metrics such as churn propensity, income distribution, and customer demographics.
  
**4. Customer Insights and Churn Analysis:**

- Analyzed customer behavior based on activity types and demographic factors to identify patterns and trends.
- Understood how churn propensity varies with activity type, enabling data-driven decision-making.

**5. Real-Time Data Processing:**

- Developed an appreciation for real-time data analytics and its role in deriving actionable insights for businesses.
- Understood the challenges and solutions associated with managing and analyzing streaming data.

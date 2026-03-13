## Predictive Maintenance

### Test Scenarios
- Anomaly Detection
    - tracking of machine patterns, e.g. power pattern, temp. pattern | "NORMAL" STATE
    - formulating reference/ threshold
    - simulating an error | "ERROR" STATE
    - generating indicator, function to indicate error | "INDICATOR"

### Decription on example "anomaly detection"
#### 1. Load and Prepare Data
- The dataset is read from a CSV file (in operation fetched from (time-series database), and a set of numerical features is selected for anomaly detection (temperature, power, current, etc.).

> In pattern recognition, features refer to individual measurable properties or characteristics of the data that are used to represent it during analysis, aiding in the identification and classification of patterns or anomalies.

#### 2. Scale the Features
- The features are standardized using ```StandardScaler``` so they all have comparable scale. This improves model stability and performance.

#### 3. Train Isolation Forest
- An Isolation Forest model is fitted on the scaled features. The model assigns each row either:

- ```1``` → normal

- ```-1``` → anomaly

#### 4. Compute Z-scores for Each Feature
- The code also computes per-feature z-scores (how many standard deviations away from the mean each feature value is).

#### 5. Identify Which Features Caused the Anomaly
- For rows detected as anomalous, the script checks which individual features exceed a threshold (±3 standard deviations). These features are stored in a new column (```anomaly_features```).

#### 6. Output
- The resulting DataFrame allows you to see:
  - which rows are anomalous
  - which specific sensor readings likely caused the anomaly

<!-- ![](https://pad.fabcity.hamburg/uploads/e6c1ebc2-ca29-4aaa-8ea0-fc4a1286b69f.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/e6c1ebc2-ca29-4aaa-8ea0-fc4a1286b69f.png" width="100%"><br>
</p>
<!-- ![](https://pad.fabcity.hamburg/uploads/37ed378b-0130-427c-aa54-6a91ad29b262.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/37ed378b-0130-427c-aa54-6a91ad29b262.png" width="100%"><br>
</p>


#### How isolation mdoel works
- It works by making many random cuts in the data. Each cut splits the data into smaller parts.
- If a point is unusual, it tends to get separated very quickly because it doesn’t fit well with the others.
- If a point is normal, it takes many cuts to isolate it because it blends in with a large group.
- After doing this many times, the model checks:
- Points isolated quickly → likely anomalies
- Points that take many steps to isolate → likely normal

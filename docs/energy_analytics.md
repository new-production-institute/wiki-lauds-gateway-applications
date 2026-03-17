## Energy Analytics

The Energy Analytics module analyzes the relationship between machine operational parameters and electrical energy measurements. By correlating machine data with electrical parameters such as **active power (apower)**, **current**, and **voltage**, users can identify how different machine activities influence energy consumption.

The results help users understand machine energy behavior and identify parameters that have the strongest impact on power usage.

## Energy Parameter Correlation

The **Energy Parameter Correlation** feature allows users to configure and run correlation analysis between selected machine and environmental parameters and electrical energy measurements.

Users can:
- Select a **machine**
- Choose **machine parameters** (e.g., axis positions, fan status, temperatures)
- Select **electrical parameters** (**apower**, **current**, **voltage**)
- Define a **time range** for the analysis

After clicking **Analyze**, the system calculates correlations and displays them in a table.

Results are color-coded for easy interpretation:
- **Green** indicates a **positive correlation**
- **Red** indicates a **negative correlation**
- **Darker colors** represent **stronger correlations**, while lighter colors indicate weaker relationships.

Correlation values range from **-1 to +1**, where values closer to the extremes indicate stronger relationships between parameters.

You can visualise the corelation dashbord from the frontend that is running in Docker using a web server and is exposed on port 8088: http://localhost:8088/
![corealtion_dashboard](<media/corealtion_dashboard.jpeg>)

## Machine State Prediction

The **Machine State Prediction** feature enables users to determine a machine's runtime state using a connected energy sensor. This means that there is no need for a communication interface with the machine in order to obtain information about its state.

The state prediction needs to be configured individually for each machine, as each machine has a different power pattern from which the state is derived. 
The following steps describe the process using an Epilog laser at Fabelhaft St. Pauli in Hamburg.

### Data Preparation and Model Training

#### 1. Extract Data from InfluxDB
- Go to the **Data Explorer** window.
- Select the relevant data fields: `aenergy`, `apower`, `current`, `voltage`.
- Use the **Download Results as .csv** option.

![Data Explorer Screenshot](https://pad.fabcity.hamburg/uploads/04a0e9cf-5531-495f-9d65-36a42e27c1ae.png)

---

#### 2. Restructure the Data
- Format the extracted data into the required structured format.

![Restructured Data](https://pad.fabcity.hamburg/uploads/96ce5c98-81bd-4e4f-8950-0ca2d12404f4.png)

---

#### 3. Assign Machine States
- Load the data.
- Assign states using:
  - Threshold-based logic **or**
  - Manual labeling for each entry.

![State Assignment Example 1](https://pad.fabcity.hamburg/uploads/fac7d622-c5e0-456b-99fc-3cc87e221a5a.png)

![State Assignment Example 2](https://pad.fabcity.hamburg/uploads/a5123e3a-52b3-4c12-806e-3d41bab91ef4.png)

![State Assignment Example 3](https://pad.fabcity.hamburg/uploads/c7b6f0e2-a7cc-49e2-ae4c-f67496f93183.png)

---

#### 4. Clean Data and Split Features/Labels
- Drop unnecessary columns.
- Split the dataset into:
  - **X** (features)
  - **Y** (labels)

![Feature Label Split](https://pad.fabcity.hamburg/uploads/ef3d962e-d3af-49b0-8f62-7b90a9159298.png)

---

#### 5. Encode Labels
- Convert categorical labels into integer values.

![Label Encoding](https://pad.fabcity.hamburg/uploads/4bf30172-38ef-4f63-8357-573782cc019a.png)

---

#### 6. Declare and Train the Model
- Define the model.
- Train it using the prepared dataset.

![Model Training](https://pad.fabcity.hamburg/uploads/d1bd7f9d-65ca-45bd-bbda-216d381463ae.png)

---

#### 7. Evaluate Model Performance
- Test the model and review performance metrics.

![Model Evaluation 1](https://pad.fabcity.hamburg/uploads/fd1e5b02-e54a-4605-b430-f23d234f948c.png)

![Model Evaluation 2](https://pad.fabcity.hamburg/uploads/f20efe1f-7fc1-4daa-9ff4-448f167db5a4.png)

---

#### 8. Save the Model and Integrate with Node-RED
- Save the trained model.
- Proceed with **Node-RED integration**.

![Model Saving](https://pad.fabcity.hamburg/uploads/fcf65f53-318a-4143-952c-7b988319d1b5.png)


## Node-Red Adaption - Laser Cutter State & Job Tracking
1. create a folder models inside the lauds gateway software/flow with the model files
2. add this in the docker file in ```/software/flow``` at the end and rebuid the container

```USER root
RUN apk add --no-cache \
python3 \
py3-pip \
python3-dev \
build-base \
pkgconf \
openblas-dev \
&& pip3 install --no-cache-dir --upgrade pip --break-system-packages \
&& pip3 install --no-cache-dir joblib numpy scikit-learn pandas --break-system-packages \
&& apk del build-base python3-dev pkgconf
USER node-red
```

3. copy the model attached into the container volume
 ```docker cp /path/to/local/file container ID:/data/models/```
4. install pip packages like this inside the container
```pip3 install numpy –break-system-packages```
5. import the flow and test with real time data.

### Node Red flow
![](https://pad.fabcity.hamburg/uploads/9f6e2381-ae66-4e56-bf7c-66d3a1f9a9d5.png)

- This Node-RED flow uses two function nodes to predict laser cutter states and track jobs automatically.

    - State Prediction Node

        - Extracts power, current, voltage, and energy from incoming payloads.
        - Loads a pre-trained decision tree to predict the machine state.
        - Adds a human-readable Predicted_state to the payload.

![](https://pad.fabcity.hamburg/uploads/56b8a2cf-92ba-40ef-b25e-44d12e033e7e.png)

- Job Tracking Node
  - Uses Predicted_state to detect job start/stop.
  - Tracks job duration and energy consumption.
  - Outputs total_time_sec and total_energy when a job finishes; otherwise, forwards the original payload.

    ![](https://pad.fabcity.hamburg/uploads/97c5f250-651d-4818-9be6-bfbe8cbe5736.png)
    ![](https://pad.fabcity.hamburg/uploads/7e2b593d-08e6-48a1-8489-0e49bb4ccfb4.png)



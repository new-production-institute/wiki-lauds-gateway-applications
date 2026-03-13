## Setting up Hardware
<!-- needs to be added
- energy sensor shelly power plug
- energy sensor open energy monitor
- 3DP machine prusalink
- 3DP machine octoprint
- laser machine olsk laser -->

## Integrate resources in Node-Red
### Configure Shelly Data in the MQTT Broker Node

Follow these steps to connect your Shelly device data to the `mqtt-broker` node.

#### 1. Configure the MQTT Server

1. In the **`mqtt-broker` node**, locate the **Server** field.  
2. Click the **pencil icon** to edit the server configuration.  
3. Enter the MQTT broker details:

   - **Server**: IP address of the MQTT broker  
     Example: `192.168.188.124`
   - **Port**: MQTT port  
     Example: `1883`

<!-- ![MQTT server configuration](https://pad.fabcity.hamburg/uploads/1319f9b4-1c77-47c1-8ad4-e2dcf99c44fe.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/1319f9b4-1c77-47c1-8ad4-e2dcf99c44fe.png" width="700"><br>
  <em>MQTT server configuration</em>
</p>

#### 2. Configure the Shelly MQTT Topic

1. In the **Topic** field, enter the MQTT topic where the Shelly device publishes its data.
2. Example topic:

```
SPPS-05/status/switch:0
```

<!-- ![Shelly topic configuration](https://pad.fabcity.hamburg/uploads/6a4a9543-3f91-439a-aa32-687cd014cd54.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/6a4a9543-3f91-439a-aa32-687cd014cd54.png" width="700"><br>
  <em>Shelly topic configuration</em>
</p>

#### Result

After completing these steps, the `mqtt-broker` node will subscribe to the specified Shelly MQTT topic and receive the device's status messages.

### Configure a Prusa 3D Printer in Node-RED

Follow these steps to connect a Prusa 3D printer to Node-RED and enable status monitoring.

#### 1. Configure the Device Settings

1. Open the **`set device config function`** node.
2. Edit the node and add the required configuration fields.
3. Provide values for the following parameters:

   - `deviceId`
   - `ipAddress`
   - `devicetype`
   - `devicebrand`
   - `electricaldeviceid`
   - `electricaldevicetype`
   - `electricaldevicebrand`

4. Use the example configuration below as a reference.

<!-- ![Device configuration example](https://pad.fabcity.hamburg/uploads/dbd05849-e14b-47b6-a3c0-c2d6d31cc683.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/dbd05849-e14b-47b6-a3c0-c2d6d31cc683.png" width="700"><br>
  <em>Device configuration example</em>
</p>

---

#### 2. Configure Authentication for the Prusa Printer

1. Open the **`http request`** node named **`prusa printer status`**.
2. Enable **Use Authentication**.
3. Set the authentication parameters as follows:

   - **Type of authentication**: `Digest Authentication`
   - **Username**: example `maker`
   - **Password**: your printer password

4. Save the configuration.

<!-- ![Authentication configuration](https://pad.fabcity.hamburg/uploads/5071a270-eed9-4934-a6a7-5f05ef9598a9.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/5071a270-eed9-4934-a6a7-5f05ef9598a9.png" width="700"><br>
  <em>Authentication configuration</em>
</p>

---

#### Result

After completing these steps, Node-RED will be able to authenticate with the Prusa printer and retrieve its status information.

---

### Job Tracking and Energy Calculation Setup in Node-RED for Prusa 3d printer

This section describes how job tracking and energy consumption calculations are implemented in Node-RED.

---

#### 1. Job Tracking

Job tracking records when a machine starts and finishes a production job and links the job to the corresponding device and time period.

##### Purpose
- Monitor machine usage during production jobs.
- Track the **start time**, **end time**, and **machine status**.
- Associate machine activity with a specific **job ID**.

##### Implementation

1. **Receive Machine Status**
   - Machine status data is received from connected devices (e.g., Shelly devices, 3D printers, or other machines).
   - Status changes are used to determine when a job begins or ends.

2. **Detect Job Start and Stop**
   - Logic inside a **Function Node** determines:
     - When a machine becomes active → **Job Start**
     - When the machine stops → **Job End**

3. **Attach Job Information**
   - The system attaches metadata such as:
  ``` 
   jobId
   deviceId
   startTime
   endTime
```

4. **Store Job Data**
   - Job tracking information is sent to **InfluxDB** for storage and later visualization in Grafana.

---

#### 2. Energy Consumption Calculation

Energy consumption is calculated using energy data collected from electrical monitoring devices such as **Shelly energy meters**.

##### Purpose
- Measure energy used during machine operation.
- Associate energy usage with specific jobs.
- Enable energy monitoring and efficiency analysis.

##### Implementation

1. **Receive Energy Data**
   - Energy data such as **power**, **voltage**, or **total energy consumption** is received from the device.

2. **Process Energy Data**
   - A **Function Node** extracts the relevant parameters:
   ```
   power
   energy
   timestamp
   ```

3. **Calculate Job Energy Usage**

Energy consumed during a job is calculated using the difference between the start and end energy readings.

```
jobEnergy = endEnergy - startEnergy
```

4. **Store Energy Metrics**
   - The calculated energy consumption is stored in **InfluxDB** for historical analysis and visualization.

---

#### 3. Data Flow Overview

Typical workflow for job tracking and energy monitoring:

1. Machine status and energy data are received by **Node-RED**.
2. Function nodes detect **job start and stop events**.
3. Energy data is processed and associated with the active job.
4. Structured data is stored in **InfluxDB**.
5. **Grafana dashboards** visualize job activity and energy consumption.

---

#### Result

After completing this setup:

- Machine jobs are automatically tracked.
- Energy consumption is calculated per job.
- Data can be analyzed through **Grafana dashboards** for monitoring production efficiency and energy usage.

---

#### Job Tracking and Energy Calculation Code


```
    // -------------------- INPUTS FROM MSG --------------------
    let currentEnergy = msg.energyCounter;
    let machineState = msg.machineState;
    let jobId = msg.jobId;

    // Define machine states
    let activeState = ['WORKING','PRINTING','CUTTING'];
    let idleStates = ['IDLE', 'FINISHED'];
    // ---------------------------------------------------------

    if (!flow.get('jobStarted')) flow.set('jobStarted', false);

    if (activeState.includes(machineState) ){
        if (!flow.get('jobStarted')) {
            flow.set('jobStarted', true);
            flow.set('initialEnergy', currentEnergy);
            flow.set('currentJobId', jobId);
            flow.set('jobStartTime', Date.now());
            node.warn(`Job started. Initial energy: ${currentEnergy}`);
        } else {
            node.warn(`Job is running. Current energy: ${currentEnergy}`);
        }
    } else if (flow.get('jobStarted') && idleStates.includes(machineState)) {
        let initialEnergy = flow.get('initialEnergy');
        let currentJobId = flow.get('currentJobId');
        let energyConsumed = currentEnergy - initialEnergy;
        let jobStartTime = flow.get('jobStartTime');
        let jobEndTime = Date.now();

    let jobResult = {
        jobId: currentJobId,
        energyConsumed: energyConsumed,
        jobDuration: (jobEndTime - jobStartTime) / 1000,
    };

    flow.set('jobStarted', false);
    flow.set('initialEnergy', null);
    flow.set('currentJobId', null);
    flow.set('jobStartTime', null);

    node.warn(`Job completed. Final energy: ${currentEnergy}, Energy consumed: ${energyConsumed}, Duration: ${jobResult.jobDuration} seconds`);
    node.send({ payload: jobResult });
    } else {
        node.warn(`No active job. Current state: ${machineState}, Current energy: ${currentEnergy}`);
    }

    return null; 
```

---

### Configure InfluxDB and Integrate with Node-RED

This section explains how to create the required resources in InfluxDB and configure Node-RED to store device data (e.g., Shelly energy data) in the database.

---

#### 1. Create a New Bucket in InfluxDB

1. Open the **InfluxDB dashboard**.
2. Navigate to **Buckets**.
3. Click **Create Bucket**.
4. Enter the required **Bucket Name**.
5. Click **Create**.

<!-- ![Create InfluxDB bucket](https://pad.fabcity.hamburg/uploads/c58dbf6f-e0d1-4bcb-a0eb-2dbb782182bd.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/c58dbf6f-e0d1-4bcb-a0eb-2dbb782182bd.png" width="100%"><br>
  <em>Create InfluxDB bucket</em>
</p>

---

#### 2. Generate an API Token

1. Go to **API Tokens** in the InfluxDB dashboard.
2. Click **Generate API Token**.
3. Select the required **Access Permissions**.
4. Enter a **Description** for the token.
5. Create and save the token.

> The token will be required later when configuring the InfluxDB connection in Node-RED.

<!-- ![Generate API token](https://pad.fabcity.hamburg/uploads/874a184f-74e3-426e-bf29-e7cc4ad2082d.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/874a184f-74e3-426e-bf29-e7cc4ad2082d.png" width="100%"><br>
  <em>Generate API token</em>
</p>

---

### Using InfluxDB in Node-RED

#### 3. Structure Data for InfluxDB (Function Node)

- **Node Name:** `structure for influx`
- **Type:** Function Node  
- **Purpose:**  
  Structures incoming device data (e.g., Shelly energy data) into a format compatible with InfluxDB, including:
  - **measurement**
  - **fields**
  - **tags**
  - **timestamp**

This transformation ensures the data can be correctly written to the database.

<!-- ![Influx data structure function](https://pad.fabcity.hamburg/uploads/9835422c-d364-493a-9dc1-5613c7f34f9e.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/9835422c-d364-493a-9dc1-5613c7f34f9e.png" width="700">
</p>

---

#### 4. Configure the InfluxDB Node

- **Node Name:** `InfluxDB-microfactory-pi`
- **Type:** Influx Batch Node  
- **Purpose:**  
  Stores structured device energy data in the InfluxDB instance for **historical storage, analysis, and visualization**.

##### Configuration Steps

1. Open the **Influx Batch Node**.
2. Edit the **Server Configuration**.
3. Enter the following details from your InfluxDB setup:

   - **Server URL**
   - **Organization**
   - **Bucket Name**
   - **API Token** (with **read and write access**)

4. Save the configuration.

<!-- ![Influx node configuration](https://pad.fabcity.hamburg/uploads/c69b69f8-2140-4db0-be61-97eb28594d39.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/c69b69f8-2140-4db0-be61-97eb28594d39.png" width="700"><br>
  <em>Influx node configuration</em>
</p>

<!-- ![Influx server configuration](https://pad.fabcity.hamburg/uploads/fbf86ef2-9e34-4733-b049-3db40a5cf1c6.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/fbf86ef2-9e34-4733-b049-3db40a5cf1c6.png" width="700"><br>
  <em>Influx server configuration</em>
</p>

---

#### Result

After completing these steps:

- Device data is **structured using the Function Node**.
- The **Influx Batch Node writes the data to the configured InfluxDB bucket**.
- The stored data can then be used for **monitoring, dashboards, and historical analysis**.

### Configure Grafana and Connect to InfluxDB

This section explains how to connect Grafana to an InfluxDB data source and configure dashboards to visualize the stored device data.

---

#### 1. Connect Grafana to an InfluxDB Data Source

1. Open **Grafana**.
2. Navigate to **Configuration (Settings)**.
3. Select **Data Sources**.
4. Click **Add Data Source**.
5. Choose **InfluxDB** as the data source.

#### Configure the Connection

Set the following parameters:

- **Query Language:** `Flux`
- **URL:** URL of your InfluxDB instance
- **Organization:** Your InfluxDB organization name
- **Token:** API token with access to the bucket
- **Bucket:** Name of the bucket created in InfluxDB
- **Username / Password:** If required by your setup

6. Click **Save & Test** to verify the connection.

<!-- ![Grafana data source setup](https://pad.fabcity.hamburg/uploads/22b1a5b9-1e1c-4398-a4a3-c3d740488848.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/22b1a5b9-1e1c-4398-a4a3-c3d740488848.png" width="100%"><br>
  <em>Grafana data source setup</em>
</p>

<!-- ![Grafana influx configuration](https://pad.fabcity.hamburg/uploads/ceadedc8-8f83-4519-aff8-e5b4366a219f.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/ceadedc8-8f83-4519-aff8-e5b4366a219f.png" width="100%"><br>
  <em>Grafana influx configuration</em>
</p>

---

#### 2. Update Dashboard Queries

1. Navigate to **Dashboards**.
2. Select **Browse**.
3. Open the **Dashboard** (Fleet Operation and Machine Dashboard in example).
4. For each panel in the dashboard:
   - Click **Edit**.
   - Set the **Data Source** to the InfluxDB source you created.
5. Save the panel changes.

<!-- ![Grafana dashboard query setup](https://pad.fabcity.hamburg/uploads/701b8685-daad-405a-8dd3-61994a3bd292.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/701b8685-daad-405a-8dd3-61994a3bd292.png" width="100%"><br>
  <em>Grafana dashboard query setup</em>
</p>

---

#### 3. Configure Dashboard Variables

Dashboard variables allow dynamic filtering of data (e.g., machines or devices).

1. Open the dashboard.
2. Click **Dashboard Settings** (top-left).
3. Navigate to **Variables**.
4. Select the variable you want to edit.
5. Change the **Data Source** to the same InfluxDB data source you configured earlier.
6. Save the changes.

<!-- ![Grafana dashboard variables setup](https://pad.fabcity.hamburg/uploads/21c2be13-db35-438c-a52f-c31703f6305b.png) -->
<p align="center">
  <img src="https://pad.fabcity.hamburg/uploads/21c2be13-db35-438c-a52f-c31703f6305b.png" width="100%"><br>
  <em>Grafana dashboard variables setup</em>
</p>

---

#### Result

After completing these steps:

- Grafana is connected to **InfluxDB** as a data source.
- Dashboard panels query data directly from the database.
- Dashboard variables allow **dynamic filtering and visualization** of device data such as machine status and energy consumption.

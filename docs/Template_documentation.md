# Template Documentation

## Node-RED Configuration

### Template Example for Prusa 3D Printer and Shelly Energy Sensor

![](https://pad.fabcity.hamburg/uploads/5371346e-0f6f-4f14-8a5c-e6a0a5b36851.png)

---

## Flow Description

### 1. Machine Configurations

This section initializes the machine’s configuration when the flow starts.

**Purpose:**  
Sets up essential device parameters (such as printer name, IP, or type) used throughout the flow.

**Key Nodes**

- **Inject** (`Set machine config once`) — triggers once on deployment.
- **Function** (`set device config`) — defines and stores configuration data for later use.

---

### 2. Prusa Printer Flow

Handles communication with the Prusa 3D printer and collects printer status data.

**Purpose:**  
Calls the printer’s API every second to monitor print status, progress, and other telemetry.

**Key Nodes**

- **Inject** (`RUN`) — starts the polling process.
- **Trigger** (`resend every 1s`) — continuously sends requests to the printer.
- **Function** (`url for printer status`) — builds the API URL for status requests.
- **HTTP Request** (`prusa printer status`) — fetches live printer data.
- **Function** (`Store Machine Data`) — processes and formats the received data.
- **Debug** (`Prusa printer telemetry`) — displays the live telemetry in Node-RED.
- **Function** (`structure job process and machine process for influx`) — formats results for InfluxDB.
- **InfluxDB nodes** — store structured machine process and job process data.

---

### 3. Calculate Energy Consumption Per Print

Computes how much energy the printer consumes for each completed job.

**Purpose:**  
Integrates power readings with printer data to estimate energy usage per print.

**Key Nodes**

- **Function** (`Input required variables`) — gathers necessary inputs (energy, machine state, job info).
- **Function** (`Calculate Energy Per Print`) — performs the calculation.
- **Debug** (`Energy Per Print`) — outputs the result for verification.
- **Function** (`structure job for influx`) — formats results to store in InfluxDB.

---

### 4. Energy Sensor Flow

Captures real-time power and energy data from a Shelly energy sensor.

**Purpose:**  
Collects energy readings, stores them, and makes them available for consumption calculations.

**Key Nodes**

- **Input** (`Shelly Data`) — receives data from the Shelly power sensor.
- **Function** (`structure for influx`) — structures data for database storage.
- **InfluxDB output** — stores power telemetry.
- **Function** (`store current energy`) — keeps the latest energy reading for calculations.
- **Debug** (`Shelly telemetry`) — displays live energy sensor data.

---

## Data Structure for InfluxDB

- Retrieves configuration data from the Node-RED flow context (`machine_config_data`).
- Maps incoming payload data to match the InfluxDB data structure.
- Formats the data as an array of measurement objects for InfluxDB.

Each object includes:

- **measurement** — specifies the data source type (e.g., `machine`, `sensor`).
- **fields** — holds dynamic payload values (e.g., printer data, energy readings).
- **tags** — adds metadata such as device ID, brand, and type from configuration.

Features:

- Supports multiple data inputs (e.g., machine data or energy sensor data).
- Ensures consistent and configurable data formatting before writing to InfluxDB.

### Example Structures from the Template

![](https://pad.fabcity.hamburg/uploads/f811a3c5-8df4-4443-9120-7b01d07c8cf1.png)

![](https://pad.fabcity.hamburg/uploads/92a734f1-06ac-46ea-881a-435c8a85f331.png)

---

# Steps to Set Up the Flow

## Step 1 — Configure the Device

Edit the **`set device config` function** node and add the required fields:

- `deviceId`
- `ipAddress`
- `devicetype`
- `devicebrand`
- `electricaldeviceid`
- `electricaldevicetype`
- `electricaldevicebrand`

Example:

![](https://pad.fabcity.hamburg/uploads/dbd05849-e14b-47b6-a3c0-c2d6d31cc683.png)

---

## Step 2 — Configure Prusa Printer Authentication

Edit the **`HTTP request` node** `prusa printer status`.

- Enable **Use Authentication**
- Set **Type of Authentication:** `Digest Authentication`
- Enter:
  - **Username** (example: `maker`)
  - **Password**

![](https://pad.fabcity.hamburg/uploads/5071a270-eed9-4934-a6a7-5f05ef9598a9.png)

---

## Step 3 — Configure InfluxDB Node

Set the **InfluxDB node** to store all the data.

### Configure the Server

Enter:

- **URL** of the InfluxDB server
- **Token**

![](https://pad.fabcity.hamburg/uploads/723381a5-574b-48de-9d2e-4ee7f619ecde.png)

### Configure Database Details

Enter:

- **Organization**
- **Bucket**

![](https://pad.fabcity.hamburg/uploads/ef565c37-bb84-479e-a94b-8383bfdfc81b.png)

---

## Step 4 — Configure Shelly Data in MQTT

Edit the **`mqtt-broker` node**.

### Configure MQTT Server

Click the **pencil icon** and enter:

- **Server:** `192.168.188.124`
- **Port:** `1883`

![](https://pad.fabcity.hamburg/uploads/1319f9b4-1c77-47c1-8ad4-e2dcf99c44fe.png)

### Configure MQTT Topic

Example topic:

```
SPPS-05/status/switch:0
```

![](https://pad.fabcity.hamburg/uploads/6a4a9543-3f91-439a-aa32-687cd014cd54.png)

---

# Grafana Setup Documentation

## Connecting to the Data Source

1. Go to **Configuration (Settings) → Data Sources**
2. Click **Add data source**
3. Select **InfluxDB**
4. Select **Query Language: Flux**
5. Enter:

- URL of the InfluxDB server
- Username
- Password
- Organization
- Token
- Bucket name

6. Click **Save & Test** to check the connection.

![](https://pad.fabcity.hamburg/uploads/22b1a5b9-1e1c-4398-a4a3-c3d740488848.png)

![](https://pad.fabcity.hamburg/uploads/ceadedc8-8f83-4519-aff8-e5b4366a219f.png)

---

## Dashboard Query Updates

1. Go to:

```
Dashboards → Browse → Fleet Operation and Machine dashboard
```

2. For each panel:

- Click **Edit**
- Select the **InfluxDB data source** (or the name you used when connecting the DB)

![](https://pad.fabcity.hamburg/uploads/701b8685-daad-405a-8dd3-61994a3bd292.png)

---

## Dashboard Variables

1. Select the **Dashboard Settings** from the top left.
2. Go to **Variables**.
3. Select each variable and change the **data source** to your InfluxDB data source.

![](https://pad.fabcity.hamburg/uploads/21c2be13-db35-438c-a52f-c31703f6305b.png)

---

# Docker Volume Backup Documentation

## Create Backup Directory

```bash
mkdir -p ~/docker-backups
cd ~/docker-backups
```

Create two files:

- `backup_volumes.sh`
- `restore_volumes.sh`

---

# Backup Script

```bash
#!/bin/bash
# Backup Docker volumes
# Usage: ./backup_volumes.sh

BACKUP_DIR=backups_$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# List of volumes to back up
VOLUMES=(
  container_grafana-dashboards
  container_grafana-data
  container_influxdb2
  container_jupyter-notebook-data
  container_nodered-data
  container_server-data
)

echo "Starting backup of Docker volumes..."

for VOLUME in "${VOLUMES[@]}"; do
  echo "Backing up $VOLUME..."
  docker run --rm \
    -v ${VOLUME}:/volume \
    -v $(pwd)/${BACKUP_DIR}:/backup \
    alpine sh -c "cd /volume && tar -czf /backup/${VOLUME}.tar.gz ."
done

echo "Backup complete!"
echo "Backups saved in: $(pwd)/${BACKUP_DIR}"
```

---

# Restore Script

```bash
#!/bin/bash
# Restore Docker volumes from backup folder
# Usage: ./restore_volumes.sh backups_20251105_142210

BACKUP_DIR=$1

if [ -z "$BACKUP_DIR" ]; then
  echo "Please specify the backup directory (e.g., ./restore_volumes.sh backups_20251105_142210)"
  exit 1
fi

if [ ! -d "$BACKUP_DIR" ]; then
  echo "Backup directory not found: $BACKUP_DIR"
  exit 1
fi

echo "Starting restore from $BACKUP_DIR..."

for FILE in ${BACKUP_DIR}/*.tar.gz; do
  VOLUME=$(basename ${FILE%.tar.gz})
  echo "Restoring $VOLUME..."

  docker volume create $VOLUME >/dev/null 2>&1

  docker run --rm \
    -v ${VOLUME}:/volume \
    -v $(pwd)/${BACKUP_DIR}:/backup \
    alpine sh -c "cd /volume && tar -xzf /backup/${VOLUME}.tar.gz"
done

echo "Restore complete!"
```

---

# Make Scripts Executable

```bash
chmod +x backup_volumes.sh
chmod +x restore_volumes.sh
```

---

# Usage

## Backup Docker Volumes

```bash
./backup_volumes.sh
```

This creates a folder similar to:

```
backups_20251105_142210
```

---

## Restore Docker Volumes

```bash
./restore_volumes.sh backups_20251105_142210
```
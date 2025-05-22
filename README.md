# IoTronic Plugin: Environmental Data Publisher

This plugin simulates a real-time environmental data stream by reading rows from a CSV file, assigning each one a fresh timestamp, and sending it to InfluxDB every 30 seconds.

It is designed to be used within the Stack4Things framework, but can also be adapted for standalone testing.

---

## Features

- Downloads a CSV dataset from a **public Google Drive URL**
- Saves the file locally in `/opt/data`
- Verifies file integrity before use
- Converts each CSV row into a structured InfluxDB data point
- Sends data with **current timestamp** (real-time simulation)
- Waits 30 seconds between each entry
- Uses a `last_index.state` file to **resume where it left off**

---

## InfluxDB Deployment with Docker

To run InfluxDB locally using Docker:

```bash
docker run -d \
  --name=influxdb \
  --restart unless-stopped \
  -p 8086:8086 \
  -e INFLUXDB_HTTP_AUTH_ENABLED=true \
  -e INFLUXDB_ADMIN_USER=admin \
  -e INFLUXDB_ADMIN_PASSWORD=admin \
  -v influxdb_data:/var/lib/influxdb \
  influxdb:1.8
```
## Troubleshooting

If the plugin does not seem to send data, check the following:

- ✅ Is InfluxDB running and accessible on `localhost:8086`?
- ✅ Has the CSV been correctly downloaded to `/opt/data/structured_time_series_v2.csv`?
- ✅ Are all Python packages installed correctly?
- ✅ Are logs printed to console (in case `oslo.log` is not configured)?
- ✅ Does the CSV contain at least one valid data row?

You can manually verify InfluxDB contents using:

```bash
docker exec -it influxdb influx -username admin -password admin
```
This opens the interactive InfluxDB shell with your credentials.

---

### 2. Create the Database

Once inside the shell, run the following commands:

```sql
CREATE DATABASE secco;
SHOW DATABASES;
```
You should see `secco` listed in the output:
```yaml
name: databases
name
_internal
secco
```

This confirms that the database was successfully created.

---

### 3. Exit the Influx Shell

To exit the Influx CLI:

```sql
exit
```
---

### 4 Deploying the Plugin in the Stack4Things / IoTronic Platform

To integrate this plugin into a real IoT deployment managed by the Stack4Things framework, follow the standard plugin lifecycle supported by IoTronic.

### 4.1. Plugin Creation

The plugin must be written in Python and structured as a class named `Worker`, inheriting from the base `Plugin` class provided by Lightning-rod. 

### 4.2. Deployment on the Board

Once the plugin is created in Stack4Things, it is copied onto the target form in Stack4Things panel. 

### 4.3. Injection via IoTronic

After the plugin is available on the S4T dashboard, it must be injected into the board. This step associates the plugin with a specific board and makes it manageable through the IoTronic APIs and dashboard.

The injection process involves assigning a name to the plugin.

### 4.4. Plugin Activation

Once injected, the plugin becomes available to be started. The IoTronic controller can request the Lightning-rod agent to start the plugin at any time. Upon activation, the plugin will begin its execution loop, reading the dataset, sending data to InfluxDB, and logging progress.

At any moment, the plugin can also be stopped or restarted from the IoTronic interface or controller, allowing full lifecycle control.


### 5. Validate Data Insertion (After Plugin Starts)

Once the plugin begins publishing data, you can return to the InfluxDB shell to check that points are being written correctly.

Run the following commands:

```sql
USE secco;
SHOW MEASUREMENTS;
SELECT * FROM environmental_data LIMIT 5;
```
If everything is working, you should see something like:
```sql
name: environmental_data
time Gust Humidity Temperature ...
2025-05-22T08:30:00Z 0.0 82.0 18.6 ...
2025-05-22T08:30:30Z 0.8 83.0 18.7 ...
2025-05-22T08:31:00Z 1.2 84.0 18.9 ...
```

### 6. Official IoTronic Repositories

The IoTronic ecosystem is actively developed and maintained under the [OpenDev](https://opendev.org/) community infrastructure.

You can explore all related repositories here:  

- [**iotronic**](https://opendev.org/x/iotronic)  
  IoTronic is the Internet of Things resource management service for OpenStack clouds.

- [**iotronic-lightning-rod**](https://opendev.org/x/iotronic-lightning-rod)  
  Lightning-rod is the IoT node-side agent responsible for executing remote procedures and managing plugins.

- [**python-iotronicclient**](https://opendev.org/x/python-iotronicclient)  
  Python client for interacting programmatically with the IoTronic service APIs.

- [**iotronic-ui**](https://opendev.org/x/iotronic-ui)  
  IoTronic plugin for the OpenStack Horizon Dashboard, enabling graphical device and board management.

These components form the complete IoTronic platform for federated, cloud-native IoT orchestration.

---

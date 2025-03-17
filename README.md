# backend-task-faradaIC-sensors

![System Overview](https://github.com/liquiir97/backend-task-faradaIC-sensors/blob/main/Architecture1.png)

## Data Transfer Architecture

### High-Level Flow:
  * Sensors read data from the outside world.
  * Using IoT devices, data is sent to **CNS** in **ALOHA** mode.
  * **CNS** receives data and stores it in a **MySQL DB**.
  * **BAS** reads data from MySQL instances based on availability.
  * **BAS** sends responses to devices (PC, mobile) for requested data.

The approach used in this flow is that we have **ONE main DB (master)**, which is used mostly for **write operations** (see image **Architecture1 MySQL write**). From this DB, we create several other DB replicas that will be used for **read operations**. **BAS** will access these DB replicas when data is requested. This approach achieves **real-time updating**: when data is written to the master DB, it is replicated to the replicas.

### Estimated Throughput Metrics:
  * **1 device → 1 packet per hour (p/h)**
  * **10,000 devices → 10,000 packets per hour (p/h)**
  * **1 packet size → 100 bytes**
  * **Packets per second** → `10000 p/h / 3600 s = 2.78 p/s`
  * **Data size** → `2.78 p/s * 100 b = 278 b/s`

#### Per Day:
  * **Packets per day** → `10000 p/h * 24 h = 240,000 p/d`
  * **Total bytes per day** → `240,000 p/d * 100 b = 24,000,000 b/d`
  * **Convert to GB** → `24,000,000 / (1024 * 1024 * 1024) = 0.024 GB/d`
  
#### Per Month:
  * **0.024 GB/day = 0.72 GB/month**

### Potential Bottlenecks:
  * **Master DB downtime**: A potential problem is when the master DB used for write operations is down. A solution can be to declare one of the replicas as a **vice-master**, so if the master is down, the vice-master steps up as the master DB.
  * **Network latency**: If replicas are in different regions, it can cause latency.
  * **High request volume**: If there are too many requests, delays may happen. Solutions include adding a **load balancer** for replicas, **adding indexes** in the DB, and increasing the number of **replicas**.
  * **Too many write operations**: Adding additional resources or optimizing the database might be necessary.

### Scaling for 100,000 Devices:
  * Add more **replicas**.
  * **Check and optimize indexes**.
  * **Restructure DB tables** for better performance.
  * Add a **load balancer** for DB read operations.

---

## Downlink Management System

### General Flow:
  * **BAS** accepts requests from **PC** or **Mobile**.
  * **BAS** sends requests to **CNS** using **REST**.
  * **CNS** stores the requests in the master DB.
  * When the scheduled window opens, data is sent to devices.
  * The status of scheduled jobs is tracked based on their success (e.g., scheduled, failed, finished, canceled, pending).

### API Endpoints:

#### 1. Schedule New Downlink Request
- **POST** `/api/v1/downlink-packets`
- **Request Body**:
  ```json
  {
    "deviceId": int,
    "sensorData": "string"  // Sensor data used for detection based on the sensor
  }
  ```
- **Response**:
```json
  {
    "deviceId": int,
    "sensorData": "string"  // Sensor data used for detection based on the sensor
  }
  ```

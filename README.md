# backend-task-faradaIC-sensors

![System Overview](https://github.com/liquiir97/backend-task-faradaIC-sensors/blob/main/Architecture1.png)


## Data transfer Architecture

High level flow in this application is:
  * sensors read data from outer world
	* using IoT devices data is send to CNS in ALOHA mode
	* CNS recives data and store it in MySQL DB
	* BAS reads data from MySql instances based on availability
	* BAS sends response to devices (PC, mobile) for requsted data

Approach that is used in this flow is that we have ONE main DB(master) which is used in this case mostly for write operation (see on image Archictecture1 MySql write). From that DB we create several other DB, replicas, that will be used for reading operation. BAS will access to those DB when some data is requested. With this approach real-time updateting is almost achived, when data is written in main DB that that will be replicated to replicas.

### Provide estimated throughput metrics
  * 1 device -> 1 p/h
	* 10000 devices -> 10000 p/h
	* 1 p size -> 100 b
	* packet per seconds -> 10000 p/h / 3600 s = 2.78 p/s
	* data size -> 2.78 p/s * 100 b = 278 b/s

	* packet per day -> 10000 p/h * 24 h = 240000 p/d
	* total bytes per day -> 240000 p/d * 100 b = 24000000 b/d
	* convert to GB -> 24000000 % (1024 * 1024 * 1024) =  0.024 GB/d
	* per month -> 0.024 GB/d = 0.72 GB/m

### Discuss potential bottlenecks
  * for this approcah problem can be when master DB that is used for write operation is down. Solution can be to one of those replicas declare as vice master, so if master is down vice master step up as master DB
	* network latency: in cases when replicas are in diferenet regions can couse latency
	* if there is a lot of requests delay can happen, so adding load balancer for replicas, adding indexes in DB,adding more replicas
	* to many write operation

### Explain how your solution would scale if device count increases to 100,000
  * add more replicas
	* check index
	* restructure DB tables
	* add load balancer for DB read operation

## Downlink Management System
General flow:
	* BAS accepts request from PC/Mobile
	* BAS sends request to CNS using REST
	* CNS accepts request from BAS store it in master DB
	* when brief window data is sent to devices
	* status of checduled job is mapped based on succession of operation(scheduled, failed, finished, canceled, pending)

### API

  1. schedule new

	```json


	/api/v1/downlink-packets:
	POST: creating new request
	body:
	{
		deviceId : int,
		sensorData : string (some values for sensors that will be used for detections, based on sensor),
	}

	response:
	{
		downLinkId : int,
		deviceId : int,
		status : string ("scheduled"),
		date_time : datetime,
	};
	```

  3. get all 
    /api/v1/downlink-packets
	  GET
	  response:
	  [
		  {
        downLinkId : int,
			  deviceId : int,
			  status : string ("scheduled"),
			  date_time : datetime,
		  },
		  {
			  downLinkId : int,
			  deviceId : int,
			  status : string ("scheduled", "finished", "canceled"),
			  date_time : datetime,

		  }
	  ];  
  
  4. get downlink-packet based on downlink 
    GET
	  /api/v1/downlink-packets/{downLinkId}
	  reposnse
	  {  
		  downLinkId : int,
		  deviceId : int,
		  status : string ("scheduled", "finished", "canceled", ...),
		  date_time : datetime,
	  }
  
 5. cancel downlink-packets
    DELETE
	  /api/v1/downlink-packets/{downLinkId}
	  response
	  {
		  downLinkId : int,
		  deviceId : int,
		  status : string ("canceled"),
		  date_time : datetime,
	  }

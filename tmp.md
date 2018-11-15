# Chapter 3
## 3.1 Key Concepts and Terminology
The following basic concepts are involved in IoTDB:
* device
* sensor
* storage group
* path
* timeseries path
* prefix path
* path with star
* timestamp
* value
* point
* column

A detailed description of the above concepts will be given in sections 3.1.1 to 3.1.11 of this chapter.

### 3.1.1 Device
A devices is an installation equipped with sensors in real scenarios. In IoTDB, all sensors should have their corresponding devices.

### 3.1.2 Sensor
A sensor is a detection equipment in an actual scene, which can sense the information to be measured, and can transform the sensed information into an electrical signal or other desired form of information output and send it to IoTDB. In IoTDB, all data and paths stored are organized in units of sensors.

### 3.1.3 Storage Group
Storage groups are used to let users define how to organize and isolate different time series data on disk. Time series belonging to the same storage group will be continuously written to the same file in the corresponding folder. The file may be closed due to user commands or system policies, and hence the data coming next from these sensors will be stored in a new file in the same folder. Time series belonging to different storage groups are stored in different folders.

Users can set any prefix path as a storage group. Provided that there are four time series `root.vehicle.d1.s1`, `root.vehicle.d1.s2`, `root.vehicle.d2.s1`, `root.vehicle.d2.s2`, two devices `d1` and `d2` under the path `root.vehicle` may belong to the same owner or the same manufacturer, so d1 and d2 are closely related. At this point, the prefix path root.vehicle can be designated as a storage group, which will enable IoTDB to store all devices under it in the same folder. Newly added devices under `root.vehicle` will also belong to this storage group.

Note: A full path (`root.vehicle.d1.s1` as in the above example) is not allowed to be set as a storage group.

Setting a reasonable number of storage groups can lead to performance gains: there is neither the slowdown of the system due to frequent switching of IO (which will also take up a lot of memory and result in frequent memory-file switching) caused by too many storage files (or folders), nor the block of write commands caused by too few storage files (or folders) (which reduces concurrency).

Users should balance the storage group settings of storage files according to their own data size and usage scenarios to achieve better system performance. (There will be officially provided storage group scale and performance test reports in the future).
Note: The prefix of a time series must belong to a storage group. Before creating a time series, the user must set which storage group the series belongs to. Only the time series whose storage group is set can be persisted to disk.

Once a prefix path is set as a storage group, the storage group settings cannot be changed.

After a storage group is set, all parent and child layers of the corresponding prefix path are not allowed to be set up again (for example, after `root.ln` is set as the storage group, the root layer and `root.ln.wf01` are not allowed to be set as storage groups).

### 3.1.4 Path
In IoTDB, a path is an expression that conforms to the following constraints:
```
path: LayerName (DOT LayerName)+
LayerName: Identifier | STAR
```
Among them, STAR is "*" and DOT is ".".

We call the middle part of a path between two "." as a layer, and thus `root.A.B.C` is a path with four layers. 

It is worth noting that in the path, root is a reserved character, which is only allowed to appear at the beginning of the time series mentioned below. If root appears in other layers, it cannot be parsed and an error is reported.

### 3.1.5 Timeseries Path
The timeseries path is the core concept in IoTDB. A timeseries path can be thought of as the complete path of a sensor that produces the time series data. All timeseries paths in IoTDB must start with root and end with the sensor. A timeseries path can also be called a full path.

For example, if device1 of the vehicle type has a sensor named sensor1, its timeseries path can be expressed as: `root.vehicle.device1.sensor1`.

Note: The layer of timeseries paths supported by the current IoTDB must be greater than or equal to four (it will be changed to two in the future).

### 3.1.6 Prefix Path
The prefix path refers to the path where the prefix of a timeseries path is located. A prefix path contains all timeseries paths prefixed by the path. For example, suppose that we have three sensors: `root.vehicle.device1.sensor1`, `root.vehicle.device1.sensor2`, `root.vehicle.device2.sensor1`, the prefix path `root.vehicle.device1` contains two timeseries paths `root.vehicle.device1.sensor1` and `root.vehicle.device1.sensor2` while `root.vehicle.device2.sensor1` is excluded.

### 3.1.7 Path With Star
In order to make it easier and faster to express multiple timeseries paths or prefix paths, IoTDB provides users with the path pith star. "\*" can appear in any layer of the path. According to the position where "\*" appears, the path with star can be divided into two types:

"\*" appears at the end of the path;

"\*" appears in the middle of the path;

When "\*" appears at the end of the path, it represents ("\*")+, which is one or more layers of "\*". For example, `root.vehicle.device1.*` represents all paths prefixed by `root.vehicle.device1` with layers greater than or equal to 4, like `root.vehicle.device1.*`, `root.vehicle.device1.*.*`, `root.vehicle.device1.*.*.*`, etc.

When "\*" appears in the middle of the path, it represents "\*" itself, i.e., a layer. For example, `root.vehicle.*.sensor1` represents a 4-layer path which is prefixed with `root.vehicle` and suffixed with `sensor1`.   

Note: "\*" cannot be placed at the beginning of the path.

Note: A path with "\*" at the end has the same meaning as a prefix path, e.g., `root.vehicle.*` and `root.vehicle` is the same.

### 3.1.8 Timestamp
The timestamp is the time point at which a data arrives. IoTDB timestamps are divided into two types: LONG and DATETIME (including DATETIME-INPUT and DATETIME-DISPLAY). When a user enters a timestamp, he can use a LONG type timestamp or a DATETIME-INPUT type timestamp, where the support format of the DATETIME-INPUT type timestamp is shown in Table 3-1.

Table 3-1 Support format of DATETIME-INPUT type timestamp

|format|
|:---:|
|yyyy-MM-dd HH:mm:ss|
|yyyy/MM/dd HH:mm:ss|
|yyyy.MM.dd HH:mm:ss|
|yyyy-MM-dd'T'HH:mm:ss|
|yyyy/MM/dd'T'HH:mm:ss|
|yyyy.MM.dd'T'HH:mm:ss|
|yyyy-MM-dd HH:mm:ssZZ|
|yyyy/MM/dd HH:mm:ssZZ|
|yyyy.MM.dd HH:mm:ssZZ|
|yyyy-MM-dd'T'HH:mm:ssZZ|
|yyyy/MM/dd'T'HH:mm:ssZZ|
|yyyy.MM.dd'T'HH:mm:ssZZ|
|yyyy/MM/dd HH:mm:ss.SSS|
|yyyy-MM-dd HH:mm:ss.SSS|
|yyyy.MM.dd HH:mm:ss.SSS|
|yyyy/MM/dd'T'HH:mm:ss.SSS|
|yyyy-MM-dd'T'HH:mm:ss.SSS|
|yyyy.MM.dd'T'HH:mm:ss.SSS|
|yyyy-MM-dd HH:mm:ss.SSSZZ|
|yyyy/MM/dd HH:mm:ss.SSSZZ|
|yyyy.MM.dd HH:mm:ss.SSSZZ|
|yyyy-MM-dd'T'HH:mm:ss.SSSZZ|
|yyyy/MM/dd'T'HH:mm:ss.SSSZZ|
|yyyy.MM.dd'T'HH:mm:ss.SSSZZ|
|ISO8601 standard time format|






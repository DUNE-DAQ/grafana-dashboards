# Grafana DAQ dashboard repository

This repository collects graphana dashboards created to support DAQ operations and testing.

| folder                     | description                              |
| --                         | ---                                      |
| dashboard                  | DUNE DAQ system overview dashboards      |
| external-dashboards        | Dashboard to monitor infrastructure tools|

# Info for developers
The most important documentation to develop DUNE DAQ dashboards are
 - [grafana documentation](https://grafana.com/docs/)
 - [InfluxDB documentation](https://docs.influxdata.com/influxdb/v1/)

They respectively cover the graphic tools and the database that contains the data.
This is assumed to be known in the following. 

Yet, in addition, it's necessary to know how the database is structured to be able to form the desired queries. 
This documentation covers the particulars of the system we are monitoring. 

## Common variables of the dashboards
Each dasbhoard has a number of standard variables used everywhere. 
 - influxdb
 - session

The first one is an automatic way to pick up the right data source. 
The second one is an easy way to select data only from the right session. 

### Data source variable
This variable is always hidden and the query can be the same for every dashboard.

### Session variable
Most dashboards are describing content that only makes sense within a session. 
Hence, these dashboards need a variable called session. 
It's important that the variable name is the same as other dashboard (case sensitive) in this way you can automatically select the same variable when opening a dashboard from the main entry one. 
An example of query is 
```
SELECT "session" FROM (SELECT "state","session" FROM "dunedaq.appfwk.opmon.AppInfo" WHERE $timeFilter)
```
But, it's good practice to change the `FROM` block so that the session is taken from a measurement that is actually used in the dashboard.
Please note that since the session is stored in InfluxDB as a tag, the right way to extract the `session` correctly is via a nested query. 

### Other common practices for variables
For measurements that are published by more that one object in a session, we tend to use a common pattern. 
The patterns is 
 - having multi-value variables to allow a selection of the sources 
 - the queries are filtering the data via the variables defined in the previous step.
Examples of this are application and DLH in the readout dashboard.

## Data structure 
`opmonlib` describes the way we publish data. 
Behind the scene, `opmonlib` turns protobuf objects into `OpMonEntry`s and a [microservice](https://github.com/DUNE-DAQ/microservices/tree/develop/opmon-protobuf-dbwriter) transforms them into InfluxDB measurements. 
Understanding the mapping is the key ingredient to write effective queries. 

### Measurement
The name of the measurement, a.k.a. the content of the `FROM` part of the query is the name of the protobuf message, including it's namespace, which is defined in the `package` line of the protobuf. 

Example:
[DataWriterInfo](https://github.com/DUNE-DAQ/dfmodules/blob/52ec3406ad8957cc4d921844cb68e3729e03b1ac/schema/dfmodules/opmon/DataWriter.proto#L5) objects are turned into `dunedaq.dfmodules.opmon.DataWriterInfo` measurements. 

### Fields
The fields of a measurements are created by the content of the message. 
They are the quantities you can `SELECT` from the database. 

### Tags
Tags are important features of InfluxDB that allow us to organise the data in the database. 
Through the way DUNE DAQ publishes data, we define a set of tags; some are common for every measurement and some are specific for the measurement. 

#### Default tags
Every measurement in the database has a set of standard tags. 
Two are mandatory: `session` and `application`. 
Then we have `element`, `subelement`, `subsubelement`, etc. which reflect the nested structure that monitoring objects can have in a C++ environment. 
Notable examples:
 - DAQModules names are stored in `element`
 - Queues and connections are stored in `subelement`
 
#### Custom tags
Lines like this
```C++
publish(std::move(td_info), { { "type", name } });
```
will allow to define custom tags. 
In this case the measurement has an additional type called `type` whose value is whatever is the content of "name". 

### Entries from python environment
Metrics from drunc are unlikely to have a substructure that goes beyond `session` and `application` as there are DAQModules defined in python. 
But it's common to have metrics with custom tags. 
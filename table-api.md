<!--
Copyright (c) 2017 Dell Inc., or its subsidiaries. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
-->

The Flink connector library for Pravega provides a table source and table sink for use with the Flink Table API.  The Table API provides a unified API for both the Flink streaming and batch environment.  See the below sections for details.

## Table of Contents
- [Table Source](#table-source)
  - [Parameters](#parameters)
  - [Custom Formats](#custom-formats)
  - [Time Attribute Support](#time-attribute-support)
- [Table Sink](#table-sink)
  - [Parameters](#parameters-1)
  - [Custom Formats](#custom-formats-1)

## Table Source
A Pravega stream may be used as a table source within a Flink table program.  The Flink Table API is oriented around Flink's `TableSchema` classes which describe the table fields.  A concrete subclass of `FlinkPravegaTableSource` is then used to parse raw stream data as `Row` objects that conform to the table schema.  The connector library provides out-of-box support for JSON-formatted data with `FlinkPravegaJsonTableSource`, and may be extended to support other formats.

Here's an example of using the provided table source to read JSON-formatted events from a Pravega stream:
```
// Create a Flink Table environment
ExecutionEnvironment  env = ExecutionEnvironment.getExecutionEnvironment();

// Load the Pravega configuration
PravegaConfig config = PravegaConfig.fromParams(params);

String[] fieldNames = {"user", "uri", "accessTime"};

// read data from the stream using Table reader
TableSchema tableSchema = TableSchema.builder()
        .field("user", Types.STRING())
        .field("uri", Types.STRING())
        .field("accessTime", Types.SQL_TIMESTAMP())
        .build();

FlinkPravegaJsonTableSource source = FlinkPravegaJsonTableSource.builder()
                                        .forStream(stream)
                                        .withPravegaConfig(pravegaConfig)
                                        .failOnMissingField(true)
                                        .withRowtimeAttribute("accessTime",
                                                new ExistingField("accessTime"),
                                                new BoundedOutOfOrderTimestamps(30000L))
                                        .withSchema(tableSchema)
                                        .withReaderGroupScope(stream.getScope())
                                        .build();

// (option-1) read table as stream data 
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);
tableEnv.registerTableSource("MyTableRow", source);
String sqlQuery = "SELECT user, count(uri) from MyTableRow GROUP BY user";
Table result = tableEnv.sqlQuery(sqlQuery);
...

// (option-2) read table as batch data (use tumbling window as part of the query)
BatchTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);
tableEnv.registerTableSource("MyTableRow", source);
String sqlQuery = "SELECT user, " +
        "TUMBLE_END(accessTime, INTERVAL '5' MINUTE) AS accessTime, " +
        "COUNT(uri) AS cnt " +
        "from MyTableRow GROUP BY " +
        "user, TUMBLE(accessTime, INTERVAL '5' MINUTE)";
Table result = tableEnv.sqlQuery(sqlQuery);
...

```

### Parameters
A builder API is provided to construct an instance of `FlinkPravegaJsonTableSource`.  See the table below for a summary of builder properties.  Note that the builder accepts an instance of `PravegaConfig` for common configuration properties.  See the [configurations](configurations) page for more information.

Note that the table source supports both the Flink streaming and batch environments.  In the streaming environment, the table source uses a `FlinkPravegaReader` connector ([ref](streaming)); in the batch environment, the table source uses a `FlinkPravegaInputFormat` connector ([ref](batch)).  Please see the documentation for the respective connectors to better understand the below parameters.


|Method                |Description|
|----------------------|-----------------------------------------------------------------------|
|`withPravegaConfig`|The Pravega client configuration, which includes connection info, security info, and a default scope.|
|`forStream`|The stream to be read from, with optional start and/or end position.  May be called repeatedly to read numerous streams in parallel.|
|`uid`|The uid to identify the checkpoint state of this source.  _Applies only to streaming API._|
|`withReaderGroupScope`|The scope to store the reader group synchronization stream into.  _Applies only to streaming API._|
|`withReaderGroupName`|The reader group name for display purposes.  _Applies only to streaming API._|
|`withReaderGroupRefreshTime`|The interval for synchronizing the reader group state across parallel source instances.  _Applies only to streaming API._|
|`withCheckpointInitiateTimeout`|The timeout for executing a checkpoint of the reader group state.  _Applies only to streaming API._|
|`withSchema`|The table schema which describes which JSON fields to expect.|
|`withProctimeAttribute`|The name of the processing time attribute in the supplied table schema.|
|`withRowTimeAttribute`|supply the name of the rowtime attribute in the table schema, a TimeStampExtractor instance to extract the rowtime attribute value from the event and a WaterMarkStratergy to generate watermarks for the rowtime attribute.|
|`failOnMissingField`|A flag indicating whether to fail if a JSON field is missing.|

### Custom Formats
To work with stream events in a format other than JSON, extend `FlinkPravegaTableSource`.  Please look at the implementation of `FlinkPravegaJsonTableSource` ([ref](https://github.com/pravega/flink-connectors/blob/master/src/main/java/io/pravega/connectors/flink/FlinkPravegaJsonTableSource.java)) for details.

### Time Attribute Support
With the use of `withProctimeAttribute` or `withRowTimeAttribute` builder method, one could supply the time attribute information of the event. The configured field must be present in the table schema and of type Types.SQL_TIMESTAMP()

## Table Sink
A Pravega stream may be used as an append-only table within a Flink table program.  The Flink Table API is oriented around Flink's `TableSchema` classes which describe the table fields.  A concrete subclass of `FlinkPravegaTableSink` is then used to write table rows to a Pravega stream in a particular format.  The connector library provides out-of-box support for JSON-formatted data with `FlinkPravegaJsonTableSource`, and may be extended to support other formats.

Here's an example of using the provided table sink to write JSON-formatted events to a Pravega stream:
```java
// Create a Flink Table environment
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// Load the Pravega configuration
PravegaConfig config = PravegaConfig.fromParams(ParameterTool.fromArgs(args));

// Define a table (see Flink documentation)
Table table = ...

// Write the table to a Pravega stream
FlinkPravegaJsonTableSink sink = FlinkPravegaJsonTableSink.builder()
    .forStream("sensor_stream")
    .withPravegaConfig(config)
    .withRoutingKeyField("sensor_id")
    .withWriterMode(EXACTLY_ONCE)
    .build();
table.writeToSink(sink);
```

### Parameters
A builder API is provided to construct an instance of `FlinkPravegaJsonTableSink`.  See the table below for a summary of builder properties.  Note that the builder accepts an instance of `PravegaConfig` for common configuration properties.  See the [configurations](configurations) wiki page for more information.

Note that the table sink supports both the Flink streaming and batch environments.  In the streaming environment, the table sink uses a `FlinkPravegaWriter` connector ([ref](streaming)); in the batch environment, the table sink uses a `FlinkPravegaOutputFormat` connector ([ref](batch)).  Please see the documentation for the respective connectors to better understand the below parameters.

|Method                |Description|
|----------------------|-----------------------------------------------------------------------|
|`withPravegaConfig`|The Pravega client configuration, which includes connection info, security info, and a default scope.|
|`forStream`|The stream to be written to.|
|`withWriterMode`|The writer mode to provide best-effort, at-least-once, or exactly-once guarantees.|
|`withTxnTimeout`|The timeout for the Pravega transaction that supports the exactly-once writer mode.|
|`withSchema`|The table schema which describes which JSON fields to expect.|
|`withRoutingKeyField`|The table field to use as the routing key for written events.|

### Custom Formats
To work with stream events in a format other than JSON, extend `FlinkPravegaTableSink`.  Please look at the implementation of `FlinkPravegaJsonTableSink` ([ref](https://github.com/pravega/flink-connectors/blob/master/src/main/java/io/pravega/connectors/flink/FlinkPravegaJsonTableSink.java)) for details.

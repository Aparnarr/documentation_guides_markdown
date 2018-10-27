<!--
Copyright (c) 2017 Dell Inc., or its subsidiaries. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
-->

## Metrics

Pravega metrics are collected and exposed to Flink metrics framework when using `FlinkPravegaReader` or `FlinkPravegaWriter` instances to perform read and write operations on Pravega streams.

### Reader Metrics

The following metrics are exposed and made available as Flink metrics for `FlinkPravegaReader` related operations.

- reader group name
- scope
- stream names
- online readers
- segment positions
- unread bytes

Metric Name                |Description|
|-----------------|-----------------------------------------------------------------------|
|`readerGroupName`|The name of the reader group.|
|`scope`|The scope name of the reader group.|
|`streams`|The streams that are part of the reader group. The stream name will be a fully qualified name i.e., `scope/stream`|
|`onlineReaders`|The readers from the reader group that are currently online/available.|
|`segmentPositions`|The StreamCut information that indicates where the readers have read so far.|
|`unreadBytes`|The total number of bytes that have not been read yet.|

### Writer Metrics

For `FlinkPravegaWriter` related operations, only the stream name is exposed.

### Querying Metrics

The metrics can be viewed either from Flink UI or using the Flink Rest API (see below for example).

```
curl -i -s -f <FLINK-HOST:PORT>/jobs/<JOB-ID>/vertices/<SOURCE-TASK-ID>/metrics?get=0.Source__<SOURCE-READER-NAME>.PravegaReader.readerGroup.readerGroupName

curl -i -s -f localhost:1028/jobs/<JOB-ID>/vertices/<SOURCE-TASK-ID>/metrics?get=0.Source__<SOURCE-READER-NAME>.PravegaReader.readerGroup.scope

curl -i -s -f localhost:1028/jobs/<JOB-ID>/vertices/<SOURCE-TASK-ID>/metrics?get=0.Source__<SOURCE-READER-NAME>.PravegaReader.readerGroup.streams

curl -i -s -f localhost:1028/jobs/<JOB-ID>/vertices/<SOURCE-TASK-ID>/metrics?get=0.Source__<SOURCE-READER-NAME>.PravegaReader.readerGroup.onlineReaders

curl -i -s -f localhost:1028/jobs/<JOB-ID>/vertices/<SOURCE-TASK-ID>/metrics?get=0.Source__<SOURCE-READER-NAME>.PravegaReader.readerGroup.stream.test.segmentPositions

curl -i -s -f localhost:1028/jobs/<JOB-ID>/vertices/<SOURCE-TASK-ID>/metrics?get=0.Source__<SOURCE-READER-NAME>.PravegaReader.readerGroup.unreadBytes

```
 

<!--
Copyright (c) 2017 Dell Inc., or its subsidiaries. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
-->
# Pravega Flink Connectors [![Build Status](https://travis-ci.org/pravega/flink-connectors.svg?branch=master)](https://travis-ci.org/pravega/flink-connectors)

This repository implements  connectors to read and write [Pravega](http://pravega.io/) Streams with [Apache Flink](http://flink.apache.org/) stream processing framework.

The connectors can be used to build end-to-end stream processing pipelines (see [Samples](https://github.com/pravega/pravega-samples)) that use Pravega as the stream storage and message bus, and Apache Flink for computation over the streams.


## Features & Highlights

  - **Exactly-once processing guarantees** for both Reader and Writer, supporting **end-to-end exactly-once processing pipelines**

  - Seamless integration with Flink's checkpoints and savepoints.

  - Parallel Readers and Writers supporting high throughput and low latency processing.

  - Table API support to access Pravega Streams for both **Batch** and **Streaming** use case.

## Building Connectors

Building the connectors from the source is only necessary when we want to use or contribute to the latest (*unreleased*) version of the Pravega Flink connectors.

The connector project is linked to a specific version of Pravega, based on a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules) pointing to a commit-id. By default the sub-module option is disabled and the build step will make use of the Pravega version defined in the `gradle.properties` file. You could override this option by enabling `usePravegaVersionSubModule` flag in `gradle.properties` to `true`.

Checkout the source code repository by following below steps:

```
git clone --recursive https://github.com/pravega/flink-connectors.git
```

After cloning the repository, the project can be built by running the below command in the project root directory `flink-connectors`.

```
./gradlew clean build
```

To install the artifacts in the local maven repository cache `~/.m2/repository`, run the following command:

```
./gradlew clean install
```

### Customizing the Build

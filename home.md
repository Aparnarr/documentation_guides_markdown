<!--
Copyright (c) 2017 Dell Inc., or its subsidiaries. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
-->
## Apache Flink Connectors for Pravega

This wiki describes the connectors API and it's usage to read and write [Pravega](http://pravega.io/) streams with [Apache Flink](http://flink.apache.org/) stream processing framework.

Build end-to-end stream processing pipelines that use Pravega as the stream storage and message bus, and Apache Flink for computation over the streams.   See the [Pravega Concepts](http://pravega.io/docs/pravega-concepts/) page for more information.

## Table of Contents

- [Quick Start](#quickstart)
- [Features](#features)
	- [Streaming](#streaming)
	- [Batch](#batch)
	- [Table API/SQL](#table-api)
	- [Serialization](#serialization)
	- [Metrics](#metrics)
- [How To Release](#how-to-release)
- [Publishing Artifacts](#publishing-artifacts)
- [Contributing](#contributing)
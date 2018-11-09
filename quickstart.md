<!--
Copyright (c) 2017 Dell Inc., or its subsidiaries. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
-->

## Creating a Flink stream processing project

*You can skip this step if you have a streaming project set up already.*

The easiest way to set up a stream processing project with Apache Flink is to use these project templates and setup guidelines:
  - [Project template for Java](https://ci.apache.org/projects/flink/flink-docs-stable/quickstart/java_api_quickstart.html)
  - [Project template for Scala](https://ci.apache.org/projects/flink/flink-docs-release-1.6/quickstart/scala_api_quickstart.html)

After that, please follow the section below to add the Flink Pravega connectors to the project.

## Add the connector dependencies

To add the Pravega connector dependencies to you project, add the following entry to your project file (for example `pom.xml` for Maven):

```
<dependency>
  <groupId>io.pravega</groupId>
  <artifactId>pravega-connectors-flink_2.11</artifactId>
  <version>0.3.2</version>
</dependency>
```

Use appropriate version as necessary. The snapshot versions are published to [jcenter](https://oss.jfrog.org/artifactory/jfrog-dependencies/io/pravega/pravega-connectors-flink_2.11/) repository and the release artifacts are available in [Maven Central](https://mvnrepository.com/artifact/io.pravega/pravega-connectors-flink_2.11) repository.

Alternatively, you could build and publish the connector project to local maven repository by following below steps and make use of that version as your application dependency.

```
./gradlew clean install
```

## Running / deploying the application

From Flink's perspective, the connector to Pravega is part of the streaming application (not part of Flink's core runtime), so the connector code must be part of the application's code artifact (JAR file).   Typically, a Flink application is bundled as a _fat-jar_ (also known as an _uber-jar_) such that all its dependencies are embedded.

If you have set up the project via the above linked templates/guides, everything should be set up properly.

If you set up a application's project and dependencies manually, you need to make sure that it builds a "jar with dependencies*, to include both the application and the connector classes.

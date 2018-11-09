<!--
Copyright (c) 2017 Dell Inc., or its subsidiaries. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
-->

Pravega/Flink connector artifacts are published in the following repositories.

1. `jcenter -> OJO` for snapshot artifacts

2. `Sonatype -> Maven Central` for release artifacts


Pravega/Flink connector artifacts are used by projects like `pravega-samples` which are used by external users. The `master` branch of the `pravega-samples` repositroy will always point to a stable release version of Pravega/Flink connector and hence should not be a concern. However, the development branch of `pravega-samples` (`develop` branch) are likely to be unstable due to the Pravega/Flink connector snapshot artifact dependency (since the development branch of Pravega/Flink connector could possibly introduce a breaking change).

A typical version label of a snapshot artifact will have the label `-SNAPSHOT` associated with it (for e.g., `0.1.0-SNAPSHOT`). There could be more revisions for a snapshot version and the maven/gradle build scripts takes care of fetching most recent version from the available list of revisions. The snapshot repositories are usually configured to discard the old revisions based on some retention policy settings. Any downstream projects that are depending on snapshots are faced with the challenge of keeping up with the snapshot changes since they could possibly introduce any breaking changes.

To overcome this issue, the Pravega/Flink connector snapshot artifacts are published to `jcenter` repository with a **stable** version label which can be referred by any downstream projects. This will guarantee to avoid any breaking changes from Pravega/Flink connector but at the same time, it is the responsibility of the downstream projects to sync with the latest shapshot updates.


# Publishing Snapshots

The snapshots are the artifacts that are coming from `master` branch (development branch). The snapshot artifacts are published automatically to `jcenter` through Travis build setup. Any updates to `master` branch will trigger a build that will publish the artifacts upon succesful completion of the build. 

We use `bintray` account to manage the artifact publishing. Here is the gradle task that is used to publish the artifacts. The published artifacts can be found here "https://oss.jfrog.org/jfrog-dependencies/io/pravega/pravega-connectors-flink_2.11/"

```
./gradlew clean assemble publishToRepo -PpublishUrl=jcenterSnapshot -PpublishUsername=<user> -PpublishPassword=<password>

```

The bintray credentials are encrypted using `travis encrypt` tool and are created for the namespace `pravega/pravega`. 

```
/usr/local/bin/travis encrypt BINTRAY_USER=<BINTRAY_USER> --adapter net-http

/usr/local/bin/travis encrypt BINTRAY_KEY=<BINTRAY_KEY> --adapter net-http

```

# Publishing Release Artifacts

We use `sonatype->Maven Central` repository to manage the release artifacts. Please follow [[How-to-release]] page to understand the complete steps require to release a Pravega/Flink connector version.

Here is the gradle task that is used to publish the artifacts. The published artifacts can be found here "https://mvnrepository.com/artifact/io.pravega"

```
./gradlew clean assemble publishToRepo -PdoSigning=true -Psigning.password=<signing-password> -PpublishUrl=mavenCentral -PpublishUsername=<sonatype-username> -PpublishPassword=<sonatype-password>
```
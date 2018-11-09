<!--
Copyright (c) 2017 Dell Inc., or its subsidiaries. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
-->

If you are releasing a version of Pravega/Flink Connector, then you must read and follow the instructions. The steps here are based on the experience we are building across releases, so if you see any point that requires changes or more detail, feel free to raise it or modify the document.

> If you are updating Flink version, please make sure any new features or changes introduced in Flink are addressed in the connector.

# Preparing the branch

Preparing the branch consists of making any necessary changes to the branch you will be working on as part of releasing. There are two possible situations:

* Bug-fix release: This is a minor release version over an existing release branch
* Feature release or non-backward-compatible release: This is a change to either the first or the middle digit, and it requires a new release branch.

## Bug-fix release

If this is a bug-fix release, then there is no need to create a new branch. First identify the branch you'll be working on. It will be named `rX.Y`, e.g., `r0.2`. The preparation consists of:
1. Changing `connectorVersion` in `gradle.properties` from `X.Y.Z-SNAPSHOT` to `X.Y.Z`. For example, if the current value is `connectorVersion=0.2.1-SNAPSHOT`, then change it to `connectorVersion=0.2.1`.
2. Merge this change
3. Tag the commit with `vX.Y.Z-rc0`. For example, `v0.2.1-rc0`.

A couple of observations about step 3:
1. There are two ways to tag:
    *  *Via the command line*:
    ```
       > git checkout rX.Y
       > git tag vX.Y.Z-rc0
       > git push upstream vX.Y.Z-rc0
    ```
    Make sure you have your `upstream` set up correctly
    
    *  *Via GitHub releases*: when creating a release candidate, GitHub automatically creates the tag for you in the case one does not exist yet. We will take more about release candidates in a minute.

2. It is possible that a release candidate is problematic and we need to do a new release candidate. In this case, we need to repeat this tagging step as many times as needed. Note that when creating a new release candidate tag, we do not need to update the Connector version.

## Major release

A major release changes either the middle or the most significant digit. In this case, you do need to create a new branch. The process is the following, assuming for the sake of example that the new release is `0.3.0`:

```
  > git checkout master
  > git tag branch-0.3
  > git push upstream branch-0.3
  > git checkout -b r0.3
  > git push upstream r0.3
```

Once the steps above are done, we need to make version changes to both `master` and `r0.3`:
*  In `master`, create an issue and corresponding pull request to change the `connectorVersion` in `gradle.properties` to `0.4.0-SNAPSHOT`. Note that we are bumping up the middle digit because, in our example, we are releasing `0.3.0`. If we were releasing say `1.0.0`, then we would change `connectorVersion` to `1.1.0-SNAPSHOT`.
* In `r0.3`, create an issue and corresponding pull request to change the `connectorVersion` in `gradle.properties` to `0.3.0`. Once that change is merged, we need to tag the commit point in the same way we described for a bug-fix release. See instructions in the previous section.

# Pushing a release candidate out

## Step 1: Create a release on GitHub

On the GitHub repository page, go to releases and create a new draft. On the draft:

* Mark it as a "pre-release". 
* Fill out the tag field and select the appropriate branch. Note that this is a release candidate, so the tag should look like `vX.Y.Z-rcA`

## Step 2: Build the distribution

Run the following commands, assuming for the sake of example that the branch we are working on is `r0.3`:

```
   > git checkout r0.3
   > ./gradlew clean install
```

The files resulting from the build will be under `~/.m2/repository/io/pravega/pravega-connectors-flink_2.11/<RELEASE-VERSION>`. For each one of the `.jar` files in that directory, generate checksums (currently `md5`, `sha1`, and `sha256`). It is easy to do it with a simple bash script along the lines of:

```
#!/bin/bash

for file in ./*.jar ; do md5sum $file > $file.md5 ; done
for file in ./*.jar ; do shasum -a 1 $file > $file.sha1 ; done
for file in ./*.jar ; do shasum -a 256 $file > $file.sha256 ; done
```

*Note*: In the future, we might want to automate the generation of checksums via gradle.

## Step 3: Upload the files to the pre-release draft

In the pre-release draft on GitHub, upload all the jar files (and its checksums) under `~/.m2/repository/io/pravega/pravega-connectors-flink_2.11/<RELEASE-VERSION>`. Follow the instructions on the page, it is straightforward.

## Step 4: Release notes

Create a release notes text file containing the following:
1. Some introductory text, highlighting the important changes going in the release.
2. A full list of commits that you can get with the following command:
```
   > git log <commit-id-of-last-release>..<current-commit-id>
```

* `<commit-id-of-last-release>` depends on the kind of release you are doing. If it is bug-fix release, then we can use the tag of the last branch release. For new branches, we have been adding `branch-X.Y` tags at the branch point for convenience. 
* `<current-commit-id>` is the commit point of the release candidate you are working on. If you have manually tagged the release candidate, then you can go ahead and use it in the log command above.

Add the list to the release notes file and attach it the notes box in the release draft. See previous releases for an example of how to put together notes.

## Step 5: Publishing

Once this is all done, publish the release candidate by clicking on the button on the draft page. Once published, request the developers and community to validate the candidate.

# Releasing

Once you are happy with the release candidate, we can start the release process. There are two main parts for a Connector release:

1. Releasing on GitHub
2. Publishing on Sonatype -> Maven Central

## Releasing on GitHub

The process to do this is pretty much the same as the one of creating a release candidate. The only two differences are:
1. The tag should not have an `rcA` in it. If the successful rc is `v0.3.0-rc0`, then the release tag is `v0.3.0`.
2. Do not check the pre-release box.

## Publishing on Sonatype -> Maven Central

For this step, you need a Sonatype account. See this [guide](http://central.sonatype.org/pages/ossrh-guide.html) for how to create an account. Your account also needs to be associated to Pravega to be able to publish the artifacts. 

Once you are ready, run the following steps:
* Build with the following command:
```
./gradlew clean assemble publishToRepo -PdoSigning=true -Psigning.password=<signing-password> -PpublishUrl=mavenCentral -PpublishUsername=<sonatype-username> -PpublishPassword=<sonatype-password>

```
* Login to Nexus Repository Manager using sonatype credentials with write access to io.pravega group.
* Under Build Promotion -> Staging Repositories, locate the staging repository that was created for the latest publish (format iopravega-XXXX, for example iopravega-1004)
* Select the repository and select the Close button in the top menu bar. This will perform validations to ensure that the contents meets the maven requirements (contains signatures, javadocs, sources, etc). This operation takes a short time to complete, press the Refresh button in the top menu bar occasionally until the operation completes.
* Once the operation completes, locate the URL field in the Summary tab of the newly closed repository (it will be something like https://oss.sonatype.org/content/repositories/iopravega-XXXX where XXXX is the number of the staging repository). This should be tested to ensure that all artifacts are present and functions as expected.
* To test, use for example, the `pravega-samples` and checkout `develop` branch to verify that it can locate and build with the staging artifacts. Concretely:
    1. Change `pravegaVersion` and `connectorVersion` in `gradle.properties` to the staging version
    2. Run `./gradlew clean build`
* When satisfied that everything is working, you can select the Release button in the top menu bar.
* Wait until it shows up in Maven Central, it takes some [time](http://central.sonatype.org/pages/ossrh-guide.html#SonatypeOSSMavenRepositoryUsageGuide-9.ActivateCentralSync).

## Change the Connector version.

Once the release is done, create an issue and corresponding pull request to change the `connectorVersion` in `gradle.properties` to `X.Y.(Z+1)-SNAPSHOT` for the release branch.

# Done!
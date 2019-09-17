---
description: In environments without Docker
---

# Manual Deployment

The installation should work under Windows, Linux and MacOS. Both Datomic and Blaze need a current Java 8 JRE. Other Versions of Java aren’t currently tested.

## Datomic

Blaze needs a [Datomic](https://www.datomic.com/) database to store FHIR® resources. Different editions of Datomic are available. Four our use case, the Free Edition is sufficient.

You can download Datomic Free here: [https://my.datomic.com/downloads/free](https://my.datomic.com/downloads/free)

Please download Version 0.9.5703 and unpack the ZIP into a directory of your choice. After that please follow the steps:

* open a terminal and change into the `datomic-free-0.9.5703` dir
* copy the sample properties file located at \(relative\) `config/samples/free-transactor-template.properties` to `config/transactor.properties`
* start the database by calling `bin/transactor config/transactor.properties`

The output should look like this:

```text
Launching with Java options -server -Xms1g -Xmx1g -XX:+UseG1GC -XX:MaxGCPauseMillis=50
Starting datomic:free://localhost:4334/<DB-NAME>, storing data in: data ...
System started datomic:free://localhost:4334/<DB-NAME>, storing data in: data
```

You can change the data dir in the properties file if you like to have it at a different location.

## Blaze

Blaze runs on the JVM and comes as single JAR file. Download the most recent version [here](https://github.com/life-research/blaze/releases/tag/v0.6.4). Look for `blaze-0.6.4-standalone.jar`. Blaze requires at least Java 8 and is also tested with Java 11.

After the download, you can start blaze with the following command \(Linux, MacOS\):

```text
DATABASE_URI=datomic:free://localhost:4334/db java -jar blaze-0.6.4-standalone.jar
```

Under Windows you need to set the Environment variables in the PowerShell before starting Blaze:

```text
$Env:DATABASE_URI = "datomic:free://localhost:4334/db"
java -jar blaze-0.6.4-standalone.jar
```

The output should look like this:

```text
WARNING: requiring-resolve already refers to: #'clojure.core/requiring-resolve in namespace: datomic.common, being replaced by: #'datomic.common/requiring-resolve
19-08-28 17:22:35 3962d91052cd INFO [blaze.system:232] - Read structure definitions resulting in: 190 structure definitions
19-08-28 17:22:37 3962d91052cd INFO [blaze.system:247] - Created database at: datomic:free://localhost:4334/db
19-08-28 17:22:37 3962d91052cd INFO [blaze.system:250] - Connect with database: datomic:free://localhost:4334/db
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:241] - Upsert schema in database: datomic:free://localhost:4334/db creating 92290 new facts
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:259] - Init terminology server connection: http://tx.fhir.org/r4
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:409] - Start metrics server on port 8081
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:370] - Start main server on port 8080
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:225] - Set log level to: info
19-08-28 17:22:44 3962d91052cd INFO [blaze.core:49] - JVM version: 1.8.0_222
19-08-28 17:22:44 3962d91052cd INFO [blaze.core:50] - Maximum available memory: 444 MiB
19-08-28 17:22:44 3962d91052cd INFO [blaze.core:51] - Number of available processors: 2
```

Blaze will be configured through environment variables which are documented here:

{% page-ref page="environment-variables.md" %}


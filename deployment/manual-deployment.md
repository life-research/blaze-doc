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
19-09-17 16:14:14 <host> INFO [blaze.system:217] - Set log level to: info
19-09-17 16:14:14 <host> INFO [blaze.system:232] - Read structure definitions resulting in: 190 structure definitions
19-09-17 16:14:15 <host> INFO [blaze.system:250] - Use existing database at: datomic:free://localhost:4334/dev?password=datomic
19-09-17 16:14:15 <host> INFO [blaze.system:252] - Connect with database: datomic:free://localhost:4334/dev?password=datomic
19-09-17 16:14:20 <host> INFO [blaze.system:259] - Init terminology server connection: http://tx.fhir.org/r4
19-09-17 16:14:20 <host> INFO [blaze.system:367] - Init FHIR RESTful API with base URL: http://localhost:8080/fhir
19-09-17 16:14:20 <host> INFO [blaze.system:438] - Start metrics server on port 8081
19-09-17 16:14:20 <host> INFO [blaze.system:395] - Start main server on port 8080
19-09-17 16:14:20 <host> INFO [blaze.core:49] - JVM version: 1.8.0_221
19-09-17 16:14:20 <host> INFO [blaze.core:50] - Maximum available memory: 3641 MiB
19-09-17 16:14:20 <host> INFO [blaze.core:51] - Number of available processors: 8
```

In order to test connectivity, query the health endpoint:

```text
curl http://localhost:8080/health
```

After that please note that the [FHIR RESTful API](https://www.hl7.org/fhir/http.html) is available under `http://localhost:8080/fhir`. A good start is to query the [CapabilityStatement](https://www.hl7.org/fhir/capabilitystatement.html) of Blaze using [jq](https://stedolan.github.io/jq/) to select only the software key of the JSON output:

```text
curl -H 'Accept:application/fhir+json' -s http://localhost:8080/fhir/metadata | jq .software
```

that should return:

```javascript
{
  "name": "Blaze",
  "version": "0.6.4"
}
```

Blaze will be configured through environment variables which are documented here:

{% page-ref page="environment-variables.md" %}


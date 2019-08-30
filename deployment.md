# Deployment

## Quick Start

In order to run Blaze with an in-memory, volatile database, just execute the following:

### Docker

```
docker run -p 8080:8080 liferesearch/blaze:0.6
```

### Java

```text
wget https://github.com/life-research/blaze/releases/download/v0.6/blaze-0.6-standalone.jar
```

```text
java -jar blaze-0.6-standalone.jar
```

Logging output should appear which prints the most important settings and system parameters like Java version and available memory.

In order to test connectivity, query the health endpoint:

```text
curl http://localhost:8080/health
```

## Production Deployment

Blaze comes as a web application which needs to access a Datomic database. The most basic production deployment uses a Datomic Free Edition database and a single Blaze instance.

### Docker

#### Network

It's best to create a separate Docker network for the communication between Blaze and Datomic.

```text
docker network create blaze
```

#### Datomic Free

A Docker run command for a Datomic Free instance would look like this. Note that there is no volume defined for the actual data. It will be lost on container removal.

```text
docker run -d --name db --network blaze -e ALT_HOST=db \
  -e ADMIN_PASSWORD=admin -e DATOMIC_PASSWORD=datomic \
  akiel/datomic-free:0.9.5703-2 
```

#### Blaze

The environment variable `DATABASE_URI` tells Blaze to use the started Datomic database. 

```text
docker run -d --name blaze --network blaze \
  -e DATABASE_URI=datomic:free://db:4334/db?password=datomic \
  -p 8080:8080 liferesearch/blaze:0.6
```

Blaze should log something like this:

```text
WARNING: requiring-resolve already refers to: #'clojure.core/requiring-resolve in namespace: datomic.common, being replaced by: #'datomic.common/requiring-resolve
19-08-28 17:22:35 3962d91052cd INFO [blaze.system:232] - Read structure definitions resulting in: 190 structure definitions
19-08-28 17:22:37 3962d91052cd INFO [blaze.system:247] - Created database at: datomic:free://db:4334/db?password=datomic
19-08-28 17:22:37 3962d91052cd INFO [blaze.system:250] - Connect with database: datomic:free://db:4334/db?password=datomic
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:241] - Upsert schema in database: datomic:free://db:4334/db?password=datomic creating 92290 new facts
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:259] - Init terminology server connection: http://tx.fhir.org/r4
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:409] - Start metrics server on port 8081
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:370] - Start main server on port 8080
19-08-28 17:22:44 3962d91052cd INFO [blaze.system:225] - Set log level to: info
19-08-28 17:22:44 3962d91052cd INFO [blaze.core:49] - JVM version: 1.8.0_222
19-08-28 17:22:44 3962d91052cd INFO [blaze.core:50] - Maximum available memory: 444 MiB
19-08-28 17:22:44 3962d91052cd INFO [blaze.core:51] - Number of available processors: 2
```

#### Docker Compose

A Docker Compose file with adds a volume for Datomic looks like this:

```text
version: '3.2'
services:
  db:
    image: "akiel/datomic-free:0.9.5703-2"
    environment:
      ALT_HOST: "db"
      ADMIN_PASSWORD: "admin"
      DATOMIC_PASSWORD: "datomic"
    volumes:
    - "db-data:/data"
  store:
    image: "liferesearch/blaze:0.6"
    environment:
      DATABASE_URI: "datomic:free://db:4334/dev?password=datomic"
    ports:
    - "8080:8080"
    depends_on:
    - db
volumes:
  db-data:
```

### Manual

The installation should work under Windows, Linux and MacOS. Both Datomic and Blaze need a current Java 8 JRE. Other Versions of Java aren’t currently tested.

#### Datomic

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

#### Blaze

Blaze runs on the JVM and comes as single JAR file. Download the most recent version [here](https://github.com/life-research/blaze/releases/tag/v0.6). Look for `blaze-0.6-standalone.jar`.

After the download, you can start blaze with the following command \(Linux, MacOS\):

```text
DATABASE_URI=datomic:free://localhost:4334/db java -jar blaze-0.6-standalone.jar
```

Under Windows you need to set the Environment variables in the PowerShell before starting Blaze:

```text
$Env:DATABASE_URI = "datomic:free://localhost:4334/db"
java -jar blaze-0.6-standalone.jar
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


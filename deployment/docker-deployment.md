---
description: In Docker environments like Docker Compose or Kubernetes
---

# Docker Deployment

Blaze comes as a web application which needs to access a Datomic database. The most basic production deployment uses a Datomic Free Edition database and a single Blaze instance.

## Network

It's best to create a separate Docker network for the communication between Blaze and Datomic.

```text
docker network create blaze
```

## Datomic Free

A Docker run command for a Datomic Free instance would look like this. Note that there is no volume defined for the actual data. It will be lost on container removal.

```text
docker run -d --name db --network blaze -e ALT_HOST=db \
  -e ADMIN_PASSWORD=admin -e DATOMIC_PASSWORD=datomic \
  akiel/datomic-free:0.9.5703-2
```

## Blaze

The environment variable `DATABASE_URI` tells Blaze to use the started Datomic database.

```text
docker run -d --name blaze --network blaze \
  -e DATABASE_URI=datomic:free://db:4334/db?password=datomic \
  -p 8080:8080 liferesearch/blaze:0.7.0-rc.1
```

Blaze should log something like this:

```text
19-12-16 11:49:18 ce3fb21de517 INFO [blaze.system:165] - Set log level to: info
19-12-16 11:49:18 ce3fb21de517 INFO [blaze.system:43] - Try to read blaze.edn ...
19-12-16 11:49:18 ce3fb21de517 INFO [blaze.system:152] - Feature OpenID Authentication disabled
19-12-16 11:49:18 ce3fb21de517 INFO [blaze.system:74] - Loading namespaces ...
WARNING: requiring-resolve already refers to: #'clojure.core/requiring-resolve in namespace: datomic.common, being replaced by: #'datomic.common/requiring-resolve
19-12-16 11:49:32 ce3fb21de517 INFO [blaze.system:76] - Loaded the following namespaces: blaze.datomic.transaction, blaze.structure-definition, blaze.datomic, blaze.fhir.operation.evaluate-measure, blaze.handler.health, blaze.interaction.history.instance, blaze.interaction.history.system, blaze.interaction.history.type, blaze.interaction.transaction, blaze.interaction.create, blaze.interaction.delete, blaze.interaction.read, blaze.interaction.search-type, blaze.interaction.update, blaze.rest-api, blaze.handler.app, blaze.metrics, blaze.handler.metrics, blaze.server, blaze.thread-pool-executor-collector, blaze.terminology-service.extern, blaze.terminology-service
19-12-16 11:49:33 ce3fb21de517 INFO [blaze.structure-definition:140] - Read structure definitions resulting in: 190 structure definitions
19-12-16 11:49:33 ce3fb21de517 INFO [blaze.datomic:25] - Created database at: datomic:mem://dev
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.datomic:18] - Upsert schema in database: datomic:mem://dev creating 92290 new facts
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.datomic:29] - Connect with database: datomic:mem://dev
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.fhir.operation.evaluate-measure:69] - Init FHIR $evaluate-measure operation executor
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.terminology-service.extern:184] - Init terminology server connection: http://tx.fhir.org/r4
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.fhir.operation.evaluate-measure:63] - Init FHIR $evaluate-measure operation handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.handler.health:21] - Init health handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.history.instance:108] - Init FHIR history instance interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.history.system:122] - Init FHIR history system interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.history.type:125] - Init FHIR history type interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.transaction:396] - Init FHIR transaction interaction executor
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.transaction:390] - Init FHIR transaction interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.create:82] - Init FHIR create interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.delete:62] - Init FHIR delete interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.read:98] - Init FHIR read interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.search-type:126] - Init FHIR search-type interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.interaction.update:113] - Init FHIR update interaction handler
19-12-16 11:49:36 ce3fb21de517 INFO [blaze.rest-api:385] - Init FHIR RESTful API with base URL: http://localhost:8080/fhir
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.handler.app:35] - Init app handler
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.system:199] - Init server executor
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.thread-pool-executor-collector:70] - Init thread pool executor collector.
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.metrics:27] - Init metrics registry
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.handler.metrics:29] - Init metrics handler
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.system:220] - Start metrics server on port 8081
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.system:208] - Start main server on port 8080
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.core:58] - JVM version: 11.0.4
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.core:59] - Maximum available memory: 1490 MiB
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.core:60] - Number of available processors: 4
19-12-16 11:49:37 ce3fb21de517 INFO [blaze.core:61] - Successfully started Blaze version 0.7.0-rc.1 in 19.4 seconds
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
  "version": "0.7.0-rc.1"
}
```

Blaze will be configured through environment variables which are documented here:

{% page-ref page="environment-variables.md" %}

## Docker Compose

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
    image: "liferesearch/blaze:0.7.0-rc.1"
    environment:
      DATABASE_URI: "datomic:free://db:4334/dev?password=datomic"
    ports:
    - "8080:8080"
    depends_on:
    - db
volumes:
  db-data:
```


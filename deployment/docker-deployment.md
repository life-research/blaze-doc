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
  -p 8080:8080 liferesearch/blaze:0.6.4
```

Blaze should log something like this:

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

```json
{
  "name": "Blaze",
  "version": "0.6.4"
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
    image: "liferesearch/blaze:0.6.4"
    environment:
      DATABASE_URI: "datomic:free://db:4334/dev?password=datomic"
    ports:
    - "8080:8080"
    depends_on:
    - db
volumes:
  db-data:
```


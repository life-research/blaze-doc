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
  -p 8080:8080 liferesearch/blaze:0.6.2
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
    image: "liferesearch/blaze:0.6.2"
    environment:
      DATABASE_URI: "datomic:free://db:4334/dev?password=datomic"
    ports:
    - "8080:8080"
    depends_on:
    - db
volumes:
  db-data:
```


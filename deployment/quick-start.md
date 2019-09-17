---
description: Fast non-production Deployment
---

# Quick Start

In order to run Blaze with an in-memory, volatile database, just execute the following:

## Docker

```text
docker run -p 8080:8080 liferesearch/blaze:0.6.4
```

## Java

Blaze requires at least Java 8 and is also tested with Java 11.

```text
wget https://github.com/life-research/blaze/releases/download/v0.6.4/blaze-0.6.4-standalone.jar
java -jar blaze-0.6.4-standalone.jar
```

Logging output should appear which prints the most important settings and system parameters like Java version and available memory.

In order to test connectivity, query the health endpoint:

```text
curl http://localhost:8080/health
```

Blaze will be configured through environment variables which are documented here:

{% page-ref page="environment-variables.md" %}


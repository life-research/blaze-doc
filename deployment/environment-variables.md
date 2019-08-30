# Environment Variables

Blaze is configured solely through environment variables. There is a default for every variable. So all are optional.

The following table constains all of them:

| Name | Default | Description |
| :--- | :--- | :--- |
| DATABASE\_URI | datomic:mem://dev | URI of the Datomic database In-memory, volatile by default  |
| PROXY\_HOST | — | Proxy server for outbound HTTP requests |
| PROXY\_PORT | — | Port of the proxy server |
| TERM\_SERVICE\_URI | [http://tx.fhir.org/r4](http://tx.fhir.org/r4) | Base URI of the terminology service |
| BASE\_URL | [http://localhost:8080](http://localhost:8080) | The URL under which Blaze is accessible by clients |
| SERVER\_PORT | 8080 | The port of the main HTTP server |
| METRICS\_SERVER\_PORT | 8081 | The port of the Prometheus metrics server |
| LOG\_LEVEL | info | one of trace, debug, info, warn or error |
| JVM\_OPTS | — | JVM options \(Docker only\) |


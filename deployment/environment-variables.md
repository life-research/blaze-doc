# Environment Variables

Blaze is configured solely through environment variables. There is a default for every variable. So all are optional.

The following table contains all of them:

| Name | Default | Since | Description |
| :--- | :--- | :--- | :--- |
| DATABASE\_URI | datomic:mem://dev |  | URI of the Datomic database In-memory, volatile by default |
| PROXY\_HOST | — | v0.6 | The hostname of the proxy server for outbound HTTP requests |
| PROXY\_PORT | — | v0.6 | Port of the proxy server |
| PROXY\_USER | — | v0.6.1 | Proxy server user, if authentication is needed. |
| PROXY\_PASSWORD | — | v0.6.1 | Proxy server password, if authentication is needed. |
| CONNECTION\_TIMEOUT | 5 s | v0.6.3 | connection timeout for outbound HTTP requests |
| REQUEST\_TIMEOUT | 30 s | v0.6.3 | request timeout for outbound HTTP requests |
| TERM\_SERVICE\_URI | [http://tx.fhir.org/r4](http://tx.fhir.org/r4) | v0.6 | Base URI of the terminology service |
| BASE\_URL | [http://localhost:8080](http://localhost:8080) |  | The URL under which Blaze is accessible by clients. The [FHIR RESTful API](https://www.hl7.org/fhir/http.html) will be accessible under `BASE_URL/fhir`. |
| SERVER\_PORT | 8080 |  | The port of the main HTTP server |
| METRICS\_SERVER\_PORT | 8081 | v0.6 | The port of the Prometheus metrics server |
| LOG\_LEVEL | info | v0.6 | one of trace, debug, info, warn or error |
| JVM\_OPTS | — |  | JVM options \(Docker only\) |


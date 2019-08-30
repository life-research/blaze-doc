# Implementation

## Threading Model

Blaze uses the [Aleph HTTP server](https://aleph.io/aleph/http.html) which is able to process requests asynchronously. Aleph doesn't need one thread per request processing. Instead it uses a low number of threads to handle I/O for multiple requests buffering HTTP headers and bodies. Once a request is fully received, a worker thread is used to calculate the response. In case the response calculation needs to perform I/O itself, like connecting to the Datomic transactor for writes, it uses [Manifold Deferreds](https://aleph.io/manifold/deferreds.html) to do that asynchronously as well. Doing so makes all other request processing CPU bound. For that reason Blaze uses a single [work-stealing thread pool](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#newWorkStealingPool--) with the parallelism level set to the number of available cores for all it's request processing.

For transactions, Blaze has to do I/O. The transaction data is send to the Datomic transactor and Blaze has to wait for the answer. In order to maximize the throughput of the transactor, Blaze uses a parallelism level of 20 while sending data to it. Again a work-stealing thread pool is used to achieve and maintain that parallelism level.

Currently the queues in front of both thread pools are unbounded. Future work should focus on ways to introduce timeouts or bounded queues.

## Database Design

There exists a separate page for implementation considerations regarding the database design.

{% page-ref page="database-design.md" %}

## History Paging

For histories, the resources are taken from transactions using the `:tx/resources` attribute. The first resource of a page can be adressed by a combination of transaction and the resource itself because a resource can't be changed two times within one transaction. The transaction will be addressed by its `t` and the resource by its `eid`. The query params are `page-t` and `page-eid`, although they aren't public. On top of that a query param `t` is also used in order to mark the database for `Bundle.total`.

#### 

#### 



#### 


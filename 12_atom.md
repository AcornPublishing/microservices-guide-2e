# 12 Recipe: Asynchronous Communication with Atom and REST {#chapter-atom}

This chapter deals with the integration of microservices based on the
Atom data format. An
[example](https://github.com/ewolff/microservice-atom)
is provided which uses a simple scenario to illustrate what an
integration with the help of Atom and REST can look like.

The readers learn:

* What the Atom format is, how it functions, and how it can be
exploited for the asynchronous communication of microservices.

* Which alternatives to Atom exist for the asynchronous communication
via REST/HTTP and which advantages and disadvantages the different
formats have.

* How HTTP and REST can be efficiently used, not only for asynchronous
communication.

* What an implementation of an asynchronous system based on HTTP, REST
and Atom can look like.

## 12.1 The Atom Format {#section-atom-protokoll}

Atom is a data format originally developed to make blogs accessible to
readers. An Atom document contains blog posts that subscribers can
read in a chronological order. Besides blogs, Atom can also be used
for podcasts. Because the protocol is so flexible, it is also suitable
for other types of data. For example, a microservice can offer
information about new orders to other microservices as an Atom feed.
This corresponds to the REST approach, which uses established web
protocols such as HTTP for the integration of applications.

[Atom](https://validator.w3.org/feed/docs/atom.html) is no protocol,
but just a data format. A GET request to a new URL such as
<http://innoq.com/order/feed> can return a document with orders in the
Atom format. This document can contain links to the details of the
orders.

#### MIME Type

HTTP based communication indicates the type of content with the help
of MIME types (Multipurpose Internet Mail Extensions). The MIME type
for Atom is `application/atom+xml`. Thanks to content negotiation, also
other data representation can be offered in addition to Atom for the
same URL. Content
negotiation is built into HTTP. The HTTP client signals via an accept
header which data formats it can process. The server will then send a
response in a suitable format. Thus, accept headers enable a client to
request all orders as JSON or the last changes as Atom feed under the
same URL.

#### Feed

{linenos=off, lang="xml"}
~~~~~~~~
<?xml version="1.0" encoding="UTF 8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title>Order</title>
  <link rel="self" href="http://localhost:8080/feed" />
  <author>
    <name>Big Money Online Commerce Inc.</name>
  </author>
  <subtitle>List of all orders</subtitle>
  <id>tag:ewolff.com/microservice-atom/order</id>
  <updated>2017 04 20T15:28:50Z</updated>
  <entry>
  ...
  </entry>
</feed>
~~~~~~~~

A document in Atom format is called Atom feed. An Atom feed contains
different components. First, the feed defines meta data. The Atom
standard defines that this meta
data has to comprise a number of elements.

* *id* is a globally unambiguous and permanent URI (Uniform Resource
Identifier). It has to identify
the feed. In the listing this is done via a
  [tag URI](https://en.wikipedia.org/wiki/Tag_URI_scheme).
  
* *title* is the title of a feed in a form that is human-readable.
  
* *updated* contains the point in time when a feed has been changed
the last time. This information is important for finding out whether
there is new data.

* There also has to be an *author* (including *name*, *email*
and *uri*). This is of course very helpful for blogs. However, it does
not seem to make sense for Atom feeds, which
only transport data. However, it can be useful for indicating a
contact persons in case of problems.

* Multiple *link* elements are recommended. Each element
  has an attribute called *rel* for indicating the relationship
  between the feed and the link. The attribute *href* contains the
actual link. A *link* can be used to provide
a link to a HTML representation of the data. In addition, the feed can
use a *self* link to indicate at which URL it is available.
  
There is additional optional meta data.

* *category* narrows down the topic area of the feed, for example, to
a field like sports. Of course, this does not make a lot of sense for
data.

* Analogous to *author*, *contributor* contains information about
people who contribute to the feed.
  
* *generator* indicates the software which produced the feed. 
  
* *icon* is a small icon, while *logo* represents a larger icon. This
makes it possible to represent a blog or podcast on the desktop. It is
not so
useful
for a data feed.
  
* *rights* defines for instance the copyright. This element is
likewise mostly interesting for blogs or podcasts.
  
* Finally, *subtitle* is a human-readable description of the feed. 

As the listing illustrates, the fields *id*, *title* and *updated* are
enough
for an Atom feed with data. Having a *subtitle* for documentation is
helpful. There should also be a *link* for documenting the URL of the
feed.
All other elements are not really needed.

#### Entry

In the example above the most important thing is missing, the feed
entries. Such an entry adheres to the following format.

{linenos=off, lang="xml"}
~~~~~~~~
<entry>
  <title>Order 1</title>
  <id>tag:ewolff.com/microservice-atom/order/1</id>
  <updated>2017 04 20T15:27:58Z</updated>
  <content type="application/json"
    src="http://localhost:8080/order/1" />
  <summary>This is the order 1</summary>
</entry>
~~~~~~~~

An entry contained in the feed must comprise the following data
according to the Atom standard.

* *id* is the globally unambiguous ID as URI. Thus, there cannot be a
second entry with the id
  `tag:ewolff.com/microservice-atom/order/1` in this feed.
  This URI is a
  [tag URI](https://en.wikipedia.org/wiki/Tag_URI_scheme) which is
  meant for such globally unambiguous identifiers.

* *title* is a human-readable title for the entry.

* *updated* is the timepoint when the entry has been changed the last
time. It has to be set so that the client can decide whether it
already knows the last state of a certain entry.

In
addition, the following elements are recommended:

* As described for the feed, *author* names the author of the entry.

* *link* can contain links, for example, *alternate* as link to the
actual entry.

* *summary* is a summary of the content. It has to be provided if the
content is only accessible via a link or if it does not have an XML
media type. This is the case in the example.

* *content* can be the complete content of the entries or a link to
the actual content. To enable access to the data of the entry,
  either *content* has to be offered or a *link* with *alternate*.

* In addition, *category* and *contributor* are optional, analogous to
the respective elements of the feed. *published* can indicate the first
date of publication. The element *rights* can state the
rights. *source* can name the original source if the entry was a copy
from another feed.

In the example, the elements *id*, *title*, *summary* and *updated* are
filled. Access to the data is possible via a link in *content*. The
data could also directly be contained in the *content* element in the
document. However, in that case the document would be very large.
Thanks to the links, the document remains small. Each client has to
access the additional data via links and can limit itself to the data
which are really relevant to the respective client.

#### Tools

At <https://validator.w3.org/feed/#validate_by_input> a validator is
provided which can check whether an Atom feed is valid i.e. all
necessary elements are present.

There are systems like [Atom Hopper](http://atomhopper.org/) which
offer a server with the Atom format. In this way, an application does
not have to generate Atom data, but can post new data to the
server. The server then converts the information into Atom. Clients
can fetch the Atom data from the server.

#### Efficient Polling of Atom Feeds

Asynchronous communication with Atom requires that the client
regularly requests the data from the server and processes new data.
This is called polling.

The client can periodically read out the feed and check the *updated*
field in the feed to see if the data has changed. If this is the case,
the client can use the *updated* elements of the entries to find out
which entries are new and process them.

This is very time-consuming because the entire feed has to be
generated and transmitted. Finally, only a few entries are read from
the feed. Most requests will show that there are actually no new
entries. For
this purpose, it does not make sense to create and transfer the
complete feed.

#### HTTP Caching

A very simple way to solve this problem is HTTP caching (see
[figure 12-1](#fig-atom-effizientes-polling)).

HTTP provides a header with the name *Last-Modified* in
the HTTP response. This header indicates when the data was last
changed. This header takes over the function of the *updated* field
from the feed.

{id="fig-atom-effizientes-polling"}
![Fig. 12-1: Efficient Pooling with HTTP Caching](images/atom-effizientes-polling.png)

The client stores the value of this header. On the next HTTP GET
request, the client sends the read value in the *If-Modified-Since*
header with the GET request. The server can now respond with an HTTP
status of 304 (not modified) if the data has not changed. Then no data is
transferred except the status code.

Whether data has changed can often be determined very easily. For
example, you can implement code that determines the time of the last
change in the database. This is much more efficient than converting
all data into an
Atom representation.

If the data has actually changed, the server responds normally
with a status of 200 (OK). Also a new value is sent in
the *Last-Modified* header so that the client can use HTTP caching
again.

The demo implements such an approach. For a request with a set
*If-Modified-Since* header, the database is used to determine the time
of the last change. This is compared with the value from the header.
An HTTP status 304 is returned if no data has changed. In this
case, only one database query is required to respond to the HTTP
request.

An alternative would be to create the Atom feed once and store it on a
web server as a static resource. In this case, dynamic generation is
only performed once.  This approach also works efficiently for very
large feeds.
In that case it would be up to the web server to implement the HTTP
caching for the static resource.

#### ETags

Another approach would be HTTP caching with
[ETags](https://en.wikipedia.org/wiki/HTTP_ETag).  Here, the server
returns an ETag together with the data. The ETag can be compared to a
version number or checksum.  For any further requests, the client
sends the ETag along. The server uses the ETag to determine whether
the data has changed in the meantime and only provides data if new
data is available.  In the example, however, it is much easier to find
out whether new orders have been accepted since a certain point in
time.
So it does not make sense to use ETags in the example.

#### Pagination and Filtering

If a client is not interested in all events, it can signal this by
setting parameters in the URL. This makes it possible to paginate, for
example with a URL like <http://ewolff.com/order?from=23&to=42>. The
events can be filtered as well:
<http://ewolff.com/order?name=Wolff>. Of course, pagination and
filtering can be combined with caching. However, if each client uses
its own pagination and filters, caching on the server side can become
inefficient because too much different data has to be stored for the
different
clients.  Therefore, it may be necessary to restrict the pagination
and filtering options in order to increase the efficiency of the
cache.

#### Push vs. Pull

HTTP optimizations such as conditional GETs can significantly speed up
communication. But it remains a pull mechanism. The client queries the
server, whereas in case of a push mechanism the client is actively
notified by the server about changes. The push model seems to be more
efficient. But pulls have the advantage that the client requests new
data when it can actually process it. This prevents the client from
handling requests if it is under too much load.

#### Guaranteed Delivery

Atom via HTTP cannot guarantee the delivery of the data. The server
only provides the data. Whether it will be read at all is an open
question. Monitoring can help to identify problems and remedy them if
necessary.  So it would be very unusual if the Atom resources are
never needed.  However, the HTTP protocol does not have any measures
for issuing receipt acknowledgements.

#### Old Events

In principle, an Atom feed can contain all events that have ever
occurred.  As mentioned in [section 10.2](#section-asynchron-events),
this might be interesting for event sourcing.  Then a microservice can
reconstruct its state by processing all old events again.

If the data on old events is already stored in a microservice
anyway, the microservice only needs to make it available. For example,
if a service already contains all orders, it can offer this
information additionally as an Atom feed if necessary. In this case,
no additional storage of old events is necessary.
Thus, the effort to make the old events accessible is
very low.

## 12.2 Example {#section-atom-beispiel}

The example can be found at <https://github.com/ewolff/microservice-atom>.

The example for Atom is analogous to the example in the Kafka chapter
([section 11.4](#section-kafka-beispiel)) and is based on the example
for events from [section 10.2](#section-asynchron-events).  Orders are
created by the order system. Based on the data in the order, the
invoicing microservice creates invoices and the shipping microservice
deliveries. The data models and database schemas are identical to the
Kafka example. Only the communication is now done via Atom.

{id="fig-atom-ueberblick"}
![Fig. 12-2: Overview of the Atom System](images/atom-ueberblick.png)

[Figure 12-2](#fig-atom-ueberblick) shows how the example is
structured:

* The *Apache httpd* distributes calls to the microservices. For this
  purpose, the Apache httpd uses Docker compose service links. Docker
  compose offers simple load balancing. The Apache httpd uses the load
  balancing of Docker compose to forward external calls to one of the
  microservice instances.

* The *order microservice* offers an Atom feed from which the
  invoicing and shipping microservice can read the information about
  new orders.

* All microservices use the same *Postgres database*. Within the
  database, each microservice has its own separate database
  schema. Thus, the microservices are completely independent regarding the
  database schema. At the same time, one database instance is
  enough to run all microservices.

[Section 0.4](#section-einleitung-quick-start) describes which
software needs to be installed to start the example. The example can
be found at <https://github.com/ewolff/microservice-atom>.  You first
have to download the code with `git clone
https://github.com/ewolff/microservice-atom.git` to start the example.
Then you have to execute the command `./mvnw clean package` (macOS,
Linux) or `mvnw.cmd clean package` (Windows) in the directory
`microservice-atom`.
See [appendix B](#anhang-maven) for more details on Maven and how to
trouble shoot the build.
Finally, you have to execute `docker-compose
build` in the directory `docker` to create the Docker images and
`docker-compose up -d` to start the environment.
See [appendix C](#anhang-docker) for more details on Docker,
docker-compose and how to troubleshoot them.
Then the Apache httpd load
balancer will be available at port 8080, i.e. for instance at
<http://localhost:8080/>. From there you can use the other
microservices to add new orders that will eventually also appear in
the invoicing and shipping microservice.

<https://github.com/ewolff/microservice-atom/blob/master/HOW-TO-RUN.md>
describes the different steps in
detail which are needed to run the example.

#### Implementation of the Atom View

The class *OrderAtomFeedView* in the project *microservice-atom-order*
implements the Atom feed as a view with the framework Spring MVC.
Spring MVC splits the system into MVC (model, view, controller). The
*controller* contains the logic, the *model* the data, and the *view*
displays the data from the model.  Spring MVC offers support for a
plethora of view technologies for HTML. For Atom, Spring uses the
[Rome Library](https://rometools.github.io/rome/). It offers different
Java objects to display data as feeds and entries in the feeds.

#### Implementation of the Controller

{linenos=off, lang="java"}
~~~~~~~~
public ModelAndView orderFeed(WebRequest webRequest) {
  if ((orderRepository.lastUpdate() != null)
   && (webRequest.checkNotModified(orderRepository.lastUpdate().getTime()))) {
   return null;
  }
  return new ModelAndView(new OrderAtomFeedView(orderRepository),
   "orders", orderRepository.findAll());
}
~~~~~~~~

The method `orderFeed()` in the class `OrderController` is responsible
for displaying the Atom feed with the help of `OrderAtomFeedView`. As
shown in the listing, `OrderAtomFeedView` is returned as the view and
a list of the orders as model. The view then displays the orders from
the model in the feed.

#### Implementation of HTTP Caching on the Server

Spring provides the `checkNotModified()` method in the `WebRequest`
class to simplify the handling of HTTP caching. The time of the last
update is passed to the method. The `lastUpdate()` method of the
`OrderRepository` determines this time point with a database
query. Each order contains the time at which it was
placed. `lastUpdate()` returns the maximum value. `checkNotModified()`
compares this passed value with the value from the `If-Modified-Since`
header in the request. If no more recent data needs to be returned,
the method returns `true`. Then, `orderFeed()` returns *null*, so that
Spring MVC returns an HTTP status code 304 (Not Modified).  So in this
case, the server just makes a simple query to the database and
returns an HTTP response with a status code. It does not provide any
data.

#### Implementation of HTTP Caching on the Client

On the client side, HTTP caching must of course also be
implemented. The microservices *microservice-order-invoicing* and
*microservice-order-shipping* implement the polling of the Atom feed
in the method `pollInternal()` of the class `OrderPoller`. They set
the `If-Modified-Since` header in the request (see line 4/5 in the
listing). The value is determined from the variable `lastModified`. It
contains the value of the `Last-Modified` header of the last HTTP
response (line 15-17). If no data has been changed in the meantime,
the server responses to the GET
request directly with an HTTP status 304 and it is clear
that no new data exists.  Accordingly, data are only processed if the
status is not `NOT_MODIFIED` (line 11).

{lang="java"}
~~~~~~~~
public void pollInternal() {
  HttpHeaders requestHeaders = new HttpHeaders();
  if (lastModified != null) {
    requestHeaders.set(HttpHeaders.IF_MODIFIED_SINCE,
     DateUtils.formatDate(lastModified));
  }
  HttpEntity<?> requestEntity = new HttpEntity(requestHeaders);
  ResponseEntity<Feed> response =
    restTemplate.exchange(url, HttpMethod.GET, requestEntity, Feed.class);

  if (response.getStatusCode() != HttpStatus.NOT_MODIFIED) {
    Feed feed = response.getBody();
    ...	// evaluate feed data
    if (response.getHeaders().getFirst(HttpHeaders.LAST_MODIFIED) != null) {
      lastModified =
       DateUtils.parseDate(
        response.getHeaders().getFirst(HttpHeaders.LAST_MODIFIED));
    }
  }
}
~~~~~~~~

`pollInternal()` is called by the method `poll()` in the class `OrderPoller`.
The user can call this method with a button in the web UI. In
addition, the microservice calls the method
every 30 seconds because of the `@Scheduled` annotation.

#### Data Processing and Scaling

If there are multiple instances of the invoicing and shipping
microservices, each instance polls the Atom feed and processes the
data. Of course, it is not correct that
several instances write an invoice for an order or initiate a
delivery because then an order would create multiple invoices or
deliveries.  Therefore, each instance must determine which orders are 
already processed and which data is in the database. If another
instance has already created the data record for the invoice or
delivery of the order, then the entry from the Atom feed for the order
must be ignored. To do this, `ShippingService` and `InvoicingService`
use a transaction in which a data record is first searched for in the
database.  A new
data record is written only if none yet exists. Therefore, only one
instance of the microservices can write the data record. All others
read the
data and find out that another instance has already written a data
record. With a very large number of instances, this can cause a
considerable load on the database. In return, the services are
idempotent. No matter how often they are called, in the end the state
in the database is always the same.

#### Atom Cannot Send Data to a Single Recipient

This is a disadvantage of Atom. It is not easily possible to send a
message to exactly one instance of a microservice. Instead, the
application has to deal with multiple instances reading the message
from the Atom feed.
Thus, especially when a lot of point to point communication is
necessary, the Atom approach can be disadvantageous.

The application must also be able to deal with messages not being
processed.  If a message has been read, the process can fail before
the message has been processed. Then for
some messages no data might be written. In this case, however, at
some point another instance of the microservice would read the message
and process 
it.
So retries are actually quite easy to implement with Atom.

## 12.3 Recipe Variations {#section-atom-variationen}

Instead of Atom, a different format can be used for the communication
of changes via HTTP.

#### RSS

Atom is just one format for feeds. An alternative is
[RSS](http://web.resource.org/rss/1.0/spec). There are different
versions of RSS. RSS is older than Atom. Atom has learnt from RSS and
represents the more modern alternative. Blogs and podcasts should
offer feeds as RSS and Atom to reach as many clients as
possible. Since in case of microservices server and client are under
the control of the same developer, it is not necessary to support
multiple protocols. Therefore, Atom is the better and more modern
alternative.

#### JSON Feed

Another alternative is [JSON Feed](http://jsonfeed.org/). It also
defines a data format for a feed. However, it uses JSON instead of
XML, which is used by Atom and RSS.

#### Custom Data Format

Atom and RSS are only formats for communicating changes. Some of the
elements are not useful for data, but only for blogs or podcasts. The
useful part of Atom and RSS is the list of changes and the links to
the actual data. Atom and RSS therefore use hypermedia to communicate
the changes without delivering all data.

It is of course conceivable to define your own data format, which
contains the changes and links to the data. In addition to
the links, the data can also be embedded directly into the
document.

Compared to Atom and RSS, a custom data format has the disadvantage
that it is not standardized. For a standardized data model, libraries
are available and learning about the format is easier. There are
also tools such as validators. A user can read and display Atom data
with an Atom reader, which is, for instance, useful for
troubleshooting.

However, the data to be transported is not very complex. So a
custom data format has no major disadvantages. In essence, even with
Atom, the approach simply uses hypermedia as an essential component
of REST. It provides a list of links that clients can use to get more
information about the changes to the data. This procedure can also be
easily implemented with a custom data format.

#### Alternatives to HTTP

HTTP supports features such as scaling or reliability very
well.  Most applications already use HTTP even without Atom to deliver
web pages or provide REST services. The alternative to HTTP would be a
messaging system like Kafka (see [chapter 11](#chapter-kafka)), which
can also be used to implement asynchronous communication. Such
messaging systems must be scalable and have to provide high
availability. The messaging systems offer these features in principle,
but must be tuned and configured accordingly. This is especially
challenging if you have never operated such a messaging system before.
In particular, the advantages of HTTP concerning operation argue for
using this protocol also for asynchronous communication.

#### Including Event Data

Of course, the feed can also include the event data instead of just
links. In this way, a client can work with the data without further
requests to load the linked data. But the feed gets bigger. The
question also arises as to which data should be included in the
feed. Each client may need different data, which makes it difficult to
model the data. Sending links has the advantage that the client can
select the appropriate representation of the data with content
negotiation .

## 12.4 Experiments {#section-atom-experimente}

* Start the system and examine the logs of
  *microservice-order-invoicing* and *microservice-order-shipping*
  with `docker logs -f msatom_invoicing_1` respectively `docker logs
  -f msatom_invoicing_1`. The microservices log messages when they
  poll data
  from the Atom feed, because there are new orders.  If you start
  additional instances of a microservice with `docker compose up
  --scale`, these new instances will also collect orders via the Atom
  feed and log information about them. In doing so, only one instance
  writes at a time, the other ones ignore the data.  Create orders and
  notice this behavior based on the log messages. Explore the code to
  find out what the log messages mean exactly and where they are
  put out.

* Supplement the system with an additional microservice.
  * As an example, a microservice can be used which credits the
    customer with a bonus depending on the value of the order or which
    counts the orders.
  * Of course, you can copy and modify one of the existing
    microservices.
  * Implement a microservice which polls the URL
  `http://order:8080/feed`.
  * In addition, the microservice should display an HTML page with
    some information (customer bonus or number of calls).
  * Package the microservice in a Docker image and reference it in
    `docker-compose.yml`. There you can also determine the name of the
    Docker container.
  * Create a link from the container `apache`
    to the container with the new service in `docker-compose.yml`  and
    from the container with the 
    new service to the container `order`. 
  * The microservice has to be accessible via the homepage. For this
    purpose, you have to create a load balancer for the new Docker
    container in the file `000-default.conf` in the Docker container
    `apache`. Use the name of the Docker container for this. Then you
    have to add a link to the new load balancer in `index.html`.
  * Optional: Add HTTP caching.

* Currently, it is only possible to request all orders at once in the
  Atom feed. Implement paging so that only a subset of the orders is
  returned.

* At the moment, the system runs with Docker compose. However, it
  could also run on a different infrastructure, for instance, on a
  microservices platform ([chapter 16](#chapter-plattformen)).
  [Chapter 17](#chapter-kubernetes) discusses Kubernetes in more
  detail, and [chapter 18](#chapter-paas) deals with Cloud Foundry.
  Port the system to one of these platforms.

* Instead of using the Atom format, you could also deliver your own
  representation of a feed for example as a JSON document. Change the
  implementation in the example so that it uses its own custom data
  format.

## 12.5 Conclusion {#atom-fazit}

REST and the Atom format offer an easy way to implement asynchronous
communication. HTTP is used as the communication protocol. This has
several advantages. HTTP is well understood, has already proven its
scalability many times, and most of the time HTTP is already used in
projects anyway to transfer other content like JSON via REST or to
deliver HTML pages. Therefore, most teams have the necessary
experience to implement scalable systems with HTTP. Since HTTP caching
is supported, polling of Atom resources can be implemented very
efficiently.

It is helpful if also very old events are still available. Then
another microservice with event sourcing can rebuild its state by
processing all events again. With an Atom-based system, a microservice
must also provide old events. This can be done very easily in some
cases. If the microservice has stored the old information anyway, it
only needs to provide it as an Atom feed.  In this case, access to old
events is thus very easy to implement.

The data in the Atom Feed comes from the same source as all other
representations. Therefore, the data is consistent with the data that
can be queried via other means. All these services represent only
different representations of the same data.

However, Atom cannot guarantee the delivery of the message.
Therefore, the microservices in which the messages are processed
should be implemented idempotently and try to read the message several
times to ensure that they are processed.

Atom has no way to limit the reception of a message to just one
microservice. Therefore, the instances of the microservice must select
an instance that then processes the message, or you have to rely on
the idempotency.

Atom can guarantee the order of the messages because the feed is a
linear document and so the order is fixed. However, this requires that
the order of the entries in the feed does not change.

All in all, Atom is a very simple alternative for asynchronous
communication.

#### Benefits

* Atom does not require additional infrastructure, just HTTP.
* Old events are easily accessible if necessary. This can be
  advantageous for event sourcing.
* The sequence can be guaranteed.
* The Atom feed is consistent. It is just another representation of
  the data and contains exactly the same information as all other
  representations of the data.

#### Challenges

* Guaranteed delivery is difficult. The recipient can attempt to read
  the data 
  several times to guarantee all data is processed. However, the
  responsibility for this lies with the
  recipient not with the infrastructure.
* Messages for only one recipient are difficult. All recipients poll
  the messages. They must then decide which recipient actually
  processes the message.
* In part, Atom is not well suited for the representation of events
  because it was originally designed for blogs.

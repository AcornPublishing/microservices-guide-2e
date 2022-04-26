# 13 Concept: Synchronous Microservices {#chapter-synchron}

Synchronous microservices are one way in which microservices can
communicate. This chapter shows:

* What is meant by synchronous communication between microservices.

* What the architecture of a synchronous microservices system can look
like.

* Which advantages and disadvantages are associated with synchronous
communication between microservices.

## 13.1 Definition {#section-synchron-defintion}

This chapter deals with the technical options for implementing
synchronous microservices. [Chapter 10](#chapter-asynchron) already
introduced the term "synchronous microservices".

A> A microservice is synchronous if it makes a request to other
A> microservices while processing requests and waits for the result.

The logic to handle a request in the microservice might therefore not
depend on the result of a request to a different microservice.

{id="fig-synchron-konzept"}
![Fig. 13-1: Synchronous Communication](images/synchron-konzept.png)

[Figure 13-1](#fig-synchron-konzept) illustrates this kind of
communication. While the left microservice processes a request, it
calls the right microservice and waits for the result of this call.

Synchronous and asynchronous communication according to this
definition are independent of the communication protocol. A
synchronous communication protocol means that a request returns a
result. For example, a REST or HTTP GET returns a result such as an
HTTP status, a JSON document, or an HTML page. If a system processes a
REST request, makes a REST request itself to another system and waits
for the response, it is synchronous. Asynchronous REST systems were
discussed in [chapter 12](#chapter-atom).

Asynchronous communication protocols, on the other hand, send messages
to which the recipients react. There is no direct response. Synchronous
communication with an asynchronous protocol occurs when one system
sends a message with an asynchronous communication protocol to another
system and then waits to receive a response with an asynchronous
communication protocol.

#### An Example

For example, a microservice for orders is synchronous if it calls 
a microservice for customer data while processing an order and waits for
the customer data.

{id="fig-synchron-architektur"}
![Fig. 13-2: Architecture for a Synchronous System](images/synchron-architektur.png)

[Figure 13-2](#fig-synchron-architektur) shows an exemplary
synchronous architecture, which corresponds to the asynchronous
architecture displayed in [figure 10-3](#fig-asynchron-architektur)
within [chapter 10](#chapter-asynchron). It describes an excerpt from
an e-commerce system. The microservices *customer
management*, *items*, *order*, *invoice*, and *delivery* manage the
respective data. The *catalog* displays all information about the
goods and takes into account the customer's preferences. Finally,
the *order process* serves to order goods, issue the invoice, and
deliver the goods. The UI accesses *catalog* and *order process* and
thus makes the processes implemented in these microservices available
to the
user. The UI can also display invoice and delivery data.

#### Consistency

The architecture of this system ensures that all microservices display
the same information about a product or order at all times because
they use synchronous calls to access the respective microservices in
which the data is stored. There is only one place where each piece of
the data is
stored. Every microservice uses that data and therefore shows the
latest data.

However, this architecture also has disadvantages. Centralized data
storage can lead to problems because the data of a customer for
displaying the catalog is completely different from that needed for
the order process. The customer's purchasing behaviour is important
for the catalog in order to display the right products. For orders, on
the other hand, the delivery address or the preferred delivery service
are relevant.

Storing all this data in a microservice can make the data model
complex. Domain-driven design[^synchron-ddd] states that a domain
model is only valid in a particular bounded context (see
[section 2.1](#section-mikro-makro-bounded-context)). Therefore, such
a centralized model is problematic.

[^synchron-ddd]: Eric Evans: Domain Driven Design: Tackling Complexity in the Heart of Software, Addison Wesley, 2003, ISBN 978 0 32112 521 7

 The illustration also reveals another problem. Most of the
functionalities use many microservices. That complicates the system.
In addition, the failure of a microservice means that many
functionalities are no longer available if no special precautions are
taken. Performance may also suffer because the microservices have to
wait for results from many other microservices.

#### Bounded Context

Such an architecture is often found in synchronous architectures.
However, it is not mandatory. Theoretically, it is conceivable that
bounded contexts could communicate synchronously with each other. But
bounded contexts typically exchange events. This is particularly easy
with asynchronous communication and not a great fit for synchronous
communication.

#### Tests

To enable independent deployment, the tests of each microservice
must be as independent as possible and integration tests should be
kept to a minimum.

With synchronous systems, for tests the communication partners must be
available. These can be the microservices used in production. In this
case, setting up an environment is hard because many microservices
have to be available, and dependencies arise between the microservices
because a new version of a microservice must be made available to all
clients in order to enable tests.

#### Stubs

An alternative are stubs that simulate microservices and simplify the
setup of the test environment. Now the tests no longer depend on other
microservices because the tests use stubs instead of microservices.

#### Consumer-driven Contract Tests

Finally
[consumer-driven contract tests](https://martinfowler.com/articles/consumerDrivenContracts.html)
can be a solution. With this pattern, the client writes a test for the
server. In this way, the server can test whether it meets the client's
expectations. This makes it easier to change the server because
changes to the interface can be tested without a client. In addition,
the tests then no longer depend on the client microservice.

Consumer-driven contract tests can be written with a testing framework
like JUnit, for example. The
team that implements the called microservice must then execute the tests.
If the tests fail, they have made an incompatible change to the
microservice. Either the microservice must then be changed to be
compatible or the team that wrote the consumer-driven contract test
must be informed so that they can change their microservice in such a
way that the interface is used differently according to the change.
Consumer-driven contract tests therefore formalize the definition of
the interface.

As part of the macro architecture, all teams must agree on a test
framework in which the consumer-driven contract tests are written.
This framework does not necessarily have to support the programming
language in which the microservices are written when the
consumer-driven contract tests use the microservices as black boxes
and only access them through the REST interface.

#### The Pact Test Framework

One option is the framework [Pact](https://pact.io). It enables
writing tests of a REST interface in a programming language. This
results in a JSON file containing the REST requests and the expected
responses. With Pact implementations, the JSON file can be interpreted
in different programming languages. This increases the technology
freedom. An example for Pact with Java can be found at
<https://github.com/mvitz/pact-example>.

## 13.2 Benefits {#section-synchron-vorteile}

In practice, synchronous microservices are a relatively frequently
used approach. Many well-known examples for microservices
architectures use such a concept. This has
the following advantages:

* All services can access the same dataset, so there are *less
consistency problems*.

* Synchronous communication is a
natural approach if the system is to offer an *API*. Any microservice
can implement part of the API. The API can be the product. This is the
case, for example, with a provider of payment solutions. Or the API
can be used by mobile applications.

* After all, it can be easier to *migrate* into such an architecture.
For example, the current architecture can already have such a division
into different synchronous communication endpoints or teams can exist
for each of the functionalities.

* Calling methods, procedures, or functions in a program is usually
synchronous. Developers are thus *familiar* with this model so that
they can understand it more easily.

## 13.3 Challenges {#section-synchron-herausforderungen}

Synchronous microservices create a number of challenges:

* The communication with other microservices during request processing
causes the latencies for responses of other microservices and the
communication times via the network to add up. Therefore, synchronous
communication can lead to a performance problem. When a microservice
reacts slowly, this can have consequences for a plethora of other
microservices. In the example in
[figure 13-2](#fig-synchron-architektur), the *catalog* uses two other
microservices (*item* and *customer management*). Thus the latencies
of these three systems can add up. A request to the *catalog
microservice* creates requests to the *customer management
microservice* and to the *item microservice*. Only when all
microservices have responded to the requests, the user gets to see the
result.

* When a synchronous microservice calls a failed microservice, it can
happen that also the calling microservice crashes and the failure
thus propagates. This makes the system very vulnerable. The
vulnerability of the microservices and the adding up of waiting times
can prevent the reliable operation of microservices systems with
synchronous communication. Therefore, these problems first have to be
largely solved when a microservices system uses synchronous
communication.

* In addition, synchronous communication can create a higher level of
dependency in the domain logic. Asynchronous communication often
focuses on
events (see [section 10.2](#section-asynchron-events)). In this case, a
microservice can decide how to react to events. In contrast,
synchronous communication typically defines what a microservice is
supposed to do. In the example, the *order process* would make the
*invoice microservice* generate an invoice. With events, the *invoice
microservice* would decide itself how to react to the event. This
facilitates the extendability of the system. If the customer is
supposed to be credited with bonus points for an order, there only has
to be an additional microservice reacting to the already existing
event. That event is probably already processed by more than one
microservice. So just another receiver needs to be added. In a
synchronous architecture an additional system has to be called.

#### Technical Solutions

For implementing a system of synchronous microservices some technical
solutions are required:

* The microservices have to know how they can communicate with other
microservices. Usually, this requires an IP address and a
port. *Service discovery* serves to find the port and IP address of a
service. Service discovery should be dynamic. Microservices can be
scaled. Then there are new IP addresses at which additional
instances of a microservice are available. In addition, a microservice
can fail. Then it is not available anymore at the known IP address.
Service discovery can be very simple. DNS (Domain Name System) for
example provides IP addresses for servers on the Internet. This
technology already provides
a simple service discovery.

* When communication is synchronous, microservices have to be prepared
for the failure of other microservices. It has to be prevented that
the calling microservice fails as well. Otherwise, there will be
failure cascade. First one microservice fails, then other
microservices call this microservice and subsequently also fail. In
the end, the entire system will be down. When in
[figure 13-2](#fig-synchron-architektur) for example the service for
the items fails, the catalog and order process could fail in turn.
This would render a large part of the system unusable. Therefore,
there has to be a technical solution to achieve *resilience*.

* Each microservice should be scalable independently of the other
microservices. Load has to be distributed between microservices. This
does not only pertain to access from the outside, but also to
internal communication. Therefore, there has to be a *load balancing*
for each microservice.

* Finally, every access from the outside should be forwarded to the
responsible microservices. This requires *routing*.
A user might want to use the catalog and the order process. While they
are separate microservices, they should appear as parts of the same
system to the outside.

Consequently, the technologies for synchronous microservices which are
discussed in the following chapters have to offer solutions for
service discovery, resilience, load balancing, and routing.

#### API Gateways

For complex APIs complex routing of requests to the microservices
might be needed. API Gateways offer additional features. Most
of the API gateways can perform user authentification. In addition,
they can throttle the network traffic for individual users to support
a high number of users at the same time. This can be supplemented e.g
by a centralized logging of all requests or caching. Moreover, API
gateways can also solve aspects like monitoring, documentation or
mocking.

The implementations of [Apigee](https://apigee.com/api-management/),
[3scale by Red Hat](https://www.redhat.com/de/technologies/jboss-middleware/3scale),
[apiman](http://www.apiman.io/latest/) or cloud products like the
[Amazon API Gateway](http://docs.aws.amazon.com/de_de/apigateway/latest/developerguide/welcome.html)
and the
[Microsoft Azure API Gateway](https://azure.microsoft.com/de-de/services/api-management/)
are examples of API gateways.

The examples in this book do not offer public REST interfaces, but
only REST interfaces for internal use. The public interfaces are just
web pages. Thus, the examples do not use API gateways.

## 13.4 Variations {#section-synchron-variationen}

Frontend integration ([chapter 7](#chapter-frontend)) can be a good
addition to synchronous communication. Asynchronous communication (see
[chapter 10](#chapter-asynchron)) is rather an alternative. Synchronous
and asynchronous communication are both possibilities with which
microservices can communicate with each other on the logic level.

One of these options should be enough to build a microservices
system. Of course, a combination is also possible. The asynchronous
communication with Atom and the synchronous communication with REST
use the same infrastructure so that these two communication
mechanisms can be used together very easily.

The following chapters show concrete implementations for synchronous
communication. The examples all use REST for communication. In today's
technology landscape, REST is the preferred architecture for
synchronous communication. In principle, other approaches such as
[SOAP](https://www.w3.org/TR/soap/) or
[Thrift](https://thrift.apache.org/) are also conceivable.

## 13.5 Conclusion {#section-synchron-fazit}

At first glance, synchronous microservices are very simple because
they correspond to the classic programming model. But synchronous
microservices lead to a high degree of technical complexity because
microservices have to deal with the failure of other microservices and
latencies add up. The resulting systems are distributed systems
and thus technically complex. This has few advantages. It is advisable
to choose this approach only if there are really good reasons for
doing so. In general, asynchronous communication makes it easier to
deal with the challenges of distributed systems than synchronous
communication.

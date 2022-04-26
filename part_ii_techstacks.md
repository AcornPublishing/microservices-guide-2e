# Part II: Technology Stacks {#chapter-techstacks}

The second part of the book deals with recipes for technologies that can be used to implement microservices.

#### Docker

First, [chapter 4](#chapter-docker) provides an introduction
to *Docker*. Docker offers a good foundation for the implementation of
microservices and is the basis for the examples in this book. Reading
this chapter is therefore critical for understanding the examples in
the later chapters.

#### Technical Micro Architecture

[Chapter 2](#chapter-mikro-makro) introduced the concepts of micro and
macro architecture. Micro architecture comprises the decisions that
can be made differently for each microservice. Macro architecture are
the decisions that have to be uniform for all microservices.
[Chapter 5](#chapter-technisch-mikro) discusses technical
possibilities for the implementation of the micro architecture
of a microservice.

#### Self-contained Systems

[Chapter 6](#chapter-scs) describes *self-contained systems*.  They
are a collection of best practices for microservice architectures with
a focus on independence and web applications. In addition to benefits
and disadvantages possible variations of this idea will be discussed.

SCS always include a web ui and rely on frontend
integration. Therefore it makes sense to discuss this approach right
before frontend integration is explained in more detail to motivate
this approach for integration.


#### Frontend Integration

One possibility for the integration of microservices is *frontend
integration*, which is explained in [chapter 7](#chapter-frontend). A
concrete technical implementation with *links* and *client-side
integration with JavaScript* is shown in
[chapter 8](#chapter-links-javascript).
[Chapter 9](#chapter-esi) describes *Edge Side Includes (ESI)* that
provide UI integration on the server.

#### Asynchronous Microservices

*Asynchronous microservices* are presented in
[chapter 10](#chapter-asynchron). [Chapter 11](#chapter-kafka)
introduces *Apache Kafka* as an example of a middleware that can be
used to implement asynchronous microservices. *Atom*, the topic of
[chapter 12](#chapter-atom), is a data format that can be useful for
asynchronous communication via REST.

#### Synchronous Microservices

Synchronous microservices are explained in
[chapter 13](#chapter-synchron). The *Netflix stack* which is
discussed in [chapter 14](#chapter-netflix) is a way to implement
synchronous microservices. The stack includes solutions for load
balancing, service discovery and
resilience. [Chapter 15](#chapter-consul) shows *Consul* as an
alternative for service discovery and introduces *Apache httpd* for
load balancing.

#### Microservices Platforms

[Chapter 16](#chapter-plattformen) discusses *microservice platforms*
which provide support for synchronous communication
and also a runtime environment for deployment and operation.
[Chapter 17](#chapter-kubernetes) demonstrates how synchronous
microservices can be implemented with *Kubernetes*. Kubernetes serves
as runtime environment for Docker containers and has, among others,
features for load balancing and service discovery.

[Chapter 18](#chapter-paas) describes *PaaS* (Platform as a Service).
A PaaS makes it possible to leave the operation and deployment of the
microservices to the infrastructure for the most part. *Cloud Foundry*
is discussed as an example for a PaaS.

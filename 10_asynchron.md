# 10 Concept: Asynchronous Microservices {#chapter-asynchron}

Asynchronous microservices have advantages over synchronous microservices. 
This chapter conveys:

* How microservices can communicate asynchronously.

* Which protocols can be used for asynchronous communication.

* How events and asynchronous communication are linked.

* What the advantages and disadvantages of asynchronous communication are.

## 10.1 Definition {#section-asynchron-definition}

Asynchronous microservices distinguish themselves from synchronous
microservices. [Chapter 13](#chapter-synchron) describes synchronous
microservices in detail. The term "synchronous microservices" stands
for the following:

A> A microservice is synchronous if it makes a request to other microservices 
A> while processing requests and waits for the result.

The logic to handle a request in the microservice might therefore not
depend on the result of a request to a different microservice.
So a definition of asynchronous microservices would be:

A> A microservice is asynchronous (a) if does not makes a request to other
A> microservice while processing requests or (b) makes a request to other
A> microservice while processing requests  and does not wait for the result.

So there are two cases here:

* (a) The microservice does not communicate at all with other systems while
 processing a request. In that case, the microservice will typically
 communicate with the other systems at a different time (see
 [figure 10-1](#fig-asynchron-konzept-keine-kommunikation)). For
 example, the microservice can replicate data that is used when
 processing a request. In this way, customer data can be replicated
 so that when processing an order, the microservice can access the
 locally available customer data, instead of having to load the
 necessary customer data for each request via a request to another
 system.

{id="fig-asynchron-konzept-keine-kommunikation"}
![Fig. 10-1: Communication Only Outside of Requests](images/asynchron-konzept-keine-kommunikation.png)

* (b) The microservice sends a request to another microservice, but does
 not wait for a response (see
 [figure 10-2](#fig-asynchron-konzept-keine-antwort)). A
 microservice responsible for processing an order can send a request
 to another microservice which generates the invoice. A response to
 this request is not necessary for processing the order so there is
 no need to wait for it.

{id="fig-asynchron-konzept-keine-antwort"}
![Fig. 10-2: Communication Without Waiting for a Response (Fire-and-Forget)](images/asynchron-konzept-keine-antwort.png)

[Figure 10-3](#fig-asynchron-architektur) shows an example for a more
complex
asynchronous architecture. In this E-commerce system, orders are
processed. Via the catalog, customers can choose goods for an order.
The *order process* generates the orders. An *invoice* and a
*shipping* is produced for the order. The *registration* microservice
adds new customers to the system. The *listing* microservice is
responsible for new goods.

{id="fig-asynchron-architektur"}
![Fig. 10-3: Architecture for an Asynchronous System](images/asynchron-architektur.png)

#### Asynchronous Communication with no Response

The four systems *catalog*, *order process*, *invoice* and *shipping*
send asynchronous notifications for processing the orders. The
*catalog* collects goods in the shopping cart. If the user 
orders the shopping cart, the *catalog* transfers the
cart to the *order process*. The *order process* turns the shopping
cart into an order. The order then becomes an invoice and a delivery.
Such requests can be executed asynchronously. No data has to
flow back. The responsibility for the order is transferred to the next
step in the process.

#### Data Replication and Bounded Context

This becomes more complicated if data is required to execute a
request. For example, in the *catalog*, in
the *order process*, and for the *invoice*, data about products and
customers has to be available. So each of the systems
stores a part of the information about these business objects.
The *catalog* must display the products. So it has
pictures and descriptions of the products. For invoices prices and tax
rates are important. This corresponds to the 
bounded contexts from
[section 2.1](#section-mikro-makro-bounded-context), each of which has
its own domain model.

Each bounded context has its own domain model. That means that all
data for the bounded context is represented in its domain
model. Therefore the data specific for the bounded context should be
store in the bounded context in its own database schema. Other bounded
context should not access that data directly. That would compromise
encapsulation. Instead the data should only be accessed by the logic
in the bounded context and its interface.

While it would be possible to have a system that contains all
information about e.g. a product, this would not make a lot of
sense. The model of the system would be very complicated. Also it
means that the domain model would be split across e.g. a system for an
order process and a system for the product data. That would lead to a
very tight coupling.

A third system, such as *registration* for customer
data or *listing* for product data, must accept all the data and
transfer the needed parts of the data to the
respective systems. This can also be done asynchronously. The other
bounded contexts then store the information about products and customer in
their local
databases.
So in that case, the replication is just a result of events being
processed. An even such as "new product added" will make each bounded
context add some data to its domain model.
*Registration* or *listing* do not need to store the data. After they
have send the data to the other microservices, their job is done.

It is also possible to do an extract-transform-load approach. In that
case a batch would *extract* the data from one bounded context,
transform it to a different format and load it into the other bounded
context. This is useful if a bounded context should be loaded with an
initial set of data or if inconsistencies in the data require a fresh
start.


#### Synchronous Communication Protocols

Asynchronous communication as defined above does not make any
assumptions about the communication protocol used. 
For synchronous communication, the server must respond
to each request. An example are REST and HTTP. A request leads to a
response that contains a status code and can contain additional data.
It is possible to implement asynchronous communication with a
synchronous communication protocol.
[Chapter 12](#chapter-atom) explains this in more detail.

#### Asynchronous Communication Protocols

It is more natural to implement asynchronous communication with
an asynchronous communication protocol. An asynchronous communication
protocol sends messages and does not expect responses. Messaging
systems like Kafka (see [chapter 11](#chapter-kafka)) implement this
approach.

There is also a 
[presentation](https://www.slideshare.net/ewolff/rest-vs-messaging-for-microservices)
which explains the difference between REST and messaging and
highlights that both technologies can be used to implement synchronous
communication like *request / reply*, but also asynchronous
communication like *fire & forget* or *events*.

## 10.2 Events {#section-asynchron-events}

With asynchronous communication, the coupling of systems can be driven
to different
lengths. As already mentioned, the system for *order processing*
could inform the *invoice* system asynchronously that an invoice has
to be written. The ordering system thus determines exactly what the
invoicing system has to do. It is supposed to generate an invoice.
It also sends a message to the bounded context *shipping* to trigger
the delivery.

The system can also be set up differently. The focus will then be on
an event like "There is a new order". Every microservice can react
appropriately to this. The invoicing system can write an invoice and
the shipping system can prepare the goods for delivery. So each
microservice decides for itself how it reacts to the events.

{id="fig-asynchron-events"}
![Fig. 10-4: Asynchronous Events](images/asynchron-events.png)

This leads to a better decoupling. If a microservice has to react
differently to a new order, the microservice can implement this change
on its own. It is also possible to add a new microservice, which
generates statistics, for example, when a new order is placed.

#### Events and DDD

However, this approach is not quite as easy to implement. The crucial
question is what data is transferred with the event. If the data is to
be used for such different purposes as the writing of an invoice,
statistics, or recommendations, then a large number of different
attributes must be stored in the event.

This is problematic since domain-driven design shows that each domain
model is only valid in a bounded context (see
[section 2.1](#section-mikro-makro-bounded-context)). For invoicing
prices and tax rates have to be known. For shipping size and weight of
the goods are needed to organize a suitable transport.

#### Patterns from Strategic Design

The views of *invoice* and *shipping* on the data of a order thus
represent two bounded contexts, which can receive data from a third
bounded context, the *order process*. Domain-driven design defines
patterns for this (see
[section 2.1](#section-mikro-makro-bounded-context)). For example,
with customer/supplier the team for invoicing and shipping can define
which data it needs to receive. The team that provides the data for
the order must meet these requirements. This pattern defines the
interaction of the teams which develop the bounded contexts that
participate in the communication relationship. Such coordination is
necessary regardless of whether events are sent or whether
communication between components takes place by a synchronous call.

In other words: Events may seem to decouple the system, but
coordination regarding the necessary data must still take place. This
means that events do not necessarily lead to a truly decoupled
system. In extreme cases, events can even lead to hidden dependencies.
Who reacts to an event? If this question can no longer be answered,
the system is hardly changeable anymore because changes to the events
have unforeseeable consequences so they are usually not changed anymore.

A solution might be to provide specific types of events for each
receiver. Each type of event would just contain the information that
this receiver needs. So if a new order appears in the system, an event
is sent to the invoicing system with the data it needs. Another event
is sent to shipping with the data for that system. The two systems are
truly decoupled: A change in the interface to one of the systems does
not influence the other system. This can be the result of a
customer/supplier relationship.

Another solution would be to use a published language (see [section
2.1](#section-mikro-makro-bounded-context)). In that case there is a
common data structure that contains all information for all
receivers. This makes it hard to understand which receivers uses
what. Changes to the data structure might lead to unforeseen
problems. However, there is just one data structure so it is somewhat
easier to implement the system.

A very important question is how different the information is that
each receiver needs. If it is mostly the same, a published language
might be better. If it is very different, it might make more sense to
have separate data structures. For the case of invoicing and shipping,
there is probably not too much overlap so two separate data structures
might be the better alternative.

#### Sending Minimal Data in an Event

There is yet another way to deal with this problem. In the event, 
only an ID number can be sent along, for example, the number of a new order. 
Then every bounded context can decide for itself how to get the necessary data. 
There can be a special interface for each bounded context that provides the 
appropriate data for that specific bounded context.

#### Event Sourcing

An architecture focus on events can also entail other advantages.
The state each microservice has in its database is the result of the
events it has 
received. So the state of a microservice can be restored by resending
all events it had received so far. The microservice can even change
its internal domain model and then process the events again to rebuild
its database with the new version of the domain model. That facilitates
database schema migration. Thus, each microservice can have its own
domain model
according to the bounded context pattern, but all microservices are
still connected  by the events they send to each other. There is no
longer
an overall state of the system, but when all events are saved and can
be retrieved, the state of each microservice can be reconstructed.
These ideas form the basis for event sourcing.

{id="fig-asynchron-event-sourcing"}
![Fig. 10-5: Event Sourcing ](images/asynchron-event-sourcing.png)

The elements of an event sourcing implementation are shown in
[figure 10-5](#fig-asynchron-event-sourcing):

* The *event queue* sends the events to the recipients.

* The *event store* saves the events.

* *Event handlers* process the events. They can save their state as a
*snapshot* in a database.

The event handler can read the current state from the snapshot. The
snapshot can be deleted. The  snapshot can be restored 
on the basis of the events, which can be retrieved from the
event store. As an optimization, an event handler can also reconstruct
its state from an older version of the snapshot.

There is a difference between the events for event sourcing and domain
events, see [Christian Stettler's blog
post](https://www.innoq.com/en/blog/domain-events-versus-event-sourcing/).

#### Individual or Shared Event Store?

The event store can be part of the microservice which receives events
and stores them in its own event store. Alternatively, the
infrastructure cannot only send the events, but also store them. At
first glance, it seems to be better if the infrastructure stores the
events because it simplifies the implementation of the microservices.
In such a case the event store would be implemented in the event queue.

If each microservice stores the events in its own event store, the
microservice can store all relevant data in the event, which the
microservice may have collected from different sources. When storing
events in the infrastructure, it is necessary to find a model of the
event that satisfies all microservices. Such a model for the events
can be a challenge because of the concept of bounded context (see
[section 2.1](#section-mikro-makro-bounded-context)). After all,
every microservice is a separate bounded context with its own domain
model so finding a common model is hard.
	
## 10.3 Challenges {#section-asynchron-herausforderungen}

If the communication infrastructure for event sourcing has to store old
events, it has to handle considerable amounts of data. If old events are
missing, the state of a microservice can otherwise no longer be
reconstructed from the events. As an optimization, it would be possible
to delete events that are no longer relevant. If a customer has moved
to a different address
several times, it is probably only the last address that is really
relevant. The others can then be deleted.

In addition, it must also be possible to continue processing old
events. If the schema of the events changes in the meantime, old
events have to be migrated. Otherwise, every microservice has to be
able to handle events in all old data formats. This is particularly
difficult if new data has to be contained in the events that has not
yet been saved in old events.

#### Inconsistency

Due to asynchronous communication, the system is not consistent. Some
microservices already have certain information, others do not. For
example, *order process* might already have information about an
order, but *invoicing* or *shipping* do not know about the order
yet. This problem cannot
be solved. It takes time for asynchronous communication to reach all
systems.

#### CAP Theorem

These inconsistencies are not only practical problems, but cannot even
be solved in theory. According to the
[CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem) there are
three characteristics in a distributed system:

* *Consistency* (C) means that all components of the system have the
 same information.

* *Partition tolerance* (P) means that a system will continue to work
 in case of arbitrary package loss in the network.

* *Availability* (A) means that no system stops working because
 another system failed.

The CAP theorem states that a system can have a maximum of two
features out of these three. Partition tolerance is a special case. A
system must react if the network fails. In fact, not even a complete
failure is
necessary, the packages loss just needs to be high or the
response time is very long. A system that responds very slowly is
indistinguishable from a system that has failed completely.

#### Reasons for the CAP Theorem

{id="fig-asynchron-CAP"}
![Fig. 10-6: If Communication Fails, a System Can Either Return a Potentially Wrong Response Upon a Request (AP) or None at All (CP).](images/asynchron-CAP.png)

There are only two options how a system can react to a request when
the network is partitioned.

* The system provides *a response*. In this case, the response can be
 wrong since changes have not reached the system. This is the *AP*
 case. The system is available. However, it might return another
 response than systems which have obtained newer information. Thus,
 there are inconsistencies.

* Alternatively, the system returns *no response*. This is the *CP*
 case. On the one hand, the system is not available when there is a
 problem, on the other hand, all systems always return the same
 responses and therefore are consistent as long as there is no
 network partitioning.

#### Compromises with CAP

In fact, you can make compromises. Let's take a system with five
replicas. When writing, each replica confirms that the data has
actually been written. When reading, several systems can be called to
find out the latest state of the data, if not all systems have
completed the replication yet.

Such a system with five replicas, in which one replica is read and
only the confirmation from one replica is waited for, focuses on
availability. Up to four nodes can fail without the system
failing. However, it does not guarantee high consistency. It is
possible that data is written to one node, the replication to the
other nodes takes some time and therefore an old value is read from
another node.

If the system with five replicas always waits for five nodes to be
confirmed and always
reads from five nodes, the data is always consistent. However, the
failure of a single node causes the system to become unavailable. A
compromise can
be to wait for confirmation of writes from three nodes and for reads
from three nodes. In this way, inconsistencies can still be ruled out
and the failure of up to two nodes can be compensated for.

#### CAP, Events and Data Replication

The CAP theorem actually considers data storage like NoSQL databases,
which achieve performance and reliability via replication. But
similar effects also occur when systems use events or data
replication. Ultimately, an event can be seen as a kind of data
replication across multiple microservices, although unlike the full
replication of data between nodes
of NoSQL databases, each microservice
can react differently to the event and may only use parts of the
data. A microservice that relies on asynchronous communication, events,
and data replication corresponds to an AP system. Microservices may
not have received some events yet, so the data may be
inconsistent. However, the system can process requests using local
data and is therefore available even if other systems fail. The CAP
theorem says that the only alternative is a CP
system. This would be consistent but not available. For example, it
could store the data in a central microservice which is accessible by
all. Then all microservices would receive the latest data. However,
if the central microservice fails, all other microservices
would no longer be available.

#### Are Inconsistencies Acceptable?

Thus, the inconsistency of an asynchronous system is inevitable unless
you want to give up availability. It is therefore important to know
the requirements for consistency, which requires some skill. Customers
want a reliable system. Data inconsistency seems to contradict
this. That's why it is important to know what happens when the data is
temporarily inconsistent and whether this really causes
problems. After all, the inconsistencies should usually disappear after
fractions of a second or a few seconds.
Besides, certain inconsistencies
can even be tolerable from a domain perspective. For example, if
goods are listed days before the first sale, inconsistencies are
initially acceptable and must only be corrected when the goods are
finally being sold.

If inconsistencies are not acceptable at all, asynchronous
communication is not an option. This means that synchronous
communication must be used with all its disadvantages. If the
tolerance for
temporarily inconsistent data is not known, this can lead to a wrong
decision regarding the communication variant.

#### Repairing Inconsistencies

In the simplest case, the inconsistencies disappear as soon as all
events have reached all systems. However, there may be exceptions. An
example: After registration, a customer receives an initial credit
balance. The event for the initial balance is received by a
microservice, but the microservice has not yet received the
registration of the user. The microservice cannot credit the initial balance because the customer has not been created in the system. If the
registration of the customer arrives later on, the initial balance
would have to be executed once again.

This problem can be solved when the order of events can be guaranteed.
In this case the problem will not arise in the first
place. Unfortunately, many solutions cannot guarantee the order of
events.

Event sourcing can also help. This allows the microservice to always
reconstruct its
state from the events. Therefore the state could
be discarded and recreated from the events as long as those are
available without any gaps.

It is also possible to extend the domain logic. In that case if the
event for the initial balance is received before the registration, a
new customer object is created. But the object is marked as invalid as
long as the data from the registration is missing. The rest of the
logic would need to handle such invalid customers. That might make the
business logic quite complex. Error might be hard to spot because now
there are so many states an object might have. Therefore this solution
should probably be avoided.

#### Guaranteed Delivery

In an asynchronous system, the delivery of messages can be guaranteed
if the system is appropriately implemented. The sender has the
messaging system confirm that it received the message. Afterwards, the
messaging system has the recipient of the message acknowledge the
receipt. However, if the recipient never picks up the data and thus
prevents delivery, the sender has an acknowledgement, but the message
still does not arrive at the recipient.
 
It is difficult to guarantee delivery when the recipient is anonymous.
In this case it is unclear who is supposed to receive the message and
whether there are any recipients at all who should get the
message. Therefore, it is also unclear who has to issue receipts.

#### Idempotency

If the messages are not acknowledged by the recipient, they are sent
again. When the receiving microservice processed the message, but was
unable to acknowledge the message due to a problem or a failure, the
recipient will receive the message a second time although it processed
the message already.

This is an *at least once strategy*. The messages are at least sent
once and in the described failure scenario more often.

Therefore, typically, one tries to design distributed systems in such
a way that the microservices are idempotent.  This means that a
message can be processed more than once, but the state of the service
does not change anymore. For example, when creating an invoice, the
*invoice* microservice can first check in its own database whether an
invoice has already been created. In this manner, only the first
time the message is received an invoice is created. If the message is
transfered again, it will be ignored.

#### One Recipient

In addition, it can be necessary that only one instance of a
microservice processes a message. For example, it would be incorrect
from a domain perspective when multiple instances of the *invoice*
microservice receive the order and all of them generate an
invoice. This would generate multiple invoices instead of one.  For
this, messaging systems have normally an option to send messages only
to a single recipient. This recipient then has to confirm the message
and process it. Such a communication type is termed
*point to point communication*.

Unfortunately, the rules for processing can be complex.  When changes
are made to customer data, parallel processing should be carried out
as far as possible to ensure high performance.  However, changes to
the data of a specific customer probably have to follow a sequence.
For example, it would not be good if changes to the billing address
are processed after the invoice has been written.  Then the
invoice would still contain the wrong address.  For this reason, it
may be important to guarantee the order of messages.

#### Test

Also with asynchronous microservices, the continuous delivery
pipelines must be independent to enable independent deployment.  To do
this, the testing of the microservices must be independent of other
microservices.

With asynchronous communication, a test can send a message to the
microservice and check whether the system behaves as expected.  Timing
can be difficult because it is not clear when the microservice has
processed the message and how long the test should wait for
processing.  The test can then check whether the microservice sends
the correct messages in response.  This allows very simple black box
tests, i.e. a test based on the interface without knowing about the
internal structure of the microservice.  In addition, such tests do
not place particularly high demands on the test environment. It just
needs to be able to transmit messages.  In particular, it is not
necessary to install a large number of other microservices in the test
environment.  Instead, the messages that other microservices send or
expect from the tested microservice can be the basis for the tests.

## 10.4 Advantages {#section-asynchron-vorteile}

Decoupling via events was presented in
[section 10.2](#section-asynchron-events). Such an architecture
achieves a high degree of decoupling.

Especially for distributed systems asynchronous communication has a number
of decisive advantages:

* When a communication partner fails, the message is sent later when
 the communication partner is available again. In this manner,
 asynchronous communication offers *resilience*, i.e. a protection
 against the failure of parts of the system.

* Delivery and also processing of a message can nearly always be
 *guaranteed*. The messages are stored for a long time.  At some point
 they will be processed. It can be ensured that they are processed,
 for example, by the recipients *acknowledging* the message.

In this manner, asynchronous communication solves challenges caused by
distributed systems.

## 10.5 Variations {#section-asynchron-variationen}

The two following chapters will introduce concrete technologies for
implementing asynchronous communication.

* [Chapter 11](#chapter-kafka) shows Apache Kafka as example for a
 message-oriented middleware (MOM).  Kafka offers the option to store
 messages for a very long time. This can be helpful for event
 sourcing. This feature distinguishes Kafka from other MOMs which are
 also good options for microservices.

* [Chapter 12](#chapter-atom) demonstrates the implementation of
 asynchronous communication with REST and the Atom data format. This
 can be helpful when MOMs are too much of an effort as additional
 infrastructure.

Asynchronous communication is easy to combine with frontend
integration (see [chapter 7](#chapter-frontend)) since both of these
integrations focus on different levels. Frontend and logic. However,
inconsistencies can easily be noticed during UI integration. When two
microservices simultaneously present their state on one web page,
inconsistencies can become apparent.  When the microservices implement
things of different domains, they will use different data and
therefore be rarely inconsistent.

However, a combination of asynchronous and synchronous communication
(see [section 13](#chapter-synchron)) should be avoided since
synchronous and asynchronous communication both start at the logic
level.  But even this combination might be sensible in special
scenarios. For example, synchronous communication can be necessary if
a response of a microservice is required immediately.

## 10.6 Conclusions {#section-asynchron-fazit}

Asynchronous communication should be preferred over synchronous
communication between microservices due to the advantages concerning
resilience and decoupling.  The only reason arguing against this is
inconsistency. Therefore, it is important to know exactly what the
requirements are, especially concerning consistency, in order to make the
technically correct decision. Choosing asynchronous communication has
the potential to elegantly solve the essential challenges of the
microservices architecture and should therefore be considered in any
case.

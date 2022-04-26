# 2 Micro and Macro Architecture {#chapter-mikro-makro}

Microservices provide much better
decoupling. Therefore, they help to 
modularize and isolate software modules (see
[section 1.2](#section-microservices-gruende)). However, microservices
are modules of a larger system. Therefore, they have to be integrated.
This poses a challenge for the architecture. On the one hand, it has
to ensure that the microservices can work together to form the overall
system.  On the other hand, the freedom of the microservices may not
be restricted too much since this would compromise their necessary
isolation and independence which are required for most of the benefits
of a microservice architecture.

For this reason, it is advisable to divide the architecture into a
micro and a macro architecture. The *micro architecture* comprises all
decisions that can be made individually for each microservice. The
*macro architecture* consists of all decisions which have to be made
at a global level and apply to all microservices.

{id="fig-mikro-makro-konzept"}
![Fig. 2-1: Micro and Macro Architecture](images/mikro-makro-konzept.png)

[Figure 2-1](#fig-mikro-makro-konzept) illustrates this idea On the
one hand, there is the overarching macro architecture which applies to
all microservices. On the other hand, there is the micro architecture
which deals with individual microservices so that each microservice
has its own micro architecture.

This chapter shows how to split the architecture in this way.

* First of all, the chapter conveys the *division of domain logic* into
  microservices and shows that *domain-driven design* and *bounded
  context* are great approaches for such a division.

* Then the chapter explains the decisions that are
  part of the *technical micro and macro architecture* and how a
  *DevOps model* affects these decisions.

* Finally, it addresses the question *who* divides the decisions into
  micro and macro architecture and creates the macro architecture.

## 2.1 Bounded Context and Strategic Design {#section-mikro-makro-bounded-context}

In regards to the domain architecture the concept of micro and macro
architecture has long been common practice. There is a macro
architecture which divides the domains into coarse-grained
modules. These modules are further divided as part of the micro
architecture.

For example, an e-commerce system can be divided into modules for
*customer registration* and *order process*. *Order process* can be further
divided into smaller modules such as, for instance, *data validation*
or *freight charge calculation*.  The internal architecture of the
*order process* module is hidden from the outside and therefore can be
altered without affecting other modules.  This flexibility to change
one module without influencing the other modules is one of the main
advantages of modular software development.

#### An Example for a Domain Architecture

{id="fig-mikro-makro-bounded-context"}
![Fig. 2-2: Example for a Split into Bounded Contexts](images/mikro-makro-bounded-context.png)

[Figure 2-2](#fig-mikro-makro-bounded-context) shows an example for
the division of a system into multiple domain modules.  In this
division each module has its own domain model.

* For *search* data such as descriptions, images or prices have to be
  stored for products. Important customer data are for instance the
  recommendations which can be determined based on the past
  orders.

* *Order process* has to keep track on what is in the shopping cart.  For
  products, only basic information is required such as name and
  price. Similarly, not too much data concerning the customer is
  necessary.  The most important component of the domain model of this
  module is the shopping cart.
  It is then turned into an order that has to be handed over and
  processed by the other bounded contexts.

* For *payment* the payment associated information like credit card
  numbers has to be kept for each customer.

* For *shipping* the delivery address is required information
  about the customer and size and weight are necessary information about the product.

This list reflects that different domain models are required for
the modules. Not only the data concerning customer and
product differ, but the entire model and also the logic.

#### Domain-driven Design: Definition

Domain-driven design (DDD) offers a collection of patterns for the
domain model of a system. For microservices, the patterns in the
area of strategic design are the most interesting. They describe how a
domain can be subdivided.  Domain-driven design offers many more
patterns which for instance facilitate the model of individual
modules. The original DDD book[^mikro-makro-ddd] provides a lot more
information. It introduced the term domain-driven design and
comprehensively describes DDD. The more compact 
book "Domain-driven Design Distilled"[^mikro-makro-ddd-distilled] focuses on design,
bounded context and domain events.
The [Domain-Driven Design
Reference](https://domainlanguage.com/ddd/reference/) is also by the
author of the original DDD book. It contains all DDD patterns but
without any additional explanation or examples.

[^mikro-makro-ddd]: Eric Evans: Domain Driven Design: Tackling Complexity in the Heart of Software, Addison Wesley, 2003, ISBN 978 0 32112 521 7

[^mikro-makro-ddd-distilled]: Vaughn Vernon: Domain Driven Design Distilled, Addison-Wesley Professional, 2016, ISBN 978 0 13443 442 1

[^mikro-makro-ddd-reference]: Eric Evans: Domain Driven Design: Tackling Complexity in the Heart of Software, Addison Wesley, 2003, ISBN 978 0 32112 521 7


##### Bounded Context: Definition

Domain-driven design speaks of a *bounded context*. Each domain model
is only valid in a bounded context.  Consequently, *search*, *order
process*, *payment* and *shipping* are such bounded contexts because they
each have their own domain model.

It would be conceivable to implement a domain model that comprises
multiple bounded contexts from the example. However, such a model
would not be the easiest solution. For instance, a price change
affects *search*, however, it must not result in a price change for
orders that have already been processed in *payment*.  It is easier to
only store the current price of a product in the bounded context
*search*, and to store in *payment* the price of the product in each
order which can also comprise rebates and other complex logic.
Therefore, the most simple design consists of multiple specialized
domain models which are only valid in a certain context.  Each domain
model has its own model for business objects such as customers or
products.

#### Strategic Design

The division of the system in different bounded contexts is part of
*strategic design* which belongs to the
practices of domain-driven design (DDD).  Strategic design
describes the *integration* of bounded contexts.

{id="fig-mikro-makro-strategic-design-grundlagen"}
![Fig. 2-3: Fundamental Terms of Strategic Design](images/mikro-makro-strategic-design-grundlagen.png)

[Figure 2-3](#fig-mikro-makro-strategic-design-grundlagen) shows the
fundamental terms of strategic design.

* The *bounded context* is the context in which a specific *domain
model* is valid.

* The bounded contexts depend on each other: 

Usually each bounded context is implemented by one team.
The *upstream* team can influence the success of the 
*downstream* team. However, the downstream team cannot influence the
success of the upstream team.
For example the team responsible for payment depends on the
information the order process teams provides. If data such as
prices or credit card numbers are not part of the order, it is
impossible to do the payment. However, the order process does not
depend on the payment to be successful. Therefore, the order
processing is upstream. It can make payment fail. Payment is therefore
downstream and also cannot make the order process fail.


#### Strategic Design Patterns

DDD describes in several patterns how exactly communication takes
place. These patterns do not only describe the architecture, but also
the cooperation within the organization.

* With the *customer/supplier* pattern, the supplier is upstream.  The
  customer is 
  downstream. However,
  the customer can factor their priorities into the planning of the
  upstream project. In
  [figure 2-4](#fig-mikro-makro-customer-supplier) payment
  uses the model of the order process. However, payment
  defines requirements for the order process. Payment can
  only be done successfully if the order process provides the
  required data. So payment can become a customer of the order
  process. That way their requirements can be included in the planning
  of the order process.

{id="fig-mikro-makro-customer-supplier"}
![Fig. 2-4: Customer / Supplier Pattern](images/mikro-makro-customer-supplier.png)

* *Conformist* means that a bounded context simply uses a domain
  model from another bounded context.  In
  [figure 2-5](#fig-mikro-makro-conformist)
  bounded context statistics and order process both use the same domain
  model. The statistics 
  are part
  of a data warehouse. They use the domain model of the order process
  bounded context and extract some information relevant to store in the data
  warehouse. However, with
  the *conformist* pattern the data warehouse team does not have a say
  in case of
  changes to the bounded context.
  So the data warehouse team could not demand additional information
  from the other bounded context. However, it is still possible that
  they would receive additional information out of altruism.
  Essentially the data warehouse team is not deemed important enough
  to get a more powerful role.

{id="fig-mikro-makro-conformist"}
![Fig. 2-5: Conformist: Domain Model Used in Other Bounded Context](images/mikro-makro-conformist.png)

* In case of an *anti-corruption layer* (ACL), the bounded
  context does not directly use the domain model of the other bounded
  context but it contains a layer for decoupling its own domain model from
  the model of the bounded context.  This is 
  useful in conjunction with *conformist* to generate a separate model
  decoupled from the other model.
  [Figure 2-6](#fig-mikro-makro-anti-corruption-layer) shows that
  the bounded context *shipping* uses an ACL at the interface to
  bounded context *legacy*
  so that both bounded contexts have their own independent domain
  models. That makes sure that the model in the legacy system does not
  affect the bounded context *shipping*. *Shipping* can implement a clean model 
  in its bounded context.

{id="fig-mikro-makro-anti-corruption-layer"}
![Fig. 2-6: Anti-corruption Layer with Conformist](images/mikro-makro-anti-corruption-layer.png)

* With *separate ways* the bounded contexts do not have a relation at
the software level although a relation would be conceivable.  Let's
assume that in
the e-commerce scenario a bounded context *purchasing* for the
purchase department is added. This bounded context could collect the
data for listing products. But it is implemented differently. With
*separate ways* the *purchasing*
would be separate from the remaining system. When the goods are
delivered, a user would use another bounded context like *listing*
to enter the
necessary data and list the products.  The purchasing causes the
shipping which in turn triggers the delivery and thereby triggers the user to
list the product with a different bounded context.  The shipping of the
products is an event in the real world. In the software, the systems
are separate. Consequently, the systems are independent and can be
evolved completely independently.

{id="fig-mikro-makro-separate-ways"}
![Fig. 2-7: Separate Ways](images/mikro-makro-separate-ways.png)

* *Shared kernel* describes a common core which is shared by multiple
bounded contexts. In [figure 2-8](#fig-mikro-makro-shared-kernel)
domain model order process and payment possess a shared kernel.
Data of 
a customer could be an example for such a scenario.  However,
*shared kernel* comprises shared  business logic and shared database
schemata and therefore should not be used in a microservices
environment.
It is an anti-pattern for microservices systems. But because DDD can
also be applied to deployment monoliths, there are still scenarios in
which a shared kernel makes sense.

{id="fig-mikro-makro-shared-kernel"}
![Fig. 2-8: Shared Kernel](images/mikro-makro-shared-kernel.png)

Some patterns are primarily useful in cases where more than one
bounded context has to be integrated.

* *Open host service* means that the bounded context offers a generic
   interface with several services. Other bounded contexts can
   implement their own
   integration with these services. This pattern is frequently found
   at public APIs
   in the Internet, however, it is also a possible alternative within
   an enterprise.

* *Published language* is a domain model that is accessible by all
bounded contexts. For example, this can be a standard format such as
EDIFACT for transactions between companies. But it is also possible to
define a data structure that is only used inside a company and
published e.g. in the Wiki.

These models can be used together. The *open host service* can use
*published language* for communication. For example the order process
might accept orders from external clients. Providing a specific
interface for each external client would be a lot of effort so there
is a generic open host service and a published language for
orders. Each external client can use this interface to submit orders
to the bounded context order process.

{id="fig-mikro-makro-open-host-published-language"}
![Fig. 2-9: Open Host Service and Published Language](images/mikro-makro-open-host-published-language.png)

#### Selecting Patterns

The choice of patterns has to be in line with the domain, the power
structure and the communication relationships between the teams.  When
the bounded context *payment* does not obtain the necessary data from
the bounded context *order processing*, the products can be ordered
but not payed. Therefore, the *customer/supplier* pattern is
an obvious choice here.  However, this is not a fact found in the
domain, but rather
a consequence of the power
structure, which in turn depends on the business model of the company.

Of course, the selected patterns influence the effort necessary for
coordination and therefore the degree of isolation between the teams.
They set the rules by which the teams have to work on the
integration.  Thus, a pattern like *customer/supplier* might not be
very desirable since it requires a lot of coordination. Still, it
might be the right solution depending on domain aspects.
It makes little sense to use a different pattern between *payment* and
the *order process* just to have less coordination. A different
pattern might make it impossible for the business to succeed.

#### Domain Events between Bounded Contexts

For the communication between bounded contexts, domain events can be
used. Ordering a shopping cart can be modeled as such an
event. This event is triggered by the bounded context *order
process* and
is received by the bounded contexts *shipping* and *payment* to
initiate shipping and invoicing of the order.  Events can be useful
for integrating bounded
contexts. [Section 10.2](#section-asynchron-events) discusses events
from a technical perspective.

Domain events are a part of the domain model. They represent something
that happened in the domain. That means they should also be relevant
to domain experts.

#### Bounded Contexts and Microservices

Bounded contexts divide a system by domains. They do not have to be
microservices. They can also be implemented as modules in a deployment
monolith. If the bounded contexts are implemented as microservices,
this results in modules which are very independent at the domain and
technical level. Therefore, it is sensible to combine the concepts of
microservices and bounded contexts.

The dependencies of the bounded contexts as part of strategic design
limit this independence. However, since the microservices are part of
a larger system, dependencies between the modules cannot be completely avoided.

#### Evolution

Over time, new functionalities might justify new bounded
contexts. Also it might become apparent that one bounded context
should really be split into two. That might be the case because new
logic is added to the bounded context or the team understand the
bounded context better. In such a case, new bounded context and
therefore new microservices might be created.

As mentioned in [section 1.2](#section-microservices-gruende), there
are two levels of microservices: a coarse-grained division by domain
and a more fine-grained division for technical reasons. It might
therefore happen that new microservices are created to e.g. simplify
scalability. If a microservice is spit in two, the resulting
microservices are smaller and therefore easier to scale. So such
reasons might also lead to a larger number of microservices.

## 2.2 Technical Micro and Macro Architecture {#section-mikro-makro-technisch}

Domain-driven design provides a domain macro architecture. Bounded
context and strategic design are patterns for this kind of
architecture. Microservices provide technological
isolation. Therefore, it is possible to extend the concept of micro
and macro architecture to technical decisions. For deployment
monoliths these decisions inevitably had to be taken globally.
So technical decisions can be made within the
framework of macro or micro architecture. However, some decisions have
to be part of the macro architecture. Otherwise no integrated
system will be created, but rather several non-integrated islands.

#### Micro or Macro Architecture Decisions

The decisions that can be taken either in the context of micro or
macro architecture include:

* The *programming language* can be defined uniformly for all
microservices in the macro architecture. But the decision can also be part of
the micro architecture. Then each microservice can
also be
implemented with a different language. The same 
applies to frameworks and infrastructure
such as application servers.  If these decisions are part of the *micro
architecture*, the best suited technology can
be used to solve the specific problems of each microservice.
A *macro architecture* definition is useful if a
company's technology strategy allows only certain technologies or if
only developers with knowledge in certain technologies should be
hired.

* The *database* can also be defined as a part of the macro
architecture or the micro architecture. At first glance, this decision
seems to be comparable to the decision concerning the programming
language. But databases are different. They store data. The loss of
data is usually unacceptable. Therefore, there must be a backup
strategy and
a disaster recovery strategy for a database. Setting these up
for many different databases can cause considerable effort. To
avoid too many different database, the database can be defined as part
of the *macro 
architecture* for all microservices. Even if the database is defined in
the macro architecture, multiple microservices must not
share a database schema. That would contradict the
bounded contexts (see
[section 2.1](#section-mikro-makro-bounded-context)). The domain
model in the database schema would be used by several
microservices. This would couple the microservices too strongly. Even
with a unified database the microservices must
have separate schemata in the database. Each
microservice can also have its own instance of the database. That has
an advantage. A
crash of one database causes only one microservice to fail. However,
the higher
effort involved, especially concerning operation, is an argument
against individual instances.

* If microservices have their own UI, the *look & feel* of
microservices can be a micro or macro architecture decision. Often a
system should have a uniform UI. Then the look & feel must be a
macro architecture decision. But sometimes a system has different
types of users, e.g. back office and customers. These types of users
have different requirements for the UI, which are often incompatible
with a uniform look & feel. Then the look & feel must be a
micro architecture decision to make sure each microservice can support
their group of users best. When defining the look & feel at the
macro architecture level, shared CSS and JavaScript are not enough to
ensure a common style of the UI of all microservices. Uniform
technical artifacts can be
used to implement very different types of user interfaces. Therefore,
a style guide must become part of the macro architecture. Often there
are concerns that microservices with separate UIs cannot provide a
consistent look & feel. But the UI can also diverge in a monolithic
system. Defining appropriate style guides and
artifacts is the only way to achieve a consistent look & feel for
large systems, regardless of the use of microservices.

* It may be necessary to standardize the *documentation*. The
documentation should be part of the micro architecture if the
the same team will build and maintain the microservice. Then they
can decide for themselves which documentation is necessary for the
long-term development. But of course, the decision about the
documentation can also be part of the macro architecture. A certain
level of documentation makes it easier to later hand the
microservice
over to another team. It may also be necessary to
document certain aspects of the microservices in a uniform manner. For
example, for security reasons, some systems need to keep track of
the libraries used in the microservices. In the case of a
security vulnerability in a specific library, it is then possible to
identify
which microservices need to be fixed. Standardized documentation
can also provide an overview of the system and the
dependencies between microservices.

#### Typical Macro Architecture Decisions

There are some decisions that must always be taken at the level of
macro architecture. Ultimately, all microservices together should
result in a coherent system. This requires some standards.

* The *communication protocol* of the microservices is a typical macro
architecture decision. Only if all microservices uniformly provide
e.g. a
REST interface or a messaging interface, can all
microservices communicate with each other and thus form a coherent
system. In addition, the data format must be standardized. It makes a
difference whether systems communicate with JSON or XML, for
example. The communication protocol could theoretically be a different
one for each communication channel between two microservices and thus
a micro architecture decision.  In the end, however, there is then no
longer a coherent system. Actually, it has disintegrated into islands
that communicate with each other in different ways.

* With *authentication*, a user proves that he or she has a certain
identity.  This can be done with a password and a user name, for
example. Since it is unacceptable for the user to re-authenticate with
every microservice, the entire microservice system should use a single
authentication system. The user then enters a user name and password
once and can then use any microservice.
  
* Integration testing technology is also a typical macro architecture
  decision. All microservices must be tested together. So they have to
  run together in an integration test. The macro architecture must
  define the necessary prerequisites for this.

#### Typical Micro Architecture Decisions

Certain decisions should be taken for each microservice individually.
They are therefore typically part of the micro architecture.

* The *authorization* of the user determines what a user is allowed to
do. Like authentication, authorization belongs to the area of
security. The authorization should be done in the respective
microservice. Authorization is closely linked to domain logic. Which
user is allowed to initiate a certain action is part of the domain
logic and therefore belongs to the microservice like the other domain
logic. Otherwise, the domain logic would be implemented in a
microservice itself, but the decision about which part of the domain
logic is available to a user would be made centrally. This makes no
sense, especially with complex rules. For example, if orders up to a
certain upper limit can be triggered by certain users, authorization,
concrete upper limits, and possible exceptions belong to the
microservice *order*. Authentication assigns the user roles that can
be used in authorization. For example, a microservice can define which
actions a user with the role *customer* can trigger and which actions
a user with the role *call center agent* can trigger.

* The *testing* can be different for
each microservice. Even the tests are ultimately part of the domain
logic. In addition, there may be different non-functional requirements
for each microservice.  For example, one microservice can be
particularly performance-critical, while another can be more
safety-critical. These risks must be covered by an individual
focus in the tests.

* Since the tests can be different, the *continuous delivery pipeline*
is also different for each microservice. It must include the relevant
tests. Of course, the technology for the continuous delivery pipeline
can be standardized.  For example, each pipeline can use a tool like
 Jenkins. What happens in the respective pipelines, however,
depends on the respective microservice.

The following table shows the typical micro and macro architecture decisions:

| Micro or Macro       | Micro Architecture           | Macro Architecture |
|----------------------|------------------------------|--------------------|
| Programming Language | Continuous Delivery Pipeline | Communication Protocol |
| Database             | Authorization                | Authentification |
| Look & Feel          | Tests of the Microservice in Isolation        | Integration Tests |
| Documentation        |                              | |

## 2.3 Operation: Micro or Macro Architecture? {#section-mikro-makro-betrieb}

Some decisions in the area of micro and macro architecture influence
mostly the operation of the applications. These include:

* In *configuration*, it is necessary to define an interface with
which a microservice can obtain its configuration parameters. This
includes both technical parameters (such as thread pool sizes) and
parameters for the domain logic. For example, a microservice can get these settings
via an environment variable or read them from a configuration
file. This decision only affects how the microservice obtains the
configuration. The decision how to store and generate the
configuration data is
independent of this. The data can be stored in a database, for
example. Either configuration files or environment variables can be
generated from the data in the database. Note that the information on which
computer and under which port a microservice can be reached, does not
belong to the configuration, but to the service discovery (see
[section 13.3](#section-synchron-herausforderungen)). Configuring
passwords or certificates is also a challenge that can be solved with
other tools. To do this, [Vault](https://www.vaultproject.io/) is a
good choice, as this information must be stored in a particularly
secure way and must be visible to as few employees as possible in order to
prevent unauthorized access to production data.
  
* *Monitoring* is about the technology that tracks metrics (see
[chapter 20](#chapter-monitoring)). Metrics provide information about
the state of a system. Examples include the number of requests
processed per second or business metrics such as revenue. The question
of which technology is used to track the metrics is independent of
which metrics are captured. Every microservice has different
interesting metrics, because every microservice has different
challenges. For example, a microservice can be under very high
load. Then performance metrics are useful.

* *Log analysis* defines a tool for managing logs (see
[chapter 21](#chapter-log-analyse)). While logs were originally stored
in log files, they are now stored on specialized servers. This makes
it easier to analyze and search the logs, even with large amounts of
data and many microservices. In addition, new instances of a
microservice can be started when the load increases and can be deleted again once
the load decreases. In this case, the logs of
this microservice instance should still be available, even if the
microservice was deleted long ago due to a decreasing load.
If the logs are just stored on a local device, the logs would be gone
after the microservice has been deleted.

* *Deployment technology* determines how the microservices are
  rolled out. For example, this can be done with Docker images (see
  [chapter 5](#chapter-docker)), Kubernetes Pods (see
  [chapter 17](#chapter-kubernetes)), a PaaS (see
  [chapter 18](#chapter-paas)) or installation scripts.

These decisions define how a microservice behaves from an operation
point of view. Typically, these decisions are either all part of the
macro architecture or all part of the micro architecture.

#### Operation Macro Architecture with Separate Operations Teams

Whether decisions in the area of operation belong to micro or macro
architecture depends on the organization. For example, a team can
develop microservices, but bear no responsibility for their
operation. The operations team is responsible for the operation of
all microservices. In this scenario, decisions for operation must be
made at the level of macro architecture. It is generally unacceptable
for the operations team to have to learn a different approach for the
operation of each microservice, especially since the number of
microservices is much larger than the number of
deployment monoliths would be for the same project.

Another reason for a macro architecture decision for the operation of microservices is that
individual solutions in this area hardly bring any advantages. While
a programming language or framework can be more or less suitable for a
particular problem, the same applies to a much lesser extent to technologies in the field of operation.

#### Standardize only Technologies!

When these decisions are made at the level of macro architecture, they
standardize only the technologies. Which configuration parameters,
which monitoring metrics, which log messages and which deployment
artifacts a microservice has, is definitely a decision at the level of
the individual microservice. The independent deployment must also be
retained as a core feature of the microservices. This means that it
has to be possible to independently change the configuration
parameters for each microservice in order to be able to make
adjustments to the configuration when a new deployment takes place.

#### Testing the Operation Macro Architecture

Adherence to the macro architecture rules can be checked with tests.
The microservices are deployed in an environment. The tests check
whether the rules for uniform deployment are adhered to. The test then
verifies whether the microservice delivers metrics and log information
in the defined way. Something similar is also possible in regards to
configuration.

The test environment for these tests should be very minimalistic and
should not contain any other microservices or a database. In this
manner, the microservice is tested in an environment in which it
cannot possibly work. When such a situation occurs in production, it
is particularly important that the microservice provides logs and
metrics to analyze potential problems. In this way, the test also
checks the resilience of the microservice.

#### "You Build It, You Run It": Operation as Micro Architecture

There is a form of organization in which operational aspects have to
be part of the micro architecture. If the same team is to develop and
operate the microservice, it must also be able to choose the
technology. This approach can be described as "You build it, you run
it". The teams are each responsible for a microservice, for its
operation *and* development. You can only expect this level of
responsibility from the team if you allow them to choose their own
technologies.

#### Operation as a Whole is Micro or Macro Architecture

So decisions for operation can be taken either at the level of
micro or macro architecture. Making operation decisions part of the macro architecture is useful if there is
a separate operations team, while a "You build it, you run it"
organization must make these decisions at the level of micro
architecture (see [figure 2-10](#fig-mikro-makro-betrieb)).

{id="fig-mikro-makro-betrieb"}
![Fig. 2-10: Depending on the organization, operation is part of the micro or macro architecture](images/mikro-makro-betrieb.png)

## 2.4 Give a Preference to Micro Architecture!

There are good reasons for making as many decisions as possible at the level
of micro architecture:

* If only *few points* are specified in the macro architecture, this
helps with focusing. Many teams have failed when trying to implement a far-reaching
unification in a complex project or IT landscape. If there are few
macro architecture
rules, the chances increase that they are actually successfully implemented.

* The rules should be *minimal*. For instance, a macro architecture
rule can define the monitoring technology. However, it is not
necessary to standardize how the metrics are measured in the
application. After all, it is only important that the metrics are
created. How this happens is irrelevant. So the macro architecture
rule should only define a protocol for transferring metrics, but leave
the selection of the library for creating and transferring metrics
to the micro architecture. In this way, the teams can choose the most
appropriate
technologies.
  
* The macro architecture rules have to be *consequently enforced*.
 For example, when the metrics are not generated, the microservice
 simply cannot be brought into production by the operations team.
 So it is important to get rid of all macro architecture elements that
 are actually not needed.
 
* In addition, *independence* is an important goal of
microservices. Too many macro architecture rules run counter to this
goal, as they hinder the independence of the teams through central
control.

* Complying with macro architecture should be in the *self-interest* of the teams
responsible for the microservices. Violations of macro architecture
usually mean that microservices cannot go into production
because, for example, operations cannot support them.

#### Evolution of Macro Architecture

At the beginning of a project, restrictive rules may initially apply.
For example, a single programming language and a fixed stack of
libraries can be defined. This reduces learning effort and
operating costs.  Over the duration of the project, more programming
languages and libraries can then be allowed, for example, to introduce
modern technologies. This leads to a more heterogeneous
system. But a heterogeneous system is certainly preferable to updating
all microservices at once since such updates entail a high risk.

#### Best Practices and Advice

In addition to mandatory macro architecture rules, recommendations and
best practices are of course conceivable. However, they do not have to
be enforced, but are optional for every microservice.

In the end, the goal of macro architecture is to create freedom rather than letting teams
run into the open knife. Advice and references to best practices are
therefore definitely good additions.

## 2.5 Organizational Aspects {#section-mikro-makro-organisation}

There is a connection between decision and responsibility. Whoever
makes a decision takes responsibility. Therefore, if the decision for
a technology for metrics is made as part of the macro architecture,
then the macro architecture group must take responsibility, for
example, if this technology proves unsuitable in the end because it does not cope
with the amount of data. If the responsibility for monitoring
microservices is completely transferred to the teams, then the teams
must also be allowed to select a technology.

#### Uncontrolled Growth?

The freedom regarding micro architecture can lead to a
a huge number of used technologies. But this is not necessarily the
case. If all teams have had good experiences with a particular
monitoring technology, then a new microservice will most likely be
monitored with the same tool. Using another tool would require a
great deal of effort. Other options are only evaluated and used if the
tool used so far is not sufficient. So, even without a macro
architecture rule,
there is standardization if uniform decisions bring advantages for
the teams. The prerequisite for this is, of course, an exchange
between the teams about best practices and about the question which technologies work and which do not.

#### Who Defines Macro Architecture?

Macro
architecture restricts the freedom of the teams when it comes to
implementing the microservices. This can be counteracted by having a
committee define the macro architecture which consists of one member
of each team. However, it is possible that the committee may become
too large to work effectively. With ten teams, the team would have ten
members and effective work is then hardly possible. You can reduce the
number of members by excluding teams or sending individual members as
representatives for multiple teams.

Unfortunately, the team members are often too focused on their own
microservices to be sufficiently interested in the overall picture of
macro architecture.  In a way, that is a good thing because they
should focus on their microservice and that is what they do.

The alternative is to have an independent architecture committee decide on macro
architecture, which is staffed by architects who do not belong to the
teams. In such a
scenario it is important that this body really has the goal of
supporting the teams in their development of microservices and of
moderating decisions rather than forcing them upon the teams.
The most important work takes place in the
teams. Therefore, the macro architecture should support the teams and
not hinder them. Collaboration between the architecture committee and
the teams can also be improved by the members of the architecture
committee working at least partly in the teams.

With an an independent architecture committee, it is important
that the members of the committee are integrated and
interested in the developed system. The specific domain and
business requirements should never be forgotten. An important part of
the work on the architecture is to understand the stakeholder and make
sure that their goals are supported by the architecture.

#### How to Enforce?

The need for macro architecture should be understandable because it
ensures that the entire
system can be developed and operated. To enforce the macro
architecture,
the reasons for each rule
should be documented. This avoids discussions because the reasoning
behind the rules is
then comprehensible. For example, certain macro architecture rules
may be necessary to allow the software to be brought into production by
the operations team or to make sure that compliance rules are
followed.

So it's not so much about getting rules enforced as it is about
promoting macro architecture and conveying the ideas and reasons for
macro architecture. If there are good reasons for changing the macro
architecture, it might indeed rather need to be
improved instead of simply enforced.

#### Testing Conformance

It is possible to test the conformance to the macro architecture in
some cases. For example, it is possible to deploy a microservice
and then check its log output and metrics. This ensure that
deployment, logging and monitoring conform to the defined macro
architecture.

Such a test is a black box test i.e. the test checkd the behavior of
the microservice from the outside. The benefit of this approach is
that it does not limit the free choice of technology for implementing
microservices and it does not enforce unnecessary standard e.g. for
specific frameworks. Therefore testing for the conformance on the code
level does not make a lot of sense.

## 2.6 Independent Systems Architecture Principles (ISA)

Micro and macro architecture are fundamental to the idea of
microservices. However, it is hard to understand why there should be
two levels of architecture.

[ISA](http://isa-principles.org) (Independent Systems Architecture) is
the term for a collection of fundamental principles for microservices.
It is based on experiences with microservices that were gained in many
different projects.

The name already suggests that these principles aim to build software
out of independent systems. Macro and micro architecture are very
important for this goal. A minimal macro architecture leaves a lot of
freedom to the level of the micro architecture. This makes the systems
independent. Technical decisions can be made for each system without
influencing the other systems.

ISA defines the term micro and macro architecture. Also the principles
explain what the minimum requirements for macro and micro architecture
are.

#### Conditions

*Must* is used for principles when they absolutely have to be adhered to.
*Should* describes principles which have many advantages but do not strictly
have to be followed.

#### Principles

1. The system must be divided into *modules* which offer
  *interfaces*.  Accessing modules is only possible via
  these interfaces.  Therefore, modules may not depend directly on the
  implementation details of another module such as the data model in a
  database.

2. The system must have two clearly separated levels of architectural decisions:
  * *Macro architecture* comprises decisions which concern all modules.
  All further principles are part of the macro architecture.
  * *Micro architecture* comprises those decisions which can be made
  differently for each individual module.

3. Modules must be *separate processes, containers or virtual
  machines* to maximize independence.

4. The choice of integration and communication options must be limited
   and standardized for the system. The integration might be done with
   synchronous or asynchronous communication, and/or on the UI
   level. Communication must use a limited set of protocols like
   RESTful HTTP or messaging. It might make sense to use just one
   protocol for each integration option.

5. *Metadata*, e.g. for *authentication*, must be
   standardized. Otherwise the user would need to log in to each
   microservices. This might be done using e.g. a token that is
   transferred with each call / request. Other examples might include
   a trace id to track a call and its dependent calls through the
   microservices.

6. Each module must have its *own independent continuous delivery
  pipeline*. Tests are part of the continuous delivery pipeline so
  that the tests of the modules have to be independent, too.

7. *Operation* should be standardized. This comprises configuration,
  deployment, log analysis, tracing, monitoring, and alarms. There can
  be exceptions from the standard when a module has very specific
  requirements.

8. *Standards* for operations, integration, or communication should be
   enforced on the interface level. For example, the communication
   protocol and data structures could be standardized to a specific
   JSON payload format exchanged using HTTP, but every module should
   be free to use a different REST library/implementation.

9. Modules have to be *resilient*. They may not fail when other
  modules are unavailable or when communication problems occur. They
  have to be able to shut down without losing data or state. It has to
  be possible to move them to other environments (server,
  networks, configurations etc.) without the module failing.

#### Evaluation

The ISA principles are not just a great guideline for building
microservices. They also explain why macro and micro architecture are
so important. Also ISA explains the structure of the book.

* The *first* ISA principle just states that a system must be build from
modules. This is common knowledge.

* The *second principle*
defines two level of architecture: macro and micro architecture. These
are the terms defined in this chapter, too.

* In a deployment monolith, most of the decisions will be on the macro
architecture level. For example a deployment will be written in one
programming language so the programming language has to be a decision
on the macro architecture level. The same is true for frameworks and
most of the other technologies. To actually create a meaningful
opportunity to make more decision on the micro architecture, each
module must be implemented in a separate container as *principle three*
states.
So ISA says that the reason why microservices run in containers is the
additional technological freedom that cannot be achieved in a
deployment monolith. Therefore microservices add more independence and
decoupling to
the architecture.
An approach where each microservice is a WAR and all run together in
one Java application server does not fit this principle. Actually, the
compromise concerning the free choice of technology and the robustness
is so high that this approach in fact usually does not make a lot of
sense.
Because decoupling is so important, ISA and microservices actually
provide fundamental improvements to modularization.


* While the goal of ISA is to create a minimal macro architecture, some
decisions still need to be made on the macro level. This is what the rest
of the principles explain. As a start, *principle four* states that
integration and communication must be standardized. That is the reason
why [part II](#chapter-techstacks) discusses technology stacks for
integration and communication. The decision to use a specific
technology for integration and communication influences all modules
and must therefore be done on the macro architecture level. It is
therefore a very important decision in microservices system.
Without a common integration approach and communication technology, it
is hard to consider the system a system and not just a few services
that cannot even really communicate with each other.

* *Principle five* states that metadata for tracing and authentication must
be standardized. Such metadata must be transferred between the
microservices and must therefore also be a part of the macro
architecture. For that reason [Chapter 22](#chapter-tracing) discusses
tracing in more detail. However, the book does not discuss security
aspects of microservices. Therefore metadata for authentication is not
discussed in this book.

* *Principle six* (independent deployment pipelines) extends the idea of
independent deployment as the definition of microservices from
[section 1.1](#section-microservices-definition). [chapter
16](#chapter-plattformen) explains platforms for the deployment of
microservices. [Chapter 17](#chapter-kubernetes) discusses Kubernetes
as a concrete example and [chapter 18](#chapter-paas) the PaaS Cloud
Foundry.

* *Principle seven* says that operations of microservices should be
standardized. It is not in all cases necessary to standardize
operations. With a separate operations department, standardization is
the only way to handle a large number of microservices. However, with
a "you build it - you run it" organization, standards are not
necessary as each time operates their microservices. Actually a
standardized operations approach might not fit all microservices. In
that case the teams will need to come up with their own operations
technologies. A standard makes little sense then.

* Principle eight* states that standards should only be defined on the
interface level. The technologies discussed throughout [part
II](#chapter-techstacks) can be used in this way. They provide
interfaces and client libraries for all commonly used programming
languages.

* *Principle nine* talks about resilience. This books focuses on asynchronous
communication which makes resilience easier. If a microservice fails,
message will be transferred later but it will not cause a
failure. Also the chapters about synchronous communication show how
systems can be resilient even if using synchronous communication.

Therefore, the ISA principles represent a good summary of the
principles introduced in this chapter i.e. a division between micro
and macro architecture as the main benefit of microservices. ISA also
explains why the rest of the book focuses on integration and
communication technologies and technologies for operations. These are
the fields that a macro architecture has to cover. Therefore these
decisions are very important and also hard to change as they influence
all microservices in the system.


## 2.7 Variations {#section-mikro-makro-variationen}

In the domain macro architecture, strategic design and domain-driven
design are ultimately unrivaled as approaches. However, the bounded
contexts depend on the specific project. Identifying the right bounded
contexts is a central challenge when designing the architecture of a microservices system.

The technical micro and macro architecture also has to be devised for
each project. This depends on many factors:

* Organizational aspects such as a *DevOps organization* or having a
separate operations team have an influence.

* In addition, *strategic technology decisions* can play a role.

* Even the *hiring policy* can be a factor. Eventually, there have to
  be experts available for the technologies who can work in the teams.

#### More Complex Rules

In reality, the rules of micro and macro architecture are often more
complex.  For example, a whitelist can exist for the programming
language. In addition, there can be a procedure for adding more
programming languages to the whitelist, for example, via a committee
for taking such decisions. And finally, there can be a general
limitation to programming languages that run on the JVM (Java Virtual
Machine) because there is a lot of experience with it in production.

Such a rule has elements of a macro architecture decision. There is a
whitelist and a restriction to JVM languages. At the same time, it
also has micro architecture elements. After all, a team can select one
of the programming languages from the whitelist and even extend the
whitelist.

At the end of the day, therefore, there are often rules for every
point in practice that allow the teams and microservices a certain
amount of leeway.  These rules are not pure micro architecture rules
and not pure macro architecture rules, but lie somewhere in between.

#### Experiments

The approach for defining micro and macro architecture can look like this:

* Consider a project you are familiar with. Look at its domain model.

  - Would a division into multiple domain models and bounded contexts
      make the system
      easier?

  - In how many bounded contexts would you split the system?  Typical
	projects comprise about ten bounded contexts. However, the exact
	number will vary of course for each individual project.

  - Determine the use cases which the system implements. Group use
    cases and analyze whether these use cases can be addressed by a
    domain model. In that case these use cases form a bounded context
    in which the domain model is valid.

  - Is a further division for technical reasons sensible?  The
    technical reasons can comprise independent scalability or security
    (see also "Two Levels of Microservices" in
    [section 1.2](#section-microservices-gruende)).
	
* In this chapter areas such as programming language and DevOps are
  addressed which can belong either to micro or macro
  architecture. Define for your project whether the individual
  decision should be part of micro or macro architecture.

* Work out at least one of the decisions in more detail. For example,
  there could be a whitelist of programming languages, or in fact
  only one programming language might be allowed that can be used by
  all microservices. A procedure for extending the whitelist is also
  conceivable.

## 2.8 Conclusion {#section-mikro-makro-fazit}

Microservices and self-contained systems allow architecture decisions
to be made individually for each microservice. If decisions can
actually be different for each microservice, then they are part of the
micro architecture. The macro architecture, on the other hand,
contains those parts of the architecture that apply uniformly to all
microservices. The division into these two levels gives the individual
microservices freedom, while at the same time ensuring the integrity
of the overall system.

Decisions at the micro architecture level are better suited to the
self-organization of teams and use the technical freedom offered by
microservices. Even if decisions are part of the micro architecture, a
standardization can still result because using the technologies other
teams have already made positive experiences with decreases the risk
the teams take and allows for the use of synergies.

In any case, decisions must be made explicitly. The teams must
consciously deal with the macro architecture and the freedoms in micro
architecture. Micro and macro architecture form a trade-off, which
can be different in each project.

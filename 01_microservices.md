# 1 Microservices {#chapter-microservices}

This chapter provides an introduction to *microservices* and discusses:

* [Advantages](#section-microservices-gruende) and
  [disadvantages](#section-microservices-herausforderungen) of
  microservices to enable the reader to evaluate the applicability and
  usefulness of this architecture for a specific project.

* The discussion of benefits explains which problems 
  microservices can solve and how this architecture can be adapted for different
  scenarios.
  
* The discussion of disadvantages illustrates where technical
  challenges and risks lie and how these can be dealt with.

* Recognizing benefits and disadvantages is critical for technology
  and architecture decisions since those have to be aimed at
  maximizing benefits and reducing disadvantages.

## 1.1 Microservices: Definition {#section-microservices-definition}

Unfortunately, there is no universally acknowledged definition for the
term *microservice*.  In the context of this book the following
definition will be used:

A> Microservices are independently deployable modules.

For example, an e-commerce system can be divided into modules for
ordering, registration and product search. Normally, all of
these modules would be implemented together in one application. In
this case, a change in one of the modules can only be brought into
production by bringing a new version of the entire application with
all its modules into production.  However, when the modules are
implemented as microservices, the ordering process cannot only be
changed independently of the other modules, but it can even be brought
into production independently.

This speeds up deployment and reduces the number of necessary tests
since only a single module needs to be deployed.  Due to this greater
level of decoupling, a large project can be turned into a number of
small projects. Each project is in charge of an individual microservice.

To achieve this at the technical level, every microservice has to be
an independent process. An even better solution for decoupling
microservices is to provide an independent virtual machine or Docker
container for each microservice. In that case, a deployment will
replace the Docker container of an individual microservice with a new
Docker container, then start the new version and direct requests to
this new version. The other microservices will not be affected if such
an approach is used.

#### Advantages of this Microservice Definition

This definition of microservices as independently deployable modules has 
several advantages:

* It is very *compact*.

* It is very *general* and covers all kinds of systems which are
  commonly denoted as microservices.

* The definition is based on *modules* and thus on an old,
  well-understood concept.  This allows one to adopt many ideas concerning
  modularization.  Besides, this definition highlights that
  microservices are part of a larger system and cannot function all on
  their own. Microservices necessarily have to be integrated with
  other microservices.

* The independent deployment is a feature which creates numerous
  [advantages](#section-microservices-gruende) and is therefore very
  important.  Thus, the definition inspite of its brevity explains
  what the most *essential feature* of a microservice really is.

#### Deployment Monolith

A system which is not made up of microservices can only be deployed in
its entirety. Therefore, it is called a *deployment monolith*.  Of
course, a deployment monolith can be divided into modules.  The term
*deployment monolith* does not make a statement about the internal
structure of the system.

#### Size of a Microservice

The given definition of microservices does not say anything about the
size of a microservice. Of course the term *microservice* suggests
that especially small services are meant. However, in practice,
microservices can vary hugely in size.  Some microservices keep an
entire team busy while others comprise only a few hundred lines of
code. Thus, the size of microservices, even though it figures as part
of the name, is ill suited to be part of the definition.

## 1.2 Reasons for Microservices {#section-microservices-gruende}

There are a number of reasons for using microservices.

#### Microservices for Scaling Development

One reason for the use of microservices is the easy scalability of
development.  Large teams often have to work together on complex
projects. With the help of microservices, the projects can be divided
into smaller units which can work largely independently of each other.

* For example, the teams responsible for an individual microservice
  can make most technology decisions on their own.  When the
  microservices are delivered as Docker containers, each Docker
  container only has to offer an interface for the other containers.
  The internal structure of the containers does not matter as long as
  the interface is present and functions correctly. Therefore, it is
  for example irrelevant in which programming language a microservice
  is written. Consequently, the responsible team can make this
  decision on its own. Of course, e.g. the selection of programming
  languages can be restricted in order to avoid increased complexity.
  However, even if the choice of programming language in a project has
  been restricted, a team can still independently use an updated 
  library with a bug fix for their microservice.
  
* When a new feature only requires changes in one microservice, it
  cannot only be developed independently, but it can also be brought
  into production on its own. This allows the teams to work on features completely
  independently.

Thus, with the help of microservices, teams can act independently in
regards to domain logic and technology. This minimizes the
coordination effort required for large projects.

#### Replacing Legacy Systems

The maintenance of a legacy system is frequently a challenge
since the code is often badly structured and changes can often
not be checked by tests. In addition, developers might have to deal with
outdated technologies.

Microservices help when working with legacy systems since the existing code
does not necessarily have to be changed. Instead new microservices can replace
parts of the old system. This requires an integration between the old system and the
new microservices -- for example via data replication, REST,
messaging, or at the level of UI. Besides, problems such as a uniform
single sign on for the old system and the new microservices have to be solved.

But then the microservices are very much like a greenfield project.
No pre-existing codebase has to be used. In addition, a
completely different technology stack can be employed. This immensely
facilitates work compared to having to modify the legacy code
itself.

#### Sustainable Development

Microservice-based architectures promise that systems remain maintainable even
in the long run.

An important reason for this is the replaceability of
microservices. When a microservice cannot be maintained
any longer, it can be rewritten. Compared to changing a deployment
monolith, this entails less effort since the microservices are much
smaller.

However, it is difficult to replace a microservice on which numerous
other microservices depend since changes might affect the other
microservices. Thus, to achieve replaceability the dependencies
between microservices have to be managed appropriately.

Replaceability is a great strength of microservices. Many developers
work on replacing legacy systems. However, when a new system is
designed, the question how to replace this system once it has turned
into a legacy system is asked much too rarely. Microservices with
their replaceability provide an answer.

To achieve maintainability, the dependencies between the microservices
have to be managed in the long term. Classical architectures often
have difficulties at this level. A developer writes some new code. In
doing so, she/he might unintentionally introduce a new dependency between two
modules which in actual fact had been forbidden in the architecture.
Typically, the developer does not even notice the mistake because
she/he only pays attention to the code level of the system and not to
the architectural level.  Often, it is not immediately clear
which module a class belongs to. So it is also unclear which module
the developer just
introduced a dependency to. In this manner, more and more dependencies
are introduced over time.  The originally designed architecture is more and
more violated culminating in a completely unstructured system.

Microservices have clear boundaries due to their interface —
irrespective of whether the interface is implemented as REST interface
or via messaging. When a developer introduces a new dependency to such
an interface, he/she will notice this since the interface has to be
called appropriately. For this reason, it is unlikely that architecture
violations will occur at the level of dependencies between
microservices.  The interfaces between microservices are in a way
architecture firewalls since they prevent architecture violations. The
concept of architecture firewalls is also implemented by architecture
management tools like
[Sonargraph](https://www.hello2morrow.com/products/sonargraph),
[Structure101](http://structure101.com/), or
[jQAssistant](https://jqassistant.org/).  Advanced module concepts can
also generate such a firewall. In the Java world,
[OSGi](https://www.osgi.org/) limits access and visibility between modules.
Access can even be restricted to individual packages or
classes.

Therefore, individual microservices remain maintainable. If the code
of a microservice is unmaintainable, it can just be replaced. That
would not influence any of the other microservices.
The architecture
at the level of dependencies between microservices remains
maintainable, too.
Developers cannot unintentionally add
dependencies between microservices. Therefore, microservices can
ensure a high architecture quality in the long term both inside each
microservices and between the microservices. Thus microservices enable
sustainable development where the speed of change does not decline
over time.

#### Continuous Delivery

[Continuous delivery](http://continuous-delivery-book.com/) is an
approach where software is continuously brought into production with
the help of a continuous delivery pipeline. The pipeline brings the
software into production via the different phases (see
[figure 1-1](#fig-microservices-continuous-delivery-pipeline)).

{id="fig-microservices-continuous-delivery-pipeline"}
![Fig. 1-1: Continuous Delivery Pipeline](images/microservices-continuous-delivery-pipeline.png)

Typically, the software is compiled, and unit tests and a statical code
analysis are performed in the commit phase.  In the acceptance test
phase, automated tests assure the correctness of the software
in regards to domain logic. Capacity tests check the performance
at the expected load. Explorative tests serve to perform not yet
considered tests or to examine new functionalities. In this manner,
explorative tests can analyze aspects which are not yet covered by
automated tests. In the end the software is brought into production.

Microservices represent independently deployable modules. Therefore each microservice
has its own continuous delivery pipeline.

This facilitates continuous delivery.

* The continuous delivery pipeline is
  significantly *faster* since the deployment units are
  smaller. Consequently, deployment is faster. Continuous delivery
  pipelines contain many test stages. The software has to deployed in
  each stage. Faster deployments speed up the tests and therefore the
  pipeline. The tests are also faster since
  they need to cover fewer functionalities.  Only the features in
  the individual microservice have to be tested while in case of a
  deployment monolith the entire functionality has to be tested due
  to possible regressions.

* Building up a continuous delivery pipeline is *easier* for microservices. Setting up
  an environment for a deployment monolith is complicated. Most of the
  time powerful servers are required. In addition, third
  party systems are frequently necessary for tests.  A microservice
  requires less powerful hardware. Besides, not so many third party
  systems are needed in the test environments. However, running all
  microservices together in one integration test can cancel out this
  advantage. An environment suitable for running all
  microservices would require powerful hardware as well as an
  integration with all third party systems.

* The deployment of a microservice poses a *smaller risk* than the
  deployment of a deployment monolith. In case of a deployment
  monolith the entire system is deployed anew, in case of a
  microservice only one module.  This will cause less problems since
  less of the functionality is being changed.

In summary, microservices facilitate continuous delivery. Just their better
support of continuous delivery can already be reason enough to
migrate a deployment monolith to microservices.

However, microservice architectures can only work when the
deployment is automated. Microservices increase the number of
deployable units substantially compared to a deployment monolith. This
is only feasible when the deployment processes are automated.

Really independent deployment means that the continuous delivery
pipelines have to be completely independent. Integration tests
conflict with this independence. They introduce dependencies between
the continuous delivery pipelines of the individual
microservices. Therefore, integration tests have to be reduced to the
minimum. Depending on the type of communication there are different
approaches to achieve this for
[synchronous](#section-synchron-herausforderungen) and
[asynchronous communication](#section-asynchron-herausforderungen).

#### Robustness

Microservice systems are more robust. When a memory leak exists
in a microservice, only this microservice will crash. The other
microservices keep running. Of course, they have to compensate the
failure of the crashed microservice. This is called resilience. To
achieve resilience, microservices can for example cache values and use
them in case of a problem.  Alternatively, there might be a fallback
to a simplified algorithm.

Without resilience, the availability of a microservice system might be
a problem.  It is likely that a microservice fails. Due to the
distribution into several processes many more servers are involved in
the system.  Each of these servers can potentially fail.
Communication between microservices occurs via the network. The
network can also fail.  Therefore, microservices need to implement
resilience to achieve robustness.

The [section about Hystrix](#section-netflix-hystrix) illustrates how
resilience can concretely be implemented in a synchronous
microservice system.

#### Independent Scaling

Most of the time it is not required to scale the whole system.
E.g. for a shop system during Christmas, the catalog might be the most
critical and hardware consuming part. By scaling the complete system,
hardware is spent on parts which don't require more power.

Each microservice can be independently scaled. It is possible to start
additional instances of a microservice and to distribute the load of
the microservice onto the instances. This can improve the scalability
of a system significantly. 
So in the example above, just the catalog would need to be scaled up.
For this to work, the microservices
naturally have to fulfill certain requirements. For instance, they
must be stateless. Otherwise requests of a specific client cannot be transferred to another
instance since this instance then would not have the state specific
for that client.

It can be difficult to start more instances of a deployment monolith
due to the required hardware. Besides, building up an environment for a
deployment monolith can be complex. This can require additional
services or a complex infrastructure with databases and additional
software components.

In the case of a microservice, the scaling can be more fine
grained so that normally less additional services are necessary
and the basic requirements are less complex.

#### Free Technology Choice

Each microservice can be implemented with an individual technology.
This facilitates the migration to a new technology since each
microservice can be migrated individually. In addition, it is simpler and less risky
to gain experience with new technologies since they can initially be used for only
a single microservice before they are employed in several microservices.

#### Security

Microservices can be isolated from each other. For instance, it is possible
to introduce firewalls into the communication between microservices.
Besides, the communication between microservices can be encrypted to guarantee
that the communication really originates from another microservice and is authentic.
This prevents the corruption of additional microservices if a hacker
takes over one  microservice.

#### In general: Isolation

In the end, many advantages of microservices can be traced back to a stronger
isolation.


{id="fig-microservices-isolation"}
![Fig. 1-2: Isolation as the Source of the Advantages of Microservices](images/microservices-isolation.png)

Microservices can be deployed in isolation which facilitates
continuous delivery.  They are isolated in respect to failures which
improves robustness. The same is true for scalability. Each
microservice can be scaled independently of the other
microservices. The employed technologies can be chosen for each
microservice in isolation which allows for free technology choice.
The microservices are isolated in such a way that they can only
communicate via the network. Therefore, the communication can be
safeguarded by firewalls which increases security.

Due to this strong isolation the boundaries between modules can
hardly be violated by mistake. The architecture is not 
violated that often any more.  This safeguards the architecture.  A
microservice can in
isolation be replaced with a new microservice.  This enables the
low-risk replacement of microservices and allows one to keep the
architecture of the individual microservices clean. Thus, the
isolation facilitates the long-term maintainability of the software.

Decoupling is an important feature of modules. With their isolation,
microservices push it to extremes. Modules are normally only
decoupled in regards to code changes and architecture. The decoupling
between microservices goes far beyond that.

So in the end for many purposes microservices are just smaller and can
be handled in isolation due to the decoupling. That also makes it
easier to reason about them. So the security of a microservice is
easier to verify, the performance is easier to measure and it is
easier to figure out whether they work correctly. That make the design
and also the development easier.

#### Prioritizing Advantages

Which of the discussed reasons for switching to microservices is the
most important, depends on the individual scenario.  The use of
microservices in a greenfield system is rather the exception.  More
often a deployment monolith is replaced by a microservice
system (see also [chapter 4](#chapter-migration)). In that case, different
advantages are relevant.

* The easier *scaling of development* can be an important reason for
  the introduction of microservices in such a scenario. Often, large
  organisations have difficulties to work on a deployment
  monolith sufficiently fast with a large number of developers.

* The *easy migration* away from the legacy deployment monolith
  facilitates the introduction of microservices in such a scenario.

* *Continuous delivery* is often an additional goal. The aim is to
  increase the speed and reliability with which changes can be brought into
  production.
  
The scaling of development is not the only scenario for a migration.
When a single Scrum team wants to implement a system with
microservices, scaling of development cannot be a sensible reason
since the organization of development is not sufficiently large for
this.  However, also other reasons are possible. Continuous delivery,
technical reasons like robustness, independent scaling, free
technology choice, or sustainable development all can play a role in
such a scenario.

At the end the it is important to focus on the business value
increase. Depending on the scenario an advantage in one of the areas
mentioned above might make the company more profitable or
competitive e.g. faster time to market or better reliability of the
system.

#### Microservices are a Trade-Off

Dependent on the aims, a team can compromise when implementing
microservices. When robustness is a goal of introducing
microservices, the microservices have to be implemented as separate
Docker containers. Each Docker container can crash without
affecting the other ones. If robustness does not matter, other
alternatives can be considered. For instance, multiple microservices
can run together as Java web applications in one Java application
server.  In this case, they all run in one process and therefore are
not isolated in respect to robustness. A memory leak in any of the
microservices will cause them all to fail. However, such a solution is
easier to operate and therefore might be the better trade-off in the
end.

#### Two Levels of Microservices: Domain and Technical

The technical and organisational advantages point to two levels at
which a system can be divided into microservices.

* A *coarse-grained division by domain* enables the teams to develop
  independently and allows them to roll out a new feature with the
  deployment of a single microservice. For example, in an e-commerce
  system the customer registration and the order process can be
  examples of such coarse-grained microservices.

* For *technical reasons* some microservices can be *further
  divided*. When for instance the last step of the order process is
  under especially high load, this last step can be implemented in a
  separate microservice. This microservice then can be scaled
  independently of the other microservices.  The microservice belongs to
  the domain of the order process, but for technical reasons is
  implemented as a separate microservice.

{id="fig-microservices-2ebenen"}
![Fig. 1-3: Two Levels of Microservices ](images/microservices-2ebenen.png)

[Figure 1-3](#fig-microservices-2ebenen) shows an example for the two
levels. Based on the domains an e-commerce application is divided
into the microservices search, check out, payment, and delivery. Search
is further subdivided. The full text search is separated from the
category-based search. Independent scaling can be one reason for this.
This architecture allows the system to scale the full text search
independently
of the category-based search which is advantageous when both have to
deal with different levels of load. Another reason could be the use of
different technologies. The full text search can be implemented with a
full text search engine which is unsuitable for a category-based
search.

#### Typical Numbers of Microservices in a System

It is difficult to state a typical number of microservices per system.
Based on the divisions discussed in this chapter 10--20 coarse-grained
domains are usually defined, and each of these might be sub divided into
one to three microservices. However, there are also systems with far
more microservices.

## 1.3 Challenges {#section-microservices-herausforderungen}

Microservices do not only have benefits, they also pose challenges.

* The *operation* of a microservice system requires more
  effort than running a deployment monolith. This is due to the fact
  that in a microservice system many more deployable units exist which
  all have to be deployed and monitored.  This is only feasible when
  the operation is largely automated and the correct functioning of
  the microservices is
  guaranteed via appropriate monitoring.  [Part III](#chapter-betrieb)
  will demonstrate different solutions to deal with this challenge.

* Microservices have to be *independently deployable*. Dividing
  them, for instance into Docker containers, is a prerequisite for
  this, but it is not enough on its own. Also the testing has to
  be independent.  When all microservices have to be tested together,
  one microservice can block the test stage and thus prevent the
  deployment of the other microservices.
  This means that testing might become harder. Due to the split into
  microservices, there are more interfaces to test and testing has to
  be independent for both sides of the interface.
  Changes to interfaces have to be
  implemented in such a way that an independent deployment of
  individual microservices is still possible.  For example, the
  microservice which implements the interface has to offer the new and
  the old interface.  Then this microservice can be deployed without that the
  calling microservice has to be deployed at the same time.

* *Changes which affect multiple microservices* are more difficult to
  implement than changes which concern several modules of a deployment
  monolith.  In a microservice system such changes require
  several deployments. These deployments have to be coordinated. In
  the case of a deployment monolith only one deployment would be
  necessary.
  
* In a microservice system the *overview* of the microservices
  can get lost. However, experience teaches that in practice a sound
  domain-based division can restrict changes to one or a few
  microservices.  Therefore, the overview of the system is less
  important in the end since the interaction between the microservices
  hardly influences development due to the high degree of
  independence.

* Microservices communicate through the *network*. Compared to local
  communication, the latency is much higher and it is also more likely
  that the communication will fail. So a microservices system cannot
  rely on the availability of other microservices. This makes the
  systems more complex.

#### Weighing Benefits and Disadvantages

The most important rule is that microservices should only be used if
they represent the most simple solution in a certain scenario.  The
above mentioned benefits should outweigh disadvantages resulting for
example from the higher level of complexity for deployment and
operation. After all, 
it does hardly make sense to intentionally decide for a too complex
solution.

## 1.4 Variations {#section-microservice-variationen}

Dependent on the concrete scenario microservice variants such as
[self-contained systems](#chapter-scs) can be used.

#### Technological Variations

[Part II](#chapter-techstacks) and [part III](#chapter-betrieb) show
different technological variations such as synchronous communication,
asynchronous communication, and UI integration. By combining one or
multiple recipes out of these parts, a custom microservice
architecture can be designed.

#### Experiments

The following approach helps to find the right recipes.

* Identify the problems in your current system (e.g. resilience,
  development agility, too slow deployment ...)

* For one of the projects that you know, prioritize the benefits of
  using microservices.

* Weigh which of the challenges in this project could pose a risk.

* Afterwards, the possible technical and architectural solutions in
 the following chapters can be
 compared with respect to whether they provide a sensible solution in
 the context of these requirements.

For the concrete division into microservices and for technical
decisions, additional concepts are necessary. Therefore, the question
how to best divide a system into microservices is the focus of
[section 2.7](#section-mikro-makro-variationen).

## 1.5 Conclusion {#section-microservice-fazit}

Microservices represent an extreme type of modularization. Their
separate deployment is the foundation for a very strong degree of
decoupling.

This results into numerous advantages. A crucial benefit is isolation
at different levels. This does not only facilitate deployment, but
also limits potential failures to individual
microservices. Microservices can be individually scaled, technology
decisions affect only individual microservices, and security problems
can also be restricted to individual microservices. The isolation
allows one to more easily develop a microservices system with a
large team since less coordination between teams is required. In
addition, the smaller deployment artefacts make 
continuous delivery easier. Moreover, replacing a legacy system is
much easier with microservices since new microservices can supplement
the system without the necessity of large code changes in the legacy system.

The challenges are mostly associated with operation. Appropriate
technological decisions should strengthen the intended benefits and at
the same time they should minimize disadvantages like the complexity
in operation.

Of course, integration and communication between microservices is more
complex than the calls between modules within a deployment monolith.
The added technological complexity represents an additional important
challenge for microservice architectures.

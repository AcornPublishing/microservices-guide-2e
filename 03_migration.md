# 3 Migration {#chapter-migration}

Migration from a deployment monolith to a microservices architecture
is the common case for introducing microservices. Most projects start
with a deployment monolith that they want to split into microservices
because the deployment monolith has too many disadvantages.

Of course, it is also possible to implement a new system directly with
microservices from scratch.

This chapter provides an overview of the challenges involved in
migrating to a microservices system.

* The chapter discusses possible *reasons* for a migration. In this
way, readers can assess whether a migration makes sense in their
context. The approach to migration depends on the objectives of
the migration. Therefore, knowing  possible reasons is helpful
for choosing a migration strategy.

* The chapter shows a *typical strategy* for migration, but also
alternatives. In this way, the reader can choose a migration approach
suitable for his or her scenario.

## 3.1 Reasons for Migrating {#section-migration-gruende}

When migrating to microservices, it is important to know the
objectives for taking this step. There are several possible reasons
for microservices. Depending on the reasons that led to the decision
to migrate to microservices, the procedure for implementing them may
vary.

#### Microservices Offer a Fresh Start

Especially when replacing legacy systems, microservices have several
advantages. The code of the legacy system no longer needs to be used
in the new microservices because the microservices are implemented
separately from the legacy system. So the microservices offer an
unencumbered restart. The code of a legacy system is often no longer
maintainable, and the technologies are frequently outdated. Therefore,
reusing the old code would hinder the development of a clean new
system. Microservices thus solve the most important challenges when
dealing with legacy systems because otherwise a restart is difficult,
as newer systems have to be integrated with old code.

Migration to microservices has the potential to finally solve the
problem with the legacy system. After migration to microservices,
further migrations can be limited to one or a few microservices. A
migration of the entire system will probably not be necessary again. A
typical reason for a system migration is an outdated technology basis.
In a microservices system such a migration can take place step-by-step
i.e. microservice by microservice. Another migration reason is an
unmaintainable system. Also in this case, each microservice can be
replaced individually.

#### The Reasons are Already Known

The reasons for a migration are in the end identical to the reasons
for using microservices. These have already been discussed in detail
in [section 1.2](#section-microservices-gruende) and may
include increased security, robustness, and independent scaling of
individual microservices.

#### A Typical Reason: Speed of Development

A typical reason for introducing microservices is the lack of speed in
developing with a deployment monolith. When many developers are
working on a deployment monolith, they need to closely coordinate
their work. This costs time and therefore slows down development. But
even with a small team, a deployment monolith can be problematic
because the deployment is quite huge. The size makes continuous
delivery difficult to implement, and each
release requires a lot of testing.

## 3.2 A Typical Migration Strategy {#section-migration-strategie}

Often there is a concept for the final target architecture that the
migration should achieve, but no concrete plan for the first steps to
be taken or for the first microservices to be implemented. In
particular, the small steps into which development can be broken down
are a major advantage of microservices. A simple microservice is
written quickly. Because of its small size, it is also easy to deploy.
And if the microservice should not prove itself, not much effort has
gone into the microservice and it can easily be removed again. In the
further course of the project, the new architecture can be implemented
step-by-step and microservice by microservice. In this way, large
risks can be avoided.

Since the migration process depends on the goals and on the structure
of the
legacy system, there is no universal approach. Therefore, the strategy
presented here must not simply be used as is, but must be adapted to
the respective situation.

#### A typical Scenario

A typical scenario for a migration to microservices is:

* The aim of the migration is to increase the development speed.
Microservices need fewer tests for a release and provide easier
continuous delivery because they are smaller. Also 
development of individual microservices is quite independent. So less
time is spent with coordination. All of this can make the development
faster.

* The migration to microservices must provide an advantage in
development as quickly as
possible. It simply makes no sense to
invest in an architectural improvement that does not bring any
only leads to an improvement much later.

The migration strategy proposed here is based on extracting individual
microservices (see [figure 3.1](#fig-migration-vorgehen)) in order to
achieve an improvement of the situation as quickly as possible.

{id="fig-migration-vorgehen"}
![Fig. 4-1: Approach for a Migration: Asynchronous Communication and UI Integration Between Legacy System and Microservice. The Microservice has its own Data Storage.](images/migration-vorgehen.png)

#### Give a Preference to Asynchronous Communication

Integration with the legacy system should take place via asynchronous
communication. This decouples the domain logic of the microservice
from the legacy system. The legacy system sends events. To do this,
the legacy system must be adapted to create events. Since the legacy
system is usually poorly maintained, this can be a challenge.

Microservices then can decide how to react to these events.
Availability also benefits. A failure of the legacy system does not
lead to the failure of the microservice, and a failure of the
microservice does not lead to the failure of the legacy system.

#### Give a Preference to UI Integration

Further integration is conceivable at the UI level. If the legacy
system and the microservice are integrated with each other via links,
then only URLs are known. What hides behind the URLs can be decided by
the linked system and can change without much impact on other systems.

Via links additional resources can be available. This is the basis of
[HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) (Hypermedia as the
Engine of Application State). The client can interact with the system
through the links. It does not need to know about possible interaction
possibilities, but can follow the links. For example, a link to cancel
an order would be sent along with the order. New interaction
possibilities can be easily supplemented by new links.

The UI integration also offers an easy way to operate the microservice
and the legacy system in parallel. Individual requests can be
redirected to the microservices, while the remaining requests are
still processed by the legacy system. Often, there is a web server
anyway which processes every request and carries out TLS/SSL
termination, for example. A parallel operation of microservices and
legacy system is then quite simple. The web server only has to forward
each request either to a microservice or to the legacy system.

UI integration is particularly easy if the legacy system is a web
application. But it is also possible, for example, to integrate web
views in a mobile application to thereby integrate parts of the UI as
web pages. In such cases, UI integration should really be considered
as an option because of its many advantages.

#### Avoid Synchronous Communication

Synchronous communication should be used sparingly. It leads to a
close dependence in regards to availability. If the called system
fails, the calling system must be able to deal with this. The degree of  coupling
in the domain logic is also quite high. A synchronous call usually
describes exactly what needs to be done. Synchronous communication may
be necessary if you want the last changes to be visible in the other
systems as soon as possible. In a synchronous call, the state at the
time of the call is always used, while asynchronous communication and
replication can lead to a delay until the current state is known
everywhere.

#### Reuse Old Interfaces?

If there is already an interface, it may be useful to use this
interface to save the effort of introducing a new interface. However,
the interface may not be well adapted to the needs of the
microservice. In addition, it cannot be changed easily because there
are already other systems that also use this interface and are
influenced by changes.

The technology the interfaces uses is not too important for the
decision whether it should be used by a microservice. Microservices
can use almost any type of interface. For a migration, it might be a
lot easier to use an existing interface even with an awkward
technology than to create a new one.

More important are the dependencies the microservices establish: As
discussed in [section 2.1](#section-mikro-makro-bounded-context) the
selected pattern for the integration influences the coordination
effort and the degree of independence. Just reusing an existing
interface might compromise a goal like independent development.


#### Integrating Authentification

For a system that consists of a legacy system and microservices, the
user should only have to log in once. Legacy system and microservices
do not necessarily have the same authentication technologies,
but the systems must be integrated in such a way that a single sign-on
is possible and the user does not have to log on to the legacy system
and microservices separately. Authentication may also need to provide
roles and permissions for authorization in the microservices. Also
here adjustments may be necessary.

#### Replicating Data

Even in a migration scenario, each microservice should have its own
database or at least its own database schema. The goal of the
migration is to achieve independent development and simple continuous
delivery of the microservices. This is not possible if the
microservices and the legacy system use the same database. A change to
the database schema then might have effects that are hard to predict. So the
microservice is then hardly changeable and difficult to put into
production.

Together with asynchronous communication, a separate database means
data replication. This is the only way for the microservices to
implement their own data model. Changes to the data can be
communicated via events.

Replication for a specific part of the data should only take place in
one direction. 
Usually replication can be done using business events i.e. events that
have a meaning for a business expert.
One system (a microservice or the legacy system) should
trigger the events and the other system reacts to the events.
So for example one system could generate a system like "customer
registered" and the other system could store the customer data that is
relevant to them.
However, there should only be one source of each type of events.
Otherwise, it can be very complicated to bring together the changes of
the different systems into a consistent state.

#### Black Box Migration

Often the code of the deployment monolith is hard to understand and
modify. This might even be a reason for a migration. Therefore it does
not make a lot of sense to reverse engineer the existing code or even
restructure it. As little knowledge about the existing system should
be required for the migration as possible.

#### Choosing the First Microservice for the Migration 

A legacy system comprises numerous domain functionalities. For
deciding about the migration strategy, it can be useful to analyze the domain
logic of the legacy system. The result should be a complete and ideal
split of the legacy system into bounded contexts (see
[section 2.1](#section-mikro-makro-bounded-context)). It is not
implemented in the legacy system but can be the goal for the migration to
microservices.
This analysis can be done without understanding the code. It is about
what the system does i.e. it is enough to treat it as a black box.

Extracting one of these bounded contexts as a microservice has the
advantage that a bounded context is largely independent of other
bounded contexts from a domain perspective since it has its own
domain model.

However, the question is which bounded context to extract first from
the legacy system. There are different approaches.

* To keep the *risk* as low as possible, an unimportant bounded
context with little load can be the right choice. This makes it
possible to gain experience with the challenges of microservices, e.g.
concerning operation, without taking too great a risk.

* Microservices are meant to simplify development. In order to exploit
the advantages of this approach as quickly as possible, you can
migrate a bounded context to a microservice that *will have to be
changed a lot* in the foreseeable future. The changes should become
easier to introduce after migrating to a microservice, so that the
cost of migration quickly pays for itself.

#### Extreme Migration Strategy: All Changes in Microservices

An extreme migration strategy is to no longer allow any changes to the
legacy system, but only allow changes to microservices. When a change
would have to be made to the legacy system, a new microservice must be
created first. The change is then implemented in this microservice
instead. This
automatically results in a migration to microservices as more and more
logic is implemented in microservices over time. It is very easy
to follow this rule.

One problem with this approach is that the microservices are created
in random places, namely where the system is currently being changed.
This can result in microservices that only implement parts of a
bounded context, while other parts of the bounded context are still
implemented in the legacy system. Then microservices and legacy
system have numerous dependencies, making independent
development difficult.

#### Further Procedure: Step-by-step Migration

The legacy system can be gradually replaced by microservices. In the
course of the migration, the focus should be on converting parts of
the system into microservices that are currently undergoing major
changes so that the migration to microservices is worthwhile. This is
called the
[*strangler pattern*](https://www.martinfowler.com/bliki/StranglerApplication.html).
The microservices increasingly strangle the legacy system until
nothing is left of the legacy system anymore (see
[figure 4-2](#fig-migration-vorgehen)).

{id="fig-migration-strangler"}
![Fig. 4-2: Further Migration: An Increasing Number of Microservices Take over Functionalities from the Legacy System](images/migration-strangler.png)

The full migration to microservices can take very long. However, this is
no problem: Only those parts of the system are migrated for which
migration brings an advantage.
For example, a part of the system must be changed. It is migrated to a
microservice. That makes the changes much easier. Parts that are
never or very seldom changed will be migrated very late or even never.
So the time the full
migration takes is a result of the flexibility to migrate only what is
actually needed.
The migration stops if there is nothing worth migrating any more.
In the end, it
does not make sense to invest into the optimization of system parts
which are rarely or never changed.


It might even happen that the legacy system could in principle be
completely migrated, but is still retained. When hardly any changes to
the legacy system are necessary anymore, because all parts that
require changes have already been migrated to microservices, retaining
the legacy system can be the best solution.

## 3.3  Alternative Strategies {#section-migration-alternativen}

There are many more strategies. There is a
[presentation](https://speakerdeck.com/ewolff/monolith-to-microservices-a-comparison-of-strategies)
that gives a good overview of the different approaches to the
migration to microservices. However, this section also shows some of
the more common approaches.

#### Goal Reliability

As mentioned already, there can be very divergent approaches for the
migration to microservices. The strategy depends mainly on the
objectives to be achieved. When the main objective for switching to
microservices is an increase in robustness, at first reliability can be
improved at the interfaces to external systems or databases with
libraries like Hystrix (see [section 13.5](#section-netflix-hystrix)).

Then the system can be split stey-by-step into individual
microservices which run independently of each other so that a failure
of one microservice does not affect the other microservices anymore.
There is an interesting talk about this approach to which the
[slides](https://www.innoq.com/de/talks/2015/11/javaday-kyiv-modernization-legacy-systems-microservices-hystrix/) are available.

#### Migration based on Layers

Another alternative is a migration based on layers. For example, the
UI can be migrated first. This can make sense when changes to the UI
are imminent and therefore the migration can be combined with
necessary changes. Of course, this migration strategy is
in contrast to the idea of combining UI, logic and data in one
self-contained system (see
[chapter 3](#chapter-scs)). However, it can still be a first step
towards this goal. In that case the remaining layers would have to be
migrated into the same microservice afterwards. Alternatively, one
stays with a division of microservices in layers although it is not
optimal. An ideal architecture, into which, however, it is impossible to migrate, is in the end a lot less helpful than a less optimal architecture
which can actually be implemented.

#### Copy/Change

Another possibility is copy/change. Here, the code of the legacy
system is copied. In one copy one part of the system is developed
further, while the other part is removed. In a second copy it is the
other way around. In this manner, the legacy system is converted into
two microservices. This approach has the advantage that the old code
is still used and therefore the functionalities of the microservices
very likely accurately correspond to the functionalities of the legacy
system.

However, at the same time it is a great disadvantage to continue using
the old code. In most cases the code of a legacy system is hard to
maintain. So it is problematic to keep using this code. In addition,
the database schema remains unaltered. The shared use of the database
schema by the legacy system and the microservices results in a tight
coupling between the two which really should be avoided to be able to
profit from the advantages microservices offer.
That is why a black box migration might be better.

Moreover, the structure of the legacy system and the technology stack
largely remain the same.

Thus, the project has a lot of technical debt from the very beginning
and does not represent a new start. This approach does not take
advantage of the benefits of microservices such as freedom of
technology. Therefore, it should only be used in exceptional cases.

## 3.4 Build, Operation, and Organisation {#section-migration-build-betrieb-organisation}

Code migration alone is not enough to turn a legacy system into a
microservices system.

* The microservices must also be *built*. A suitable tool must be
selected for this purpose. In addition, the continuous integration
server has to cope with the multitude of microservices.

* Similarly, technologies and approaches must be introduced to enable
the *deployment* and *operation* of microservices.

* Finally, a suitable *test strategy* must be established. This also
requires the automated setup of test environments. It must also
be ensured that the tests are independent. For example, stubs that
simulate microservices or the legacy system are useful for this
purpose. Also
[consumer-driven contract tests](https://martinfowler.com/articles/consumerDrivenContracts.html)
might be useful.
They safeguard the requirements for the interfaces of microservices
or legacy systems with the help of tests. However, legacy systems are
often very complicated, so these techniques are difficult to
implement.

Therefore, dealing with the first microservice can require extra
effort because the infrastructure for build and deployment needs to be
set up. It is conceivable to build the infrastructure later,
but it is recommended to start building the infrastructure as early as
possible in order to reduce the risk of migration. One or a few
microservices can still be operated with an inadequate solution for
build and deployment. However, once the number of microservices
increases, without an appropriate infrastructure the necessary effort
will become so high that it can lead to the failure of the project.

#### Co-existence between Microservices and Legacy System

During a migration, the legacy system must be deployed and further
developed in addition to the microservices. It is unrealistic to
deploy the legacy system as often as the microservices because the
effort for deploying the legacy system is usually far too high.
Therefore, changes affecting both the legacy system and the
microservices are difficult to implement. They require at least one
deployment of the microservices and one deployment of the legacy
system. Solutions can be found at the architectural level. If a new
feature is only implemented in a microservice, then only a deployment
of this microservice is necessary. This speaks for a division of the
microservices according to bounded context. Another option would be to
integrate the monolith with patterns such as *open host service*
or *published language* (see
[section 2.1](#section-mikro-makro-bounded-context)) to provide a
generic interface that rarely needs to be changed.

#### Integration Test of Microservices and Legacy System

There must also be integration tests that test microservices with the
version of the legacy system that is currently in production and with
the one that is currently being developed. This ensures that the
microservices continue to work when the legacy system is deployed. The
legacy system can support two different versions of the interfaces so
that microservices can switch to a new version of an interface when it
is provided. However, no microservice is forced to use a new interface
that has not yet been tested together with the microservice. In this
way, the version of the microservice that uses a new interface of the
legacy system can be deployed at any time.

#### Coordinated Deployment between Legacy System and Microservices

A coordinated deployment of microservices together with the legacy
system would be an alternative. When a change is made, the new version
of the microservices and the legacy system are rolled out at the same
time. This increases the risk because more changes occur at the same
time and it is harder to roll back the deployment. It is also
difficult to implement this approach without downtime. With a
complex microservices environment, this option is anyhow hardly
possible anymore because too many microservices would have to be
deployed at once. Therefore, the deployment of microservices and
legacy system should be decoupled from the outset.

#### Organizational Aspects

An essential advantage of microservices is the possibility to
scale the development process (see
[section 1.2](#section-microservices-gruende)).

If the goal of a migration to microservices is to have independent
teams, the migration of the architecture must be accompanied by a
reorganization. [Section 2.5](#section-mikro-makro-organisation)
already discussed the essential aspects of the target organization.

The organizational change must be coordinated with the technical
migration. For example, a microservice can be detached from the legacy
system and then developed autonomously by a team. At the same time,
the other organizational structures can be set up, for example the
ones required for defining the macro architecture.

#### Recommendation: Do not Implement all Aspects at Once

Microservices require changes in architecture and organization as well
as the introduction of new technologies. Implementing all these
changes at once is risky and complicated. Unfortunately, many of the
changes are connected. Without new technologies, the architecture is
difficult to implement. Without the architecture, organizational
changes are difficult to introduce. However, making all these changes
at once should still be avoided. For each change the question should
therefore be asked as to when the change is actually necessary, in
order to implement it at a later time point, if possible.

## 3.5 Variations {#section-migration-variationen}

The ideas for migration can easily be combined with many other
approaches.

* The ideas concerning typical migration strategies from
[section 3.2](#section-migration-strategie) fit very well to the
concept of *self-contained systems* ([chapter 3](#chapter-scs)). The
migration can therefore simply separate a part of the legacy system
into an SCS.

* Rules for authentication or communication between microservices as
well as between microservices and legacy system can be the starting
point of a *macro architecture* (see
[chapter 2](#chapter-mikro-makro)). A domain macro architecture is
very useful, which can also include the legacy system in addition to
microservices.

* *Frontend integration* (see [chapter 7](#chapter-frontend))
  can make sense for the integration between legacy system and microservices.

* *Asynchronous microservices* ([chapter 10](#chapter-asynchron)) fit
very well to migration since they allow for a loose coupling.
Especially for a migration it can be sensible to continue to use an
existing messaging technology for asynchronous communication to
minimize the effort.

* Synchronous microservices ([chapter 13](#chapter-synchron)) should
be used cautiously because this creates a tight coupling and
resilience becomes a difficult topic.
  
* Kubernetes ([chapter 17](#chapter-kubernetes)), PaaS
([chapter 18](#chapter-paas)) or Docker ([chapter 5](#chapter-docker))
are certainly also interesting in a migration scenario. However, they
represent a *new environment* that needs to be operated. It may
therefore make sense to use a classical deployment and operation
environment at least at the beginning to reduce the initial migration
effort. In the long term, however, such environments have many
advantages. In addition, the old system can of course also be operated
in such an environment.

#### Experiments

The migration strategy must match the respective scenario. The
following questions are important in order to design your own
strategy.

* What are the goals of the migration to microservices?
  * Which are especially important?
  * What impact does this have on the migration strategy?

In principle, migration should take place gradually. The selection of
the parts to be migrated to microservices can be made according to
technical or domain criteria. However, domain criteria are better
suited, at least in the long term.

The following approach is suitable for a migration based on domain
criteria:

* Split the system into bounded contexts.

* Which of the bounded contexts will you migrate first? Why? Reasons
can be the simple migration of the bounded context or many planned
changes in the bounded context. Consider different scenarios.

## 3.6 Conclusion {#section-migration-fazit}

Migration to microservices is the typical approach for the
introduction of microservices. Implementing a completely new system
with microservices is rather the exception, but of course it is also
possible. One of the most important advantages of microservices is
that they do not only work well in greenfield projects.

Choosing the right migration strategy is a complex task. It depends on
the legacy system and migration goals. This chapter shows a starting
point from which each project must develop its own strategy.

Because of the benefits of migration, microservices should be
considered in any project that is meant to modernize a legacy
system. Microservices enable step-by-step modernization, in which completely different technologies can be used. This is very helpful.

The migration strategy can have a significant impact on the
architecture and technology selection. A split into microservices that
is similar to the split of the modules in the legacy
system can greatly simplify migration. Such a compromise has
far-reaching consequences and can lead to a worse target architecture,
but it can still make sense. Finally, it must be possible to actually implement the architecture, and for this a simple approach for the migration into the architecture is essential.

# 6 Self-contained Systems {#chapter-scs}

A self-contained system (SCS) is a type
of microservice that specifies elements of a macro architecture. SCSs
do not represent a complete macro architecture. The area of
operation, for instance, is completely missing.

The idea behind SCS is to provide microservices that truly provide
everything needed for the implementation of a part of the domain logic
and are therefore self-contained. This means an SCS includes logic,
data, and a UI. That means if a change impacts all technical layers,
it can still be contained in one SCS. That makes it easier to do the
change and put it into production. So an SCS for payment would store
all information relevant to payment i.e. it would implement a bounded
context. But it would also implement the UI - i.e. web pages to show
the payment history or do a payment. Data about customers or ordered
items would need to be replicated from other SCSs.

This chapter conveys the following knowledge:

* First, it describes the reasons for using SCSs.

* SCSs determine different macro architecture decisions. The chapter
  explains the reasons for these macro architecture decisions and
  discusses which advantages result from the different decisions.

* SCSs are only a variation of microservices. The chapter addresses
  the differences between the terms "SCS" and "microservice".
  
* The chapter then discusses the benefits of the SCS approach.

* Finally, challenges connected to the development of an SCS system
  are described and potential solutions are discussed.

SCS include a UI and focus on UI integration. Therefore they are a
good motivation for the UI integration techniques the next chapters
explain.

## 6.1 Reasons for the Term Self-contained Systems {#section-scs-warum}

There is no uniform definition for microservices. Self-contained
systems, however, are very precisely defined. You can read the
definition of SCSs on the <http://scs-architecture.org> website. The
content of the site is provided under a creative commons license so
that the
material on the site may be reused by anyone if the source is named
and the materials are distributed under the same license terms.

The content of the website is available as source code at
<https://github.com/innoq/SCS> so that everyone can make changes. The
website contains links to many articles about experiences with SCSs
from different projects and companies.

Self-contained systems are best practices that have proven their
usefulness in various projects. While microservices do not provide
many rules about how systems should be built, SCSs have precise rules
based on proven patterns. Thus, SCSs give a point of reference as to how a
microservices architecture can look like.

Although SCSs are a collection of best practices, the SCS approach is
not the best architecture for every situation. Therefore, it is
important to understand the reasons for the SCS rules. In this way,
teams can choose variations of this approach or even completely
different approaches that might be better adapted to the respective
project.

## 6.2 Definition {#section-scs-definition}

Self-contained systems make different macro architecture decisions.

* Each self-contained system is an *autonomous web application*. Thus,
  SCSs contain a web UI.

* There is *no common UI*. An SCS can have HTML links to other SCSs or can
integrate itself by different means into the UI of other SCSs. But every part
of the UI belongs to an SCS. [Chapter 7](#chapter-frontend) describes
different options for the integration of frontends. This means each
SCS generates a part of the UI. There is no separate microservice that
generates all of the UI and then calls logic in the other microservices.


* The SCSs can have an *optional API*. This API can, for instance, be
  useful if mobile clients or other systems have to use the logic in
  the SCS.

* The *complete logic* and *all data* for the domain are contained in
  the SCS.  This is what SCSs were named for. An SCS is self-contained
  because it contains UI, logic and data.

These rules ensure that an SCS completely implements a domain. This
means that a new feature only causes changes to one SCS even if the
logic,
data, and also the UI need to be changed. These changes can be rolled out with
a single deployment.

If several SCSs shared a UI, many changes would affect not only the
SCSs but also the UI. Then two well-coordinated deployments and a
closely coordinated development would be necessary.

#### Rules for Communication

The communication between SCSs has to follow a number of rules.

* Integration at the *UI level* is ideal. The coupling is then very
loose. The other SCS can display its UI as required. Even if the UI is
changed, other SCSs are not affected. For example, if HTML links are
used, the integrated SCS does not even have to be available. The link
is displayed even if the system is unavailable. Only when the user
clicks on the link, there will be
an error if the linked system still fails. This helps with resilience,
as the failure of an integrated SCS does not affect other SCSs.

* The next option is *asynchronous* communication. The advantage of
this is that if the integrated SCS fails, the requests only take
longer until the failed SCS is available again. However, the calling
SCS will not fail because it has to be able to deal with longer
latency times anyway.

* Finally, integration with *synchronous* communication is also
possible. Then precautions must be taken to deal with the potential failure and slow
responses of the integrated SCSs. In addition, the response times add up
if you have to wait for the responses of all synchronous services.

These rules focus on a very loose coupling between the SCSs to avoid
error cascades in which, when one system fails, the dependent systems
as a consequence also fail.

From these rules follows that an SCS replicates data. The SCS has its
own database. Since it is primarily intended to communicate
asynchronously with other SCSs, it is a challenge if the SCS must use
data from another SCS to process a request. If the data is requested
during the processing of the request, the communication is
synchronous. For asynchronous communication, the data must be
replicated beforehand so that it is available in the SCS when
processing the request.

Consequently, SCSs are not always consistent. If a change to the
dataset has not yet been passed on to all SCSs, then the SCSs have
different datasets. This may not be acceptable in some situations. For
particularly high consistency requirements, the SCSs must use
synchronous communication. In this scenario, one SCS receives the current
state of the data from the other SCS.
	
#### Rules for the Organization

Also for the organization SCSs provide macro architecture rules. An
SCS belongs to *one team*. The team does not necessarily have to make
all changes to the code, but it must at least review, accept, or reject
changes. This enables the team to direct and control the development
of the SCS.

It is possible for a team to handle several SCSs. However, it must not
happen that an SCS is changed by more than one team on an equal
footing.

Thus, self-contained systems use the strong architectural decoupling
to achieve organizational advantages. The teams do not need to
coordinate very much and can work in parallel on their tasks.
Requirements can usually be implemented in one SCS by one team because
an SCS
implements a domain. Coordination in this respect is therefore hardly
necessary. Since the technical decisions mostly concern only one SCS,
there are hardly any arrangements necessary either.

For SCSs, as well as for microservices and other types of modules, the
division into modules is aimed at enabling independent development.
Having several teams change the same module is obviously
not a very good idea. The modules allow independent development. But
when several teams are working on one module, close coordination is
still necessary. The advantage of modularization is not exploited
then. This is the reason why equal joint development of an SCS by
several teams is not allowed.

#### Rule: Minimal Common Basis

Since self-contained systems aim at enabling a high degree of
independence, the common basis should be minimal.

* Business logic must not be implemented in code used by more than one
  SCS.
  *Shared business logic* leads to a close coupling of the SCSs. That
  should be avoided. Otherwise, a change to one SCS could require
  changes to the shared code. Such changes must be coordinated with
  other users of the code. This creates a tight coupling that SCSs
  should avoid. In addition, common business code indicates a bad
  domain macro architecture. Business logic should be implemented in a
  single
  SCS. The business logic in an SCS can of course be called and used
  by another SCS via the optional interface of the SCS.

* *Common infrastructure* should be avoided. SCSs should not share a
  database, for instance. Otherwise, the failure of the database would
  lead to a failure of all SCSs. However, a separate database for each
  SCS requires a considerable effort.  For this reason, compromises
  are conceivable if the robustness of the system is not quite so
  important. The SCSs could have a separate schema in a common
  database. A shared schema would violate the rule that each SCS
  should have its own data.

{id="fig-scs-konzept"}
![Fig. 6-1: SCS: Concept](images/scs-konzept.png)

[Figure 6-1](#fig-scs-konzept) provides an overview of the most
important features of the concept of self-contained systems.

* Each SCS contains its own *web UI*.

* In addition, the SCS contains the *data* and the *logic*.

* Integration is *prioritized*. UI integration has the highest
priority followed by asynchronous and finally synchronous integration.

* Each SCS ideally has its *own database* to avoid a common
  infrastructure.

## 6.3 An Example {#section-scs-beispiel}

{id="fig-scs-beispiel"}
![Fig. 6-2: Example of an SCS Architecture](images/scs-beispiel.png)

[Figure 6-2](#fig-scs-beispiel) demonstrates how the example from
[section 2.1](#section-mikro-makro-bounded-context) can be implemented
with SCSs.

* One SCS implements the *search* for products.

* With *check out* a shopping cart which was filled during searching
is turned into an order.

* *Payment* ensures that the order is payed and provides information
about the payment.

* *Shipping* sends the goods to the customer and offers the customer
  the possibility to obtain information regarding the
  state of the delivery.

Each SCS implements a bounded context with its own database or at
least one schema in a common database. In addition to the logic, each
SCS also contains the web UI for the respective functionalities. An
SCS does not necessarily have to implement a bounded context, but this
achieves a high degree of independence of the domain logic.

#### Communication

A user sends an HTTP request to the system. The HTTP request is then
usually
processed by a single SCS without another SCS being involved. This is
possible because all the needed data is available in the SCS.  This is
good for performance and also for resilience. The failure of
one SCS does not lead to the failure of another SCS, since the SCS
does not call any other systems when processing the requests. Slow
calls of other SCSs via the network are thus also avoided which
promotes performance.

Of course, the SCSs must still communicate with each other. After all,
they are part of an overall system. One reason for communication is
the lifecycle of an order. The customer searches for products in
*search*, orders them in the *check out*, pays them in *payment* and
then tracks the *shipping*.  When moving from one step to the next,
the information about the order must be exchanged between the
systems. This can be done asynchronously. The information does not
have to be available in the other SCSs until the next HTTP request so
that temporary inconsistencies are acceptable.

In some places, close integration seems to be necessary. This means
that *search*, for instance, must display not only the search results,
but also the contents of the shopping cart. However, the shopping
cart is managed by *check out*. UI integration can be a solution to
such challenges. Then the *check out* SCS can decide how the shopping
cart is displayed, and the representation will be integrated in the
other SCSs. Even changes to the logic rendering the shopping cart will be implemented only in one SCS, although many SCSs will display the
shopping cart as part of their web pages.

## 6.4 SCSs and Microservices {#section-scs-microservices}

SCSs stand out from deployment monoliths in the same way as
microservices. A deployment monolith would implement the entire
application in a single deployable artifact. SCSs divide the system
into several independent web applications.

Since an SCS is a separate web application, it can be deployed
independently of the other SCSs. SCSs are also modules of an overall
system. Therefore, SCS are independently deployable modules and
consequently correspond to the definition of microservices.

However, an SCS may be split into several microservices. If the
payment in the *check out* SCS of an e-commerce system causes a
particularly high load, then it can be implemented as a separate
microservice, which can be scaled separately from the rest of the
*check out* SCS. So the *check out* SCS consists of a microservice for
payment and contains the rest of the functionality in another
microservice.

Another reason for separating a microservice from an SCS is the
security advantage offered by greater isolation. In addition, domain
services, which calculate the price of a product or quotation for all
SCSs, can also be a useful shared microservice. The microservice
should be assigned to a team like an SCS in order to avoid too much
coordination.

In summary, microservices and SCSs differ in the following ways.

* Typically, microservices are *smaller* than SCSs. An SCS can be so
large that an entire team is busy working on it. Microservices can
potentially have only a few hundred lines of code.

* SCSs focus on a *loose coupling*.  There is no such rule for
  microservices, although tightly coupled microservices have many
  disadvantages and therefore should be avoided.

* In any case, an SCS must have a *UI*. Many microservices offer only
  a technical interface for other microservices, but no user
  interface.

* SCSs recommend UI integration or asynchronous communication, while
  synchronous communication is allowed, but not recommended.  Large
  microservices systems such as Netflix also focus on *synchronous
  communication*.


## 6.5 Challenges {#section-scs-herausforderungen}

SCS describes an architectural approach which is more narrowly defined
than, for instance, microservices. Therefore, SCS cannot be the
approach for solving all problems.

#### Limitation to Web Applications

The first limitation of SCSs is that they are web applications.  Thus
SCSs are no solution when no web UI is required.

However, some aspects of SCSs can still be implemented in a scenario
that is not about web applications. The clear separation of domains
and the focus on asynchronous communication. If a system is to be
developed that, for example, only offers an API, an architecture can
be created that provides at least some of the advantages of SCSs.

#### Single Page App (SPA)

A single page app (SPA) is usually an application written in
JavaScript that runs in the browser. SPAs often implement complex UI
logic. Applications such as Google Maps or GMail are examples of
applications that are highly complex and must be very
interactive. SPAs are ideal for these cases.

However, SPAs also have disadvantages:

* Since logic can be implemented in an SPA, in practice *business logic
often also leaks into the UI*. This makes further development more
difficult, since logic is now implemented on the server and the client
and thus at two different points in two, often different, programming
languages.


* The *load times* of an SPA can be higher than the load times of a
  simple website. Not only HTML must be displayed, but also the
  JavaScript code must be loaded and started. Loading times are very
  important in some areas such as e-commerce, because the user
  behaviour of customers depends on loading times.

With SCSs, these problems are complemented by further challenges such as:

* An *SPA for the entire system* would mean that there is a common UI.
This is forbidden in SCSs.

* An *SPA per SCS* is one possibility for providing each SCS with its
own UI.  In this case, switching from one SCS to another means
starting and loading a new SPA, which can take some time. Moreover, it
is not so easy to split one SCS into multiple SCSs, because the SPA
also needs to be split up. However, a further division can be
important for the further development of the system in order to adapt
the architecture.

Due to the high popularity of SPAs among developers, the
contradictions between SPAs and SCSs in practice are a major reason
for difficulties in implementing SCSs. An alternative is ROCA (see
[section 7.3](#section-frontend-roca)). It establishes rules that
stand for classic web applications and are much easier to use with the
SCS idea.

#### Mobile Applications

SCSs are not suitable as just a backend for mobile applications. The
mobile application is a separate UI from the backends so that UI and
logic are separated and the concepts of SCS are violated. Of course, it
is still possible to implement an SCS with a web UI that also provides
a backend for a mobile application.

There are different alternatives:

* A *web application* can be developed instead of a mobile
application. It can be responsive so that the layout adapts to
desktop, tablet or smartphone. Compared to a native app, the
advantage is that there is no need to download and install an app from
the app store. The number of apps a typical mobile user uses is quite
low. Therefore, the installation of an app can be a hurdle.  Thanks to
HTML5, JavaScript applications can now use many of the mobile phone
features.  Websites like <https://caniuse.com/> show which features
are provided by which browser. When the decision is made in favor of a
web interface, real SCSs can be implemented.

* A *web application can be implemented with a framework* such as
[Cordova](https://cordova.apache.org/) that takes advantage of the
smartphone's specific features.  There are other solutions based on
Cordova. So you can still create a real SCS, but the app can be
downloaded from the app store, use all features of the smartphone and
behave like a native app. Of course, it is possible to implement parts
of an app with such a framework and implement the rest as a truly
native app.

* Finally, a *native app* can be created. Then, for example, the
backends will offer a REST interface. If the backends do not also
implement a web interface, they are not SCSs. Still, even in this case
it is of course possible to divide the logic into largely independent
services and use asynchronous communication between the services in
order to achieve many advantages of the SCS architecture.

#### Look & Feel

Dividing a system into several web applications quickly raises the
question of a uniform look &
feel. [Section 2.2](#section-mikro-makro-technisch) has already shown
that a uniform look & feel can only be achieved with a macro
architecture decision. This also applies to SCSs, of course.

## 6.6 Benefits

SCS are a logical extension of the idea of bounded context presented
in [section 2.1](#section-mikro-makro-bounded-context). A bounded
context contains the logic for on a specific part of the domain. SCS
make sure that also the UI and the persistence are part of one single
microservices. So the split by domain that domain-driven design
advocates is further improved by SCS.

Ideally one feature should lead to one change. In an SCS system the
chance that this actually happens is quite high: The change will
likely be about in one domain. Then it can be contained in one SCS
even if the UI and the persistence need to be change. And the change
can be put into production with one deployment. Therefore SCS provide
a better changeability.

Also testing is easier because all required code for the domain are in
one domain. So it is easier to do meaningful tests of the logic
together with the UI.

Due to the focus on the UI, SCS make it easier to implement
integration in the UI. So SCS open up additional integration
options. This is in particular useful for heterogeneous UI technology
stacks. With the rate of innovation in UI technologies, it is
unrealistic to assume a uniform UI technology stack.

## 6.7 Variations {#section-scs-variationen}

Instead of implementing an SCS with UI, logic and data there are
different variations. These architectures are no SCS architectures.
However, they might still be sensible in certain contexts.

* Domain microservices with logic and data but *without UI* can
  be sensible when an API or a backend are to be implemented.
  
* Microservices without logic or data, which consequently implement
  *only a UI*, can be a good approach for implementing a portal or a
  different kind of frontend.

Both of these variations are sensible when the project is a pure
frontend or backend project and does not allow the full implementation
of an SCS.

#### Typical Changes

A good architecture should limit a change to one microservice.  The
basic assumption of the SCS architecture is that a change typically
goes through all layers and is limited to a domain implemented in one
microservice.  This assumption has been confirmed in many
projects. Still, if a change in a project usually only affects the UI
or only the logic, it may be better to implement a division by
layers. Then a new look & feel or new colors can be implemented in the
UI layer, while logic changes can be implemented in the logic layer,
of course only if they do not affect the UI.

#### Possibilities for Combinations

Self-contained systems are easy to combine with other recipes.

* Frontend integration (see [chapter 7](#chapter-frontend)) is
prefered when integrating SCSs.

* SCSs focus on asynchronous communication
  ([chapter 10](#chapter-asynchron)). SCSs are web applications so
  that the use of Atom ([chapter 12](#chapter-atom)) for asynchronous
  REST communication is especially easy since it also builds on HTTP.
  Kafka ([chapter 11](#chapter-kafka)) on the other hand has a different
  technical foundation.  Even though it can still be combined with
  SCSs, it then represents an additional technical effort.

* Synchronous communication ([chapter 13](#chapter-synchron)), even
though also possible, should be avoided.
  
Thus, UI integration, asynchronous communication and synchronous
communication between SCSs are possible, but there is a clear
prioritization.  Of course, SCSs can also communicate with
microservices or other systems via these mechanisms.

## 6.8 Conclusion {#section-scs-fazit}

Self-contained systems are an approach for the implementation of
microservices that has proven its usefulness in numerous
projects. They provide best practices on how to implement successful
architectures with microservices. However, they also restrict the
general approaches of microservices and are therefore more
specialized. Nevertheless, at least aspects such as asynchronous
communication and division into different domains are a helpful
approach in many situations.

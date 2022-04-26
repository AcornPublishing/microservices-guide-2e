# 19 Concept: Operation {#chapter-betrieb-grundlagen}

This chapter deals with the operation of microservices and covers the
following points:

* Operations can help to evaluate the business results of changes to
  the software.

* Better operation can accelerate response times to problems, thereby
improving application availability and quality.

* Operation influences micro and macro architecture.

## 19.1 Why Operation is Important {#section-betrieb-gruende}

Microservices change the importance of operation. There are various reasons for this.

#### Many Microservices

In a microservices architecture a system is not a single deployment
monolith. Instead, each module of the system is a separate microservice
that must be individually
deployed, operated, and monitored. So there are much more
applications that need to be deployed and monitored.

If a project runs for a longer period of time, more and more code is
written. This can lead to the microservices growing in size. This is
problematic because the advantages of microservices can be lost. Thus,
new microservices should be built over time so that the size of the
microservices remains constant and just the number of microservices
grows. Thus the challenges of operation become greater with time as
the
number of microservices increases.

Of course, it is not acceptable for the costs of operation to increase
by an order of magnitude. Measures must therefore be taken to keep
expenditure within reasonable bounds. Standardization and automation
are suitable methods for this purpose.

Even if new microservices are created in the system, no manual effort
should be required to integrate the microservices into the
environment. This not only simplifies operation, but also facilitates
the creation of new microservices in order to keep the size of the
microservices constant. For this purpose, a template for a new
microservice that already
contains all necessary technologies for efficient operation can be
helpful.

#### Evaluating Experiments

Every change to a microservices system should aim to achieve a
business objective. For example, the optimization of the user
registration may be aimed at increasing the number of customers who
register.

The change is comparable to a scientific experiment. In a scientific
experiment, a hypothesis is made, then an experiment is carried out,
and the result is measured to check the hypothesis. The same procedure
is used here. The hypothesis is: "The new registration will increase
the number of active users." Then the change must be made and the
results measured.

Collecting and measuring data from applications is a classic task of
operations. Usually, operations confines itself to monitoring system
metrics. In principle, however, the results of the experiments could
also be evaluated with the approaches operations uses.

#### Distributed System

Microservices turn the system into a distributed system. Instead of
local method calls, microservices communicate via the network. This
increases the number of possible error sources. The network and
server hardware may fail. This increases the demands on operations,
which is
responsible for the reliability of all components.

Troubleshooting is also a challenge in a distributed system. If the
microservices call each other, an error in a microservice can be
caused by one of the called microservices. Therefore, there must be a
way to track calls between microservices in order to identify the
source of the error. This tracing is only required in a microservices
environment and poses an additional challenge during operation.
[Chapter 22](#chapter-tracing) shows Zipkin as a technological
solution for this problem.

In order to analyse the microservices, further measures are necessary.
For a deployment monolith, it is enough to examine the processes
and resource consumption on the server using operating system tools.
Log files can also be a good source of information. In a distributed
system, the number of servers is too large to be successful with this
approach. You cannot log on to every server and look for the cause of
the error. So there must be a centralized infrastructure that collects
information from all microservices in one central location.
[Chapter 21](#chapter-log-analyse) describes the Elastic Stack for
managing log files and [chapter 22](#chapter-monitoring) deals with
Prometheus for collecting metrics about all microservices.

#### Ensuring Performance

Each system must offer a certain performance. Capacity tests are used
to avoid performance problems. To do this, the tests have to use a
realistic amount of data, simulate user behavior and run on
production-like hardware, which is hardly feasible:

* The *data volumes* from production often are too large to be
processed in a test environment.

* It can also be difficult to simulate *user behavior*. In the end, it
is unclear how long users wait and think until they perform the next
action or how often they choose a certain course of a process.

* It is often already difficult to set up the production environment.
This applies to sizing as well as to the integration of third-party
systems. Building additional *environments for tests* that are
similarly powerful as the production environment is often impossible.

* For *new features* it is impossible to predict the behavior of the
users and the success of the feature and thus the load. Capacity tests
alone are not an adequate measure to secure the performance of the
application in production since the test scenarios are based on pure
assumptions and therefore cannot produce realistic results.

#### Supplements to Tests

In addition to capacity tests, other measures may be useful.

* Effective *monitoring* can detect a problem with performance in
production at an early stage. This reduces the time it takes to
respond and resolve the problem. Since microservices are individually
scalable, more instances of a microservice can be started to cope with
the load. Ideally, the problem is then solved without a user noticing
anything about it.

* If the problem is more complex, a *fix must be delivered*. For this,
a fast and highly automated deployment is useful, which microservices
typically
bring with them.

Similar considerations apply not only to performance problems, but
also to problems with the implemented logic, for example. Monitoring
and fast deployment can also speed up the response to such problems.
For example, if registration numbers or sales suddenly drop, this can
be an indication of a problem with the implemented logic. Maybe nobody
is actually able to register any more because of an error. Monitoring
is therefore also useful as a supplement to tests of the logic.

#### Dynamic Scaling

One advantage of microservices is that they can be scaled
independently. More or less instances of each microservice can run to
handle the current load. This requires the use of technologies that
allow new instances to be started. Kubernetes (see
[chapter 17](#chapter-kubernetes)) can do this, for example, and uses
resources from an entire cluster for the operation of the
microservices.

But even if such a technology is used, there is only a limited number
of servers available in the cluster. Although the infrastructure
becomes more flexible, capacity planning is still necessary to ensure
scalability.

Therefore, the necessary prerequisites must be created to enable
dynamic scaling of the microservices system.

{id="fig-betrieb-einflussfaktoren"}
![Fig. 19-1: Factors Influencing the Operation of Microservices](images/betrieb-einflussfaktoren.png)

## 19.2 Approaches for the Operation of Microservices {#section-betrieb-ansaetze}

The operation of microservices comprises.

* The *deployment*: This aspect is solved by technologies like Docker (see
  [chapter 5](#chapter-docker)) or microservices platforms
  like Kubernetes (see [chapter 17](#chapter-kubernetes)) or a
  PaaS like Cloud Foundry (see [chapter 18](#chapter-paas)).

* *Monitoring* is a focus of [chapter 20](#chapter-monitoring).

* The *analysis of log files* is described in
[chapter 21](#chapter-log-analyse).

* Finally, [chapter 22](#chapter-tracing) shows how *tracing* can
trace the calls between microservices.

{id="fig-betrieb-herausforderungen"}
![Fig. 19-2: Challenges in the Operation of Microservices](images/betrieb-herausforderungen.png)

#### Independent Deployment

Microservices also bring advantages for the operation.

A microservice is much smaller than a deployment monolith and
therefore easier to deploy. Even if the microservice fails for some
time during deployment, this should not have any dramatic consequences
because of resilience, as the other microservices continue to run.

This requires that the microservices can be deployed independently. If
a feature requires changes to multiple microservices, it has to be
ensured that deployment is still independent. If the client and server
of an interface have to be deployed simultaneously, deployment is no
longer independent. Instead, the microservices can provide the old and
the new interface in parallel to decouple the deployment of the
interface's client and server. This way a new server can be
deployed. The client can still use the old interface. So the client
can be deployed at a later point in time. Eventually, the old
interface can be removed which requires yet another deployment. The
deployment of the individual microservice is easier, but more
deployments are needed to update both client and server.

If the deployments are not decoupled, several microservices or
even all microservices must be deployed in a coordinated manner. This
is difficult because all deployments have to run smoothly and in case
of a failure all deployments have to be rolled back.

Independent deployment is one of the most important advantages of
microservices because it leads to a high degree of independence.
Therefore, you should always strive for independent deployment.

#### Step-by-step Introduction of Operation for Microservices

As discussed, the operation of microservices is of great importance.
In addition to operation, many other fundamental changes are necessary
for microservices. New technologies, frameworks, and architectures must
be implemented. In addition, organizational changes may also be necessary.
Independent and self-organized teams
should develop independent microservices to derive maximum
benefit from the microservices architecture. With so many changes, the risk of
problems increases.

When the first microservice is supposed to go into production, it is
not yet necessary to have the level of automation and sophistication
that
would be required for running a great number of microservices. It is
therefore conceivable to gradually build up the necessary operation
technologies. However, this implies that if the setup of the 
environments cannot keep pace with the number of microservices, the
microservices system becomes unreliable and costly to operate.
At one point manual processes are by far too expensive to support the
huge number of microservices.

## 19.3 Effects of the Discussed Technologies {#section-betrieb-technologien-auswirkungen}

The technologies discussed so far have an impact on operation.

* *Docker* (see [chapter 5](#chapter-docker)) allows a very easy
installation of software. The complete environment including the Linux
distribution is included in the Docker image.
The image just needs to be pulled from a repository to install a
microservice.
At the same time, Docker
is very efficient, so not too much additional hardware is needed.

* *Links and client-side integration*
([chapter 8](#chapter-links-javascript)) only means that several web
applications need to be run. Most companies already run web
applications so that only more environments of the same type need to
be run.

* With *UI integration on the server*, for example with ESI (Edge Side
Includes, see [chapter 9](#chapter-esi)), an additional server must be
operated for the integration. For ESI this might be a Varnish
cache. With Server
Side Includes, a web server is enough, which may already be used
in the system anyway and only needs to be configured appropriately.

* *Kafka* ([chapter 11](#chapter-kafka)) or other Message-oriented
Middlewares (MOMs) introduce powerful, but also complex software for
asynchronous communication. Accordingly, operation is also complex. A
failure or problem of the MOM affects the entire microservices system.
Therefore, this alternative is a challenge for operations.

* In contrast, asynchronous communication with *Atom*
([chapter 12](#chapter-atom)) uses HTTP and REST. This means that no
other infrastructure is required compared to a synchronous REST
system. So
operation is identical to that of a REST or web application.

* The *Netflix stack* ([chapter 14](#chapter-netflix)) implements all
necessary infrastructure with Java. Spring Cloud enables uniform
configuration of microservices and infrastructure services from the
Netflix stack. Especially with a Java microservices system, the
operation of the entire system can be simplified in this way. On the
other hand, the Netflix stack uses its own custom solution for routing
instead of a web server, with which operations has already gained
experience in most cases.

* The *Consul stack* (see [chapter 15](#chapter-consul)) introduces on
the one hand with Consul a Go application which might cause more
effort for operation compared to a system which just uses Java. On the
other hand, Consul is so flexible that a
configuration of web servers like Apache httpd is possible without any
problems. The example uses the Apache httpd for routing, but Apache
httpd could also be used for load balancing between microservices.
This makes it possible to use technologies that companies usually
already
know, which minimizes the risk.

* *Kubernetes* or a *PaaS* such as *Cloud Foundry* offer a complete
solution for the operation of microservices. However, they are also
complex and take over functions from existing systems such as
virtualization software. It is therefore a major step to put
them into production. But then they offer many advantages.

## 19.4 Conclusion {#section-betrieb-grundlagen-fazit}

Although the subject of "operation" is at the end of the book, it is
an important subject. Without a good operation, the many microservices
cannot be brought into production. The reliability and performance of a
microservices solution also depends heavily on the operation, which is
why it is so important.

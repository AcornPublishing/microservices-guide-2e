# 16 Concept: Microservices Platforms {#chapter-plattformen}

The following chapters describe microservices platforms. Readers learn
that:

* Microservices platforms provide support for the operation and also
for the communication of microservices.

* PaaS (Platform as a Service) cloud offerings and Docker scheduler
are examples for microservices platforms.

* Microservices platforms have their own advantages and disadvantages,
which make them superior to other approaches in some scenarios.

## 16.1 Definition {#section-plattformen-definition}

The platforms in the next chapters differ from all other technologies
presented so far in that they not only enable the communication of
microservices, but also support aspects of operation such as
deployment, monitoring, or log analysis.

#### Support for HTTP and REST

The platforms support HTTP and REST with load balancing, routing,
and service discovery. But they can also be supplemented with
other communication mechanisms. This makes it possible to use the
platforms for setting up asynchronous systems. However, this chapter mainly considers synchronous communication. Microservices on platforms that use this type of communication differ most from microservices on different infrastructures concerning their mode of operation.

{id="fig-plattformen-konzept"}
![Fig. 16-1: Features of Microservices Platforms](images/plattformen-konzept.png)

#### Expenditure for Installation and Operation

Microservices platforms are very powerful, but consequently also very
complex. They make it much easier to work with microservices, but
installing and operating the platform itself can really be a challenge. If
the platform is running in the public cloud, the complex installation
and operation does not play a role, because the operator of the public
cloud has to take care of it. However, if an installation is carried
out in the company's own data centre, the operations team has to
make the effort.

However, the effort for installing the platform has to be made only
once. After that, deploying the microservices is much easier. This
means that the cost of installing the platform will be amortized
quickly.

Only limited operations support is then necessary when
installing the microservices. In this way, microservices can be
deployed quickly and easily even if operations cannot support each
deployment of each microservice.

#### Migration to a Microservices Platform

In contrast to the solutions shown so far, microservices platforms
require a fundamental change in the operation and installation of the
applications. The other examples can be run on virtual machines or
even physical servers and can be combined with existing deployment
tools. Since the microservices platforms also cover the deployment and
operation of the microservices, they include features for which
operations has already established tools in most cases. Therefore, the
use of a microservices platform is a bigger step than the use of other
technologies.

Such a move can be deterrent for conservative operations teams. In
addition, the introduction of microservices often leads to
organizational changes and many new technologies in addition to a new
architecture. It can be helpful if a complex microservices platform
does not also have to be introduced in this situation.

#### Influence on the Macro Architecture

Of course, a platform only supports certain technologies. In addition
to the programming language, these are above all the technologies for
deployment, monitoring, logging, and so on. The platform therefore
defines large parts of the technical infrastructure.

Therefore, microservices platforms have a relationship to the macro
architecture (see [chapter 2](#chapter-mikro-makro)).

* The macro architecture should state the *requirements of the
platform*. This ensures that the platform can actually run the
microservices. So the platform specifies, for example, deployment and
logging in the macro architecture.

* The microservices platform *restricts the options* the team can
choose in the macro architecture. After all, it is impossible to
operate microservices that cannot be run on the platform. For example,
the platform can define restrictions concerning the programming
language.

* The microservices platform can *enforce* compliance with macro
architecture rules. If developers disregard the rules, the
microservices simply cannot run on the platform. This ensures that the
rules are actually observed.

#### Specific Platforms

The two following chapters show two approaches for microservices
platforms.

* *Kubernetes* (see [chapter 17](#chapter-kubernetes)) can run Docker
containers and solves challenges such as load balancing, routing,
and service discovery at the network level. It is quite flexible
since it is able to run arbitrary Docker containers. With
[Operators](http://coreos.com/operators) or 
[Helm](https://helm.sh/) other services can be integrated into
Kubernetes, for example,
for monitoring.

* Cloud Foundry (see [chapter 18](#chapter-paas)) supports applications.
All you have to do is provide Cloud Foundry with e.g. a Java
application. Cloud Foundry creates a Docker container from it, which
can then be executed. Cloud Foundry also solves load balancing, routing, and
service discovery. In addition, Cloud Foundry includes additional
infrastructure such as databases.

## 16.2 Variations {#plattform-variationen}

Microservices platforms appear to be particularly suitable for
synchronous microservices and REST communication, as they offer
particularly good support for this. However, the platforms can be
extended to allow other communication mechanisms. Frontend integration
can be implemented with these platforms. Client frontend integration
is completely independent of the platform used. Only with server-side
frontend integration would the server have to be installed and
operated on the platform.

The platform can also cover the operational aspect for asynchronous
microservices. Better support for the operation of the microservices
alone is a good reason to use the platforms. Operation is
one of the most important challenges with microservices. This aspect is
independent of the communication mechanism used.

#### Physical Hardware

So the question arises whether there are other environments for the operation of
microservices. A theoretical alternative to the platforms is physical
hardware. However, physical hardware is hardly used any more for cost
reasons.

#### Virtual Hardware

Also virtual hardware is inflexible and heavyweight, as already
discussed in [section 4.7](#section-docker-variationen). So the only
alternative to a platform is "Docker without scheduler",
as discussed in [section 4.7](#section-docker-variationen). In this scenario Docker
containers are installed on classic servers.

In this case the macro architecture standardizes the operation (see
[section 2.2](#section-mikro-makro-technisch)) to use only one
technology for log analysis or monitoring and thus to work efficiently.
Microservices platforms already have such features. So you do not have
to build support for logging or monitoring yourself, but you can use
these parts of the platform.
This can be the simpler solution because implementing log analysis
for many microservices can be very costly.

Resilience and other features such as load balancing have to be
implemented at the virtual machine level since the Docker
infrastructure does not offer these. At the end of the day, it can
happen that the features of a microservices platform such as log
analysis, monitoring, reliability, and load balancing are recreated
step by step instead of just installing and using a prepackaged
microservices platform.

## 16.3 Conclusion {#section-plattform-fazit}

Microservices platforms appear to be very helpful due to the
large number of features, even though the operation of these platforms
can be complex. In practice, Kubernetes is a very important platform
for the operation of microservices, while PaaS such as Cloud Foundry
are not in focus despite their convincing features.

Microservices platforms should definitely be considered for
microservices because they represent a significant simplification and
complete solution to typical microservices challenges. The only reason
against a microservices platform is the high cost of installation. In
case of a small number of microservices or at the beginning of a
project, the installation of a microservices platform can be bypassed.
In addition, this effort is eliminated if a public cloud offer is
used.

# 0 Introduction {#chapter-introduction}

Microservices are one of the most important software architecture
trends. By now there is already a number of detailed guides to microservices,
among them the [microservices-book](http://microservices-book.com)
written by the author of this volume. So why do we need still another
book on microservices?

It is one thing to define an architecture, it is quite another thing
to implement it. This book presents technologies for the
implementation of microservices and highlights the associated benefits
and disadvantages.

The focus rests specifically on technologies for entire microservices
systems. Each individual microservice can be implemented
using different technologies. Therefore, the technological decisions for frameworks
for individual microservices are not as important as the decisions at the
level of the overall system. For individual microservices the decision
for a framework can be quite easily revised. However, technologies
chosen for the overall system are hard to change.

This means compared to the
[microservices-book](http://microservices-book.com) this book talks
primarily about technologies. This does discuss architecture and
reasons for or against microservices - but only briefly.

#### Basic Principles

To become familiar with microservices, an introduction into
microservices-based architectures and their benefits, disadvantages
and variations is essential. However, this book explains the basic
principles only to the extent to which they are required for
understanding the practical implementations.
A more complete discussion is part of the
[microservices-book](http://microservices-book.com).

#### Concepts

Microservices require solutions for different challenges. Among those
are concepts for integration (*frontend integration, synchronous and
asynchronous microservices*) and for operation (*monitoring, log
analysis, tracing*).  Microservices platforms such as *PaaS* or
*Kubernetes* represent exhaustive solutions for the operation of
microservices.

#### Recipes

The book uses recipes as metaphor for the technologies which can be
used to implement the different concepts. Each approach shares a
number of features with a recipe.

* Each recipe is described in *practical* terms, including an
 exemplary technical implementation.  The most important aspect of the
 examples is their *simplicity*.  Each example can be easily followed,
 extended and modified.

* The book provides the reader with a *plethora of recipes*. The
 readers have to *select* a specific recipe from this collection for
 their projects, akin to a cook who has to select a recipe for her/his
 menu.  The book shows different options. In practice, nearly every
 project has to be dealt with differently. The recipes build the basis
 for this.

* For each recipe there are *variations*.  After all, a recipe can be
  cooked in many different ways. This is also true for the
  technologies described in this book.  Sometimes the variations are
  very simple so that they can be immediately implemented as
  *experiments* in an executable example.

For each recipe there is an *executable example* based on a concrete
technology. The examples can be run individually. They are not based
on each other.  This allows the readers to concentrate on the recipes
which are interesting and useful for them and to skip the examples
which are less relevant for their work.

In this manner, the book provides an easy *access* for obtaining an
*overview* of the relevant technologies and thereby enables the
readers to select a suitable technology stack. Subsequently, the
readers can use the links supplied in the book to acquire in depth
knowledge about the relevant technologies.

#### Source Code

Sample code is provided for almost all of the technologies presented
in this book. If the reader wants to really understand the
technologies, it makes sense to browse the code. It also makes sense
to look at the code to understand how the concepts are actually
implemented. The reader might even want to have the source right next
to the book to read both code and this book.

## 0.1 Structure of the Book {#section-introduction-structure}

This book consists of three parts.

#### Part I --- Architecture Basics

[Part I](#chapter-basic) introduces the basic principles of a microservices-based architecture.

{id="fig-preface-teil-I", width=50%}
![Overview of Part I](images/preface-teil-I.png)

* [Chapter 1](#chapter-microservices) defines the term *microservice*.

* A microservices architecture has two levels: micro and
macro architecture. They represent global and local decisions as explained in
[chapter 2](#chapter-mikro-makro)).

* Often old systems are supposed to be migrated into microservices, a topic which
is covered in [chapter 3](#chapter-migration).

#### Part II --- Technology Stacks

*Technology stacks* are the focus of [part II](#chapter-techstacks).

{id="fig-preface-teil-II"}
![Overview of Part II](images/preface-teil-II.png)

* *Docker* serves as basis for many microservices architectures
  ([chapter 4](#chapter-docker)). It facilitates the roll-out of
  software and the operation of the services.

* The *technical micro architecture*
  ([chapter 5](#chapter-technisch-mikro)) describes technologies which
  can be used for implementing microservices.

* [chapter 6](#chapter-scs) explains *self-contained systems (SCS)* as
an especially useful approach for microservices. It focuses on
microservices that also include a UI as well as logic.

* One possibility for integration in particular for SCS is
 *integrating at
 the web frontend* ([chapter 7](#chapter-frontend)).  Frontend
 integration results in a lose coupling between the microservices and
 a high degree of flexibility.

* The recipe for web frontend integration presented in
 [chapter 8](#chapter-links-javascript) 
 capitalizes on *links* and
 *JavaScript* for dynamical content loading.  This approach is easy
 to implement and utilizes well established web technologies.

 * On the server, integration can be achieved with *ESI (Edge Side
  Includes)* ([chapter 9](#chapter-esi)).  ESI is implemented in web
  caches so that the system can attain high performance and
  reliability.

* The concept of *asynchronous communication* is the focus of
 [chapter 10](#chapter-asynchron).  Asynchronous communication
 improves reliability and decouples the systems.

* *Apache Kafka* is an example for an asynchronous technology
  ([chapter 11](#chapter-kafka)) for sending messages.  Kafka
  can save messages permanently and thereby enables a different
  approach to asynchronous processing.

* An alternative for asynchronous communication is *Atom*
  ([chapter 12](#chapter-atom)).
  Atom uses a REST infrastructure and thus is very easy to implement
  and operate.

* [Chapter 13](#chapter-synchron) illustrates how the concept of
 *synchronous microservices* can be implemented.  Synchronous
 communication between microservices is often used in practice
 although this approach can pose challenges in regards to
 response times and reliability.

* The *Netflix Stack* ([chapter 14](#chapter-netflix)) offers Eureka
  for service discovery, Ribbon for load balancing, Hystrix for
  resilience and Zuul for routing.  The Netflix stack is especially widely used
  in the Java Community.

* *Consul* ([chapter 15](#chapter-consul)) is an alternative option for
  service discovery.  Consul contains numerous features and can be
  used with a broad spectrum of technologies.

* [Chapter 16](#chapter-plattformen) explains the concept of
  *microservices platforms* which support operation and communication
  of microservices.

* *Kubernetes* ([chapter 17](#chapter-kubernetes)) is an
  infrastructure that can be used as a microservices
  platform which is able to execute Docker containers and also has
  solutions for service discovery and load balancing.  The
  microservice remains independent of this infrastructure.

* *PaaS (Platform as a Service)* is another infrastructure that can be
    used as a microservices platform
    ([chapter 18](#chapter-paas)). It is  illustrated using Cloud
    Foundry as an example.  Cloud Foundry is very flexible and can be
    run in your own computing center as well as in the public cloud.


#### Part III --- Operation

It is a huge challenge to ensure the *operation* of a plethora of
microservices. [Part III](#chapter-betrieb) discusses possible recipes
that address this challenge.

{id="fig-preface-teil-IIi"}
![Overview of Part III](images/preface-teil-III.png)

* [Chapter 19](#chapter-betrieb-grundlagen) explains *basic
  principles* of operation, and why it is so hard to operate microservices.

* [Chapter 20](#chapter-monitoring) deals with *monitoring* and
  introduces the tool Prometheus.  Prometheus supports multi
  dimensional data structures and can analyze metrics  even
  of numerous microservice instances.

* [Chapter 21](#chapter-log-analyse) concentrates on the *analysis of log
  data*.  As tool the Elastic Stack is presented. This stack is very
  popular and represents a good basis for analyzing large amounts of log data.

* *Tracing* allows one to trace calls between microservices
 ([chapter 22](#chapter-tracing)). This is often done with the help of
 Zipkin.  Zipkin supports different platforms and represents a
 de facto standard for tracing.

* *Service meshes* add proxies to the network traffic between
  microservices. This enables them to support monitoring, log
  analysis, tracing and other features like resilience or
  security. [Chapter 23](#chapter-service-mesh) explains Istio as an
  example of a service mesh.

#### Conclusion and Appendices

At the end of the book [chapter 24](#chapter-und-nun) offers an
*outlook*.

The appendices explain the software installation
([appendix A](#anhang-installation)), how to use the build tool
Maven ([appendix B](#anhang-maven)) and also Docker and Docker Compose
([appendix C](#anhang-docker)) which can be used to run the
environments for the examples.

## 0.2 Target Group {#section-introduction-target-group}

The book explains basic principles and technical aspects of
microservices. Thus, it is interesting for different audiences.

* For *developers*, [part II](#chapter-techstacks) offers a guideline
  for selecting a suitable technology stack.  The example projects
  serve as basis for learning the foundations of the technologies. The
  microservices contained in the example projects are written in Java
  using the Spring Framework. However, the technologies used in the
  examples serve to integrate microservices. So additional
  microservices can be written in different languages.
  [Part III](#chapter-betrieb) completes the book by
  including the topic operation which becomes more and more important
  for developers. [Part I](#chapter-basic) explains the basic
  principles of architecture concepts.

* For *architects*, [part I](#chapter-basic) contains fundamental
  knowledge about microservices. [Part II](#chapter-basic) and
  [III](#chapter-betrieb) present practical recipes and technologies
  for implementing microservice architectures.  With these topics this
  book goes much more into depth than other
  books which just focus on architecture, but do not cover technologies.
  
* For experts in *DevOps* and *operations*, the recipes in
  [part III](#chapter-betrieb) represent a sound basis for a
  technological evaluation of operational aspects such as log
  analysis, monitoring and tracing of
  microservices. [Part II](#chapter-techstacks) introduces
  technologies for deployment such as Docker, Kubernetes or Cloud
  Foundry that also solve some operational
  challenges. [Part I](#chapter-techstacks) provides background about
  the concepts behind the microservices architecture
  approach.

* *Managers* are presented with an overview of the advantages and
  specific challenges of the microservices architecture approach
  in [part I](#chapter-techstacks).  If they are interested in
  technical details, they will benefit from reading
  [part II](#chapter-techstacks) and [III](#chapter-betrieb) .
  

## 0.3 Prior Knowledge {#section-einleitung-vorwissen}

The book assumes the reader to have basic knowledge of software
architecture and software development. All practical examples are
documented in such a way that they can be executed with very little
prior knowledge.  The book focuses on technologies which can be
employed for microservices using different programming languages. However, the
examples are written in Java using the Spring Boot and Spring Cloud
frameworks so that changes to the code require knowledge of Java.

## 0.4 Quick Start {#section-einleitung-quick-start}

The book focuses first of all on introducing technologies. For each
technology in each chapter there is an example system. In addition, there is
a quick start to allow the reader to
rapidly gain practical experience with the different technologies and
to understand how they work with the help of the examples.

* First, the necessary software has to be *installed* on the computer.
 The installation is described in [appendix A](#anhang-installation).

* The build of the examples is done with
  *Maven*. [Appendix B](#anhang-maven) explains how to use Maven.

* All of the examples use *Docker* and *Docker Compose*.
 [Appendix C](#anhang-docker) describes the most
 important commands for Docker and Docker Compose.

For the Maven-based build and also for Docker and Docker Compose the
chapters contain basic instructions and advice on trouble shooting.

The examples are explained in the following sections:

| Concept                    | Recipe                           | Section |
|--------------------------  | --------------------------------------------|
| Frontend Integration       | Links & Client-side Integration  | [8.2](#section-links-javascript-beispiel) |
| Frontend Integration       | Edge Side Includes (ESI)         | [9.2](#section-esi-beispiel) |
| Asynchronous Microservices | Kafka     		     	| [11.4](#section-kafka-beispiel) |
| Asynchronous Microservices | REST & Atom 			| [12.2](#section-atom-beispiel) |
| Synchronous Microservices  | Netflix Stack 			| [14.1](#section-netflix-beispiel) |
| Synchronous Microservices  | Consul &  Apache httpd 		| [15.1](#section-consul-beispiel) |
| Microservices Platform     | Kubernetes       		| [17.3](#section-kubernetes-detail) |
| Microservices Platform     | Cloud Foundry 			| [18.3](#section-paas-beispiel) |
| Operation 		     | Monitoring with Prometheus 	| [20.4](#section-monitoring-beispiel) |
| Operation 		     | Log Analysis with Elastic Stack 	| [21.3](#section-log-analyse-beispiel) |
| Operation 		     | Tracing with Zipkin     		| [22.2](#section-tracing-zipkin) |
| Operation 		     | Service Mesh with Istio   	| [23.2](#section-service-mesh-example) |

All projects are available on GitHub. Within the projects there is
always a file `HOW-TO-RUN.md` containing step by step instructions how
the demos can be installed and started.

The examples are independent of each other. So it is possible to start
with any one of them.

## 0.5 Acknowledgements {#section-einleitung-danksagung}

I would like to thank everybody who discussed microservices with me,
who asked me about them, or worked with me. Unfortunately, these are
way too many to name them all individually. The exchange of ideas is
enormously helpful and also fun!

Many of the ideas and also their implementation would not have been
possible without my colleagues at INNOQ. I would especially like to
thank
Alexander Heusingfeld,
Christian Stettler,
Christine Koppelt,
Daniel Westheide,
Gerald Preissler,
Hanna Prinz,
Jörg Müller,
Lucas Dohmen,
Marc Giersch,
Michael Simons,
Michael Vitz,
Philipp Neugebauer,
Simon Kölsch,
Sophie Kuna,
Stefan Lauer,
and
Tammo van Lessen.

Also Merten Driemeyer
and
Olcay Tümce provided important feedback.

Finally, I would like to thank my friends and family which I often
neglected while writing this book - especially my wife. She also did
the translation into English.

Of course, my thanks go also to the people who developed the
technologies which I introduce in this book and thereby created the
foundation for microservices.

I also would like to thank the developers of the tools of
<https://www.softcover.io/> and Leanpub.

Last but not least, I would like to thank my publisher dpunkt.verlag and René Schönfeldt
who professionally supported me during the creation of the German
version of this book.

## 0.6 Website {#section-einleitung-website}

The website accompanying this book can be found at
<http://practical-microservices.com/>. It contains links to the examples
and also errata.

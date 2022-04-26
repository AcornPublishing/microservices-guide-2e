# 5 Technical Micro Architecture {#chapter-technisch-mikro}

One of the strengths of microservices is that different technologies
can be used in each individual microservice. The technologies in the
microservices can be defined
as part of the micro architecture (see
[chapter 2](#chapter-mikro-makro)).

However, there are technical challenges to consider when selecting
technologies for microservices.

This chapter explains how to deal with the technical micro
architecture.

* The reader gets to know the *requirements* in regards to e.g.
operation or resilience which the micro architecture has to
fulfill.

* Often microservices are implemented with *reactive technologies*.
Thus, the chapter discusses this option in more detail and explains
when this approach makes sense.

* As a concrete example of a technical micro architecture, the chapter
shows *Spring Boot* and *Spring Cloud*, which are used in the following
chapters for most examples.

* Based on Spring Boot and Spring Cloud, the chapter shows how the
  *technical requirements* the micro architecture has to address can
  be fulfilled.

* In addition, the chapter shows how the programming language *Go* in
conjunction with appropriate frameworks fulfills the requirements for
implementing microservices.

## 5.1 Requirements {#section-technisch-mikro-anforderungen}

A technology for implementing microservices has to fulfill different requirements.

{id="fig-technisch-mikro-einflussfaktoren", width=60%}
![Fig. 6-1: Influencing Factors for the Technical Micro Architecture](images/technisch-mikro-einflussfaktoren.png)


#### Communication

Microservices have to communicate with other microservices. This
requires a UI integration in the
web UI or protocols such as REST or  messaging. It is a macro
architecture decision which communication
protocol is used (see [section 2.2](#section-mikro-makro-technisch)).
However, the microservices have to support the chosen communication
mechanism. Therefore, the macro architecture decision influences the
micro architecture. The technology choices at the micro architecture
level have to ensure that the communication protocol defined by the
macro architecture can really be implemented in each microservice.

In principle, every modern programming technology can support the
typical communication protocols. Therefore, this requirement does not
represent a real restriction.

#### Operation

Operating the microservices should be as easy as possible. Topics in
this area are:

* *Deployment*: The microservice has to be installed in an environment
and has to run in this environment.

* The application's *configuration*: The microservice has to be
adapted to different scenarios. It is possible to use custom code for
reading the configuration. However, an existing library can
facilitate this task and promote a uniform configuration.

* *Logs*: Writing log files is easy. However, the format should be
uniform for all microservices. In addition,
[chapter 21](#chapter-log-analyse) shows that a simple log file is not
enough when a server has to collect the logs from all
microservices and provide them for analysis. Therefore, technologies
have to be in place for formatting the log outputs and for sending
them to the server where all logs are stored and analyzed.

* *Metrics* have to be delivered to the central monitoring
infrastructure. This requires appropriate frameworks and libraries.

In principle, different libraries can be used for implementing a macro
architecture rule which for instance predefines a log format and a log
server. In this case, the micro architecture has to choose a library
for the microservice. Macro architecture rules can also determine the
library. However, this limits the technological freedom of the
microservices to those programming languages which can use the chosen
library.

#### New Microservices

It should be easy to create new microservices. When a project over
time accumulates more and more code, there are two options.
Either the microservices become larger or the number of microservices
of constant size increases. If the microservices increase in size, at
some point they will not *deserve* the name *microservice* anymore. To
avoid this, it is important that it is easy to generate new
microservices to keep the size of the individual microservices
constant over time.

#### Resilience

Each microservice has to be able to deal with the failure of other
microservices. This has to be ensured when microservices are
implemented.

## 5.2 Reactive {#section-technisch-reactive}

One way to implement a microservice is *reactive
programming*. Oftentimes it is stated that microservices must be
implemented with reactive technologies. This section discusses what
reactive actually is and determines whether reactive technologies are
truly needed for microservices.

Similar to microservices, *reactive* has an ambiguous
definition. The
[Reactive Manifesto](http://www.reactivemanifesto.org/) defines the
term "reactive" based on the following characteristics:

* *Responsive* means that the system responds as fast as possible.

* Because of *resilience* the system remains available even if parts
fail.

* *Elastic:* The system can deal with different levels of load, for
instance by using additional resources. After the load peak
the resources are freed again.
  
* The system uses asynchronous communication (*message driven*).

These characteristics are useful for microservices. They pretty much
correspond to the features discussed in
[chapter 1](#chapter-microservices) as essential characteristics of
microservices.

So at first sight it seems that microservices in fact must be written
with reactive technologies.

#### Reactive Programming

However,
[reactive programming](https://en.wikipedia.org/wiki/Reactive_programming)
means something completely different. This programming concept
resembles the data flow. When new data comes into the system, it is
processed. A spreadsheet is an example. When the user changes a value
in a cell, the spreadsheet recalculates all dependent cells.

#### Classical Server Applications

A similar approach is possible for server applications. Without
reactive programming, a server application typically processes an
incoming request in a thread. If the processing of the request
requires a call to a database, the thread blocks until the result
of this call arrives. In this model, a thread has to be provided for
each request that is processed in parallel and for each network
connection.

#### Reactive Server Applications

Reactive server applications behave very differently. The application
only reacts to events. It must not block because it is waiting for
instance for I/O. Thus an application waits for an event such as an
incoming HTTP request. If a request arrives, the application executes
the logic and then sends a call to the database at some point.
However, subsequently, the application does not wait for the result of
the call to the database, but suspends processing the HTTP request.
Eventually, the next event arrives, namely the result of the call to
the database. The processing of the HTTP request then resumes. In
this model, only one thread is needed. It processes the respective
current event.

[Figure 6-2](#fig-technisch-mikro-reactive) shows an overview of this
approach. The event loop is a thread and processes one event at a
time. Instead of waiting for I/O, the processing of the event is
suspended. Once the results of the I/O operation are available, they
are part of a new event which is processed by the event loop. In this
way, a single event loop can process a plethora of network connections.
However, processing of the event must not block the event loop for
longer than is absolutely necessary. Otherwise processing of all
events will be stopped.

{id="fig-technisch-mikro-reactive"}
![Fig. 6-2: Event Loop](images/technisch-mikro-reactive.png)

#### Reactive Programming and the Reactive Manifesto

Reactive programming can support the goals of the
Reactive Manifesto:

* *Responsive*: The model can cause the application to respond faster
because fewer threads are blocked. However, whether this really leads
to an advantage over a classical application depends on how
efficiently the threads are implemented in the system and how
efficiently it handles blocked threads.

* *Resilience*: If a service no longer responds, nothing is blocked in
reactive programming. This helps with resilience. However, for
example, in a classical application a timeout can avoid a blockage by
aborting the processing of the request.

* *Elastic*: With a higher load, more and more instances can be
started. This is also possible with the classical programming model.

* *Message driven*: Reactive programming does not affect the
communication between the services. Therefore, communication can or
cannot be message driven in reactive programming as well as in
classical applications.

#### Reactive Programming is not Necessary for Microservices 

The Reactive Manifesto is certainly relevant for microservices. But a
microservice does not have to be implemented with reactive programming
in order to achieve the goals of the Reactive Manifesto.

Whether or not a microservice is implemented with reactive programming
can be different for each microservice. This can be a micro
architecture decision and therefore affects only individual
microservices, but not the system as a whole.

It is important to understand the difference because otherwise the
choice of technologies might be limited to reactive programming
frameworks even though that is not necessary. It is perfectly fine to
stay with established technologies. In fact using a technology stack
that you are used to might be easier and bring faster results. At the
same time it is possible to try new technologies like reactive
programming in one microservice and then use it in other microservices
if it has proven to be useful.

## 5.3 Spring Boot {#section-technisch-mikro-spring-boot}

The Spring Framework has long been part of the Java community. It has
a broad set of features covering most of the technical requirements of
typical Java applications.
[Spring Boot](https://projects.spring.io/spring-boot/) facilitates the
use of Spring.

A minimal Spring Boot application can be found in the directory
`simplest-spring-boot` of the project
<https://github.com/ewolff/spring-boot-demos>.

#### Java Code

The Java code from the project shows how Spring Boot can be used.

{linenos=off, lang="java"}
~~~~~~~~
@RestController
@SpringBootApplication
public class ControllerAndMain {

  @RequestMapping("/")
  public String hello() {
    return "hello\n";
  }

  public static void main(String[] args) {
    SpringApplication.run(ControllerAndMain.class, args);
  }

}
~~~~~~~~

The annotation `@RestController` means that the class
`ControllerAndMain` should process HTTP requests.
`@SpringBootApplication` triggers the automatical configuration of the
environment. So the application starts an environment with a web
server and with the parts of the Spring framework that are fitting for
a web application.

The method `hello()` is annotated with `@RequestMapping`. Therefore it
is
called upon an HTTP request to the URL `"/"`. The method's return
value is returned in the HTTP response.

Finally, the main program `main` starts the application with the help
of the class `SpringApplication`. The application can simply be
started as a  Java application even though it processes HTTP requests.
A web server is required for handling HTTP in the Java world. It is
included in the
application.

#### Build

For compiling the project Spring Boot supports among others
[Maven](https://maven.apache.org/).

{linenos=off, lang="xml"}
~~~~~~~~
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.ewolff</groupId>
  <artifactId>simplest-spring-boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.2.RELEASE</version>
  </parent>

  <properties>
    <java.version>10</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
~~~~~~~~

The build configuration inherits settings from the parent
configuration `spring-boot-starter-parent`. Maven's parent configuration
makes it easy to reuse settings for the build of multiple projects.
The version of the parent determines which version of Spring Boot is
used. The Spring Boot version defines the version of the Spring
framework and the versions of all other libraries. Thus, the developer
does not have to define a stack with compatible versions of all
frameworks, which is otherwise often a challenge.

#### Spring Boot Starter Web as Single Dependency

The application has a dependency on the library
`spring-boot-starter-web`. This dependency integrates the Spring
framework, the Spring web framework, and an environment for the
processing of HTTP requests. The default for the processing of the
HTTP requests is a Tomcat server which runs embedded as part of the
application.

Thus, the dependency on `spring-boot-starter-web` would be enough
as
sole dependency for the application. The dependency on
`spring-boot-starter-test` is necessary for tests. The code
for the test is not shown in this chapter.

#### Spring Cloud

[Spring Cloud](http://projects.spring.io/spring-cloud/) is a
collection of extensions for Spring Boot which are useful for cloud
applications and for microservices. Spring Cloud contains additional
starters. To be able to use the Spring Cloud starters, an entry has to
be inserted into the `dependency-management` section in the `pom.xml`
for importing the information about the Spring Cloud starter. The
`pom.xml` files in the examples already contain the required import
for this.

#### Maven Plug In

The Maven plug in `spring-boot-maven-plugin` is necessary to build a
Java JAR that starts an environment with the Tomcat server and the
application. `mvn clean package` deletes the old build results and
builds a new JAR. JARs are a Java file format which contains all the
code for an application. Maven gives this file a name that is derived
from the project name. It can be started with
 `java -jar simplest-spring-boot-0.0.1-SNAPSHOT.jar`. Spring Boot can
also generate WARs (web archives) which can be deployed on a Java web
server like Tomcat or a Java application server.

#### Spring Boot for Microservices?

The suitability of Spring Boot for the implementation of microservices
can be decided according to the criteria of
[section 5.1](#section-technisch-mikro-anforderungen).

#### Communication

For communication, Spring Boot supports REST, as the listing above shows.
The listing uses the Spring MVC API. Spring Boot also supports
the JAX RS API. For JAX RS, Spring Boot uses the library Jersey. JAX
RS is standardized as part of the Java Community Process (JCP).

For messaging, Spring Boot supports the Java Messaging Service (JMS).
This is a standardized API that can be used to address different
messaging solutions from Java. Spring Boot has starters for the JMS
implementations [HornetQ](http://hornetq.jboss.org/),
[ActiveMQ](http://activemq.apache.org/) and
[ActiveMQ Artemis](https://activemq.apache.org/artemis/). In addition,
there is a Spring Boot starter for [AMQP](https://www.amqp.org/). This
protocol is also a standard, but on the network protocol level. The
AMQP starter uses [RabbitMQ](https://www.rabbitmq.com/) as an
implementation of the protocol.

For AMQP as well as for JMS, Spring offers an API that makes it easier
to send messages. In addition, simple Java objects (Plain Old Java
Objects, POJOs) with no dependencies on any of the APIs can process
AMQP and JMS messages with Spring and also
return responses to messages.

Spring Cloud offers
[Spring Cloud Streams](https://cloud.spring.io/spring-cloud-stream/)
for implementing applications for the processing of data streams. This
library supports messaging systems such as Kafka (see also
[chapter 11](#chapter-kafka)), RabbitMQ (see above) and
[Redis](https://redis.io/). Spring Cloud Streams builds on these
technologies and extends them with concepts such as streams and
therefore goes beyond
just simplifying the use of the technology's APIs.

The integration of technologies in Spring Boot with Spring Boot
starters has the
advantage that Spring Boot provides the configuration of the
environment. The example Spring Boot application in this chapter uses
an infrastructure such as a Tomcat server to handle HTTP requests.
This does not require a separate configuration and no additional
dependencies. Spring Boot starters also offer such simplifications
for messaging and other REST technologies.

Spring Boot applications can also use technologies without a Spring
Boot starter. A Spring Boot application can use any technology that
supports Java. In the end, a Spring Boot project is a Java project and
can be extended with Java libraries. However, it is possible that the
configuration can be more complex than when using a Spring Boot
starter.

#### Operation

Spring Boot also has some interesting approaches for operation.

* To deploy a Spring Boot application, it is enough to just copy the
JAR file to the server and start it. Deploying a Java application
can't be further simplified.

* Spring Boot offers numerous options for the
[configuration](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/html/boot-features-external-config.html).
 For example, a Spring Boot application can
  read the configuration from a configuration file or from an
  environment variable. Spring Cloud offers support for Consul (see
  [chapter 15](#chapter-consul)) as server for
  configurations. The examples in this book use
`application.properties` files for configuration because they are
relatively easy to handle.

* Spring Boot applications can generate
[logs](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/html/boot-features-logging.html)
in many different ways. Usually, a Spring Boot application displays the logs
in the console. Output to a file is also possible.
[Chapter 21](#chapter-log-analyse) shows a Spring Boot application
that sends the logs as JSON data to a central server instead of using
a simple human-readable text format. JSON facilitates the processing
of log data on this server.

* For *metrics*, Spring Boot offers a special starter, namely the
[Actuator](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/html/production-ready.html).
After adding a dependency to `spring-boot-starter-actuator`, the
application collects metrics, for example about the HTTP requests. In
addition, Spring Boot Actuator provides REST endpoints under which
the metrics are available as JSON documents. The example in
[chapter 20](#chapter-monitoring) is a Spring Boot application that
exports the data to the monitoring tool Prometheus based on
Actuator. However, Spring Boot Actuator does not support Prometheus.
But custom code can extend Actuator in such a way that other
monitoring systems can also be integrated.

#### New Microservices

Creating a new microservice is very easy with Spring Boot. A build
script and a main class are enough, as shown in the example
[`simplest-spring-boot`](https://github.com/ewolff/spring-boot-demos/tree/master/simplest-spring-boot).
To further simplify the creation of a new microservice, a template can
be created. The template only needs to be adapted for a new
microservice. Settings for the configuration of the microservices or for
logging can be defined in the template. Thus, a template simplifies
the creation of new microservices and facilitates compliance with
macro architecture rules.

A particularly easy way to create a new Spring Boot project is to use
<http://start.spring.io/>. The developer must select the build tool,
the programming language, and a Spring Boot version. In addition,
he/she can select different starters. Based on this, the website then
creates a project that can be the basis for the implementation of a
microservice.

#### Resilience

For resilience a library like Hystrix (see
[section 14.5](#section-netflix-hystrix)) can be useful. Hystrix
implements typical resilience patterns such as timeouts in Java.
Spring Cloud offers an integration and further simplification for
Hystrix.

## 5.4 Go {#section-technisch-mikro-go}

[Go](https://golang.org/) is a programming language that is
increasingly being used for microservices. Similar to Java, Go is
based on the programming language C. But in many areas Go is
fundamentally different.

#### Code

A part of the example in [chapter 9](#chapter-esi) is implemented with
Go.

This Go program responds to HTTP requests with HTML code.

{linenos=off, lang="Go"}
~~~~~~~~
package main

import (
  "fmt"
  "time"
  "log"
  "net/http"
)

func main() {
  http.Handle("/common/css/",
   http.StripPrefix("/common/css/",
    http.FileServer(http.Dir("/css"))))
  http.HandleFunc("/common/header", Header)
  http.HandleFunc("/common/footer", Footer)
  http.HandleFunc("/common/navbar", Navbar)
  fmt.Println("Starting up on 8180")
  log.Fatal(http.ListenAndServe(":8180", nil))
}

// Header and Navbar left out

func Footer(w http.ResponseWriter, req *http.Request) {
  fmt.Fprintln(w,
   `<script src="/common/css/bootstrap-3.3.7-dist/js/bootstrap.min.js" />`)
}
~~~~~~~~

The key word `import` imports some libraries, among others for HTTP.
The main program `main` defines which methods should respond to which
URLs. For example, the method `Footer` returns HTML code. On the other
hand, for the URL `/common/css` the application delivers content from
files.

As you can see, also with Go it is very easy to implement a REST
service. In addition, libraries like
[Go kit](https://github.com/go-kit/kit) offer many more
functionalities to implement microservices.

#### Build

Go compilers are particularly well suited for Docker environments
because they can create static binaries. Static binaries do not
require any further dependencies or a specific Linux distribution.
However, the applications must be compiled to Linux binaries. This
requires a Go environment that can create Linux binaries.

#### Docker Multi Stage Builds

The example in [chapter 9](#chapter-esi) uses Docker multi stage
builds. Such a build divides the build process of the Docker image into
several stages. The first stage can compile the program in a Docker
container with a Go build environment. The second stage can execute
the Go program in a Docker container as runtime environment that
contains only the compiled
program. Consequently, the runtime environment has no build tools and
is therefore much smaller.

Docker multi stage builds are not very complicated, as a look at the
`Dockerfile` shows.

{lang="Dockerfile"}
~~~~~~~~
FROM golang:1.8.3-jessie
COPY /src/github.com/ewolff/common /go/src/github.com/ewolff/common
WORKDIR /go/src/github.com/ewolff/common
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o common .

FROM scratch
COPY bootstrap-3.3.7-dist /css/bootstrap-3.3.7-dist
COPY --from=0 /go/src/github.com/ewolff/common/common /
ENTRYPOINT ["/common"]
CMD []
EXPOSE 8180
~~~~~~~~

The base image `golang` contains the Go installation (line 1). Into
this image the Go source code is copied (line 2) and compiled (line
3/4). With that, stage 0 of the build is finished.

Stage 1 creates a new Docker image. The image `scratch` (line 6) is an
empty Docker image. The `Dockerfile` copies the bootstrap library
(line 7) and the compiled Go binary from stage 0 (line 8) into this
image. The option `--from=0` indicates that the file `common`
originates from
stage 0 of the Docker build.

Finally, `ENTRYPOINT` defines the binary that is supposed to be
started (line 9). `CMD` (line 10) indicates that no options are to be
passed to the binary at the start. Normally, `ENTRYPOINT` would be a
shell which starts the process that is configured with `CMD`. However,
in the `scratch` image there is no shell. Therefore, `ENTRYPOINT`
replaces the shell by the Go binary `common`, and `CMD` defines that
this binary
is to be started without options. `EXPOSE` makes port 8180, on which
the process is listening, available to the outside.

#### Multi Stage Builds: Advantages

Multi stage builds have a number of advantages.

* Build and runtime are clearly *separated*. In the runtime
environment there are no build tools provided.

* *No Go environment* has to be installed on the local computer.  Otherwise, an
environment for cross compiling Go to Linux would have to be installed
on the local computer because the target system is a Linux Docker container.

* The image is very small. It is just 5.91 MB. 5.92 MB of this are the
Go binary and 984 kB the bootstrap library.

* The image does not contain a Linux distribution and therefore has a
minimal attack surface from a security point of view.

However, a multi
stage build creates a new Docker image for each build, which is only
needed for the build process and can be removed afterwards. At some
point, these images have to be deleted. But these images are only
stored on the build system or the developer computer so that all old
Docker images can simply be deleted at a defined point in time, for
example with `docker image prune`.

#### Go for Microservices?

The criteria from
[section 5.1](#section-technisch-mikro-anforderungen) for the
implementation of microservices can serve as a basis to assess Go's
suitability as a microservices programming language.

#### Communication

Go supports REST in the standard libraries. Libraries are also
available for messaging systems such as AMQP, for example
<https://github.com/streadway/amqp>. There is also a library for
messaging with
[Redis](https://github.com/go-redis/redis). Due to the widespread use
of Go, there is hardly any communication infrastructure that does not
support Go.

#### Operation

Go also offers many options for operation.

* The *deployment* in a Docker container is very easy with Docker
multi stage builds, as already illustrated.

* Libraries like [Viper](https://github.com/spf13/viper) support
the *configuration* of Go applications. This library supports formats
such as YAML or JSON.

* Go itself already offers support for logs. The Go microservices
framework Go Kit contains additional features for
  [logs](https://godoc.org/github.com/go-kit/kit/log) in more complex scenarios.

* For *metrics*, [Go Kit](https://godoc.org/github.com/go-kit/kit)
supports a plethora of tools such as Prometheus (see
[chapter 20](#chapter-monitoring)), but also Graphite or InfluxDB.

#### New Microservices

For a new microservice, it is enough to create the Docker build and
then write the source code. That is not very elaborate.

#### Resilience

Go Kit contains an implementation of resilience patterns such as
[Circuit Breaker](https://godoc.org/github.com/go-kit/kit/circuitbreaker).
In addition, there is a port of the
[Hystrix library](https://github.com/afex/hystrix-go) for Go.

## 5.5 Variations {#section-technisch-mikro-variationen}

The technical micro architecture decisions can be made differently for
each microservice. But there is a connection with the macro
architecture. The uniformity of the operational aspects can be
enforced by the macro architecture. If you want to implement a
microservice with other technologies in a Spring Boot microservices
architecture, this can lead to a lot of effort.

A macro architecture decision could be to read out configurations from
an `application.properties` file. This decision does not restrict the
choice of implementation technologies. But for a Spring Boot
application, the implementation is very simple because this mechanism
is built into Spring Boot and the default for Spring Boot
applications. A Go application, on the other hand, would have to be
adapted to this requirement.

This effect supports a uniform choice of technology for the
microservices because implementing a microservice with Spring Boot is
easier. Therefore, developers will prefer Spring Boot. A uniform
choice of technology has further advantages. For example, developers
are more likely to find their way around in other microservices, and
developers of different microservices can help each other out with
technology issues.

In order to really treat other technologies on an equal footing, a
different macro architecture decision should be made. Spring Boot
offers
[many more options](https://docs.spring.io/spring-boot/docs/1.5.6.RELEASE/reference/html/boot-features-external-config.html).
For example, the configuration can be stored in environment variables,
transferred via the command line or read from a configuration server.

#### Alternatives to Spring Boot

In the Java area there are some alternatives to Spring Boot.

* A classic Java EE application with an application server or a web
server is also conceivable as an implementation for a microservice.
However, in this case deployment is more complex because the
application server has to be installed additionally. Also
application servers and applications must be configured, in some cases
even with two different technologies. There are anyhow doubts about
the [usefulness of application servers](https://jaxenter.com/java-application-servers-dead-112186.html).

* [Wildfly Swarm](http://wildfly-swarm.io/) provides a simple JAR
deployment. However, instead of Spring APIs it implements the
standardized Java EE APIs and supplements them with technologies from
the microservices area such as Hystrix.

* [Dropwizard](http://www.dropwizard.io/) has long been offering the
possibility of developing Java REST services and deploying them as
JARs.

Of course, there are also a lot of other possible choices for
the programming
language apart from Java or Go.
It is impossible to even list them in this book. Actually the point
this book makes is that the technologies for the implementation of
each microservice are not that important. It is easily possible to
implement each microservice with a different programming language and
framework. So the decision can easily be changed. However, it is much
harder to change the technologies for communication, integration and
operations that this book focuses on.

The criteria from
[section 5.1](#section-technisch-mikro-anforderungen) are a yardstick
to check the technologies for their suitability for microservices, as
the [section 5.3](#section-technisch-mikro-spring-boot) does for
Spring Boot and [section 5.4](#section-technisch-mikro-go) for
Go. Such an assessment is recommended for each technology used. The
examples in [chapter 8](#chapter-links-javascript) are mostly
implemented with Node.js and JavaScript. This shows that microservices
can also 
be implemented with completely different technologies.

## 5.6 Conclusion {#section-technisch-mikro-fazit}

Individual microservices can differ greatly in their technical micro
architecture. Exactly this freedom is a major advantage of
microservices architectures.

* The macro architecture and the challenges associated with
implementing microservices can be used to derive requirements for
micro architecture and microservices technologies.

* Reactive programming can be used to implement microservices, but
this is not mandatory to meet the requirements.

* Spring Boot and Java meet the requirements, just like Go does with
the appropriate libraries.

* There are also many other alternatives.

Since each microservice can use a different micro architecture and
other technologies, the technical decisions at this level are not so
important. They can be revised in any microservice. The rest of the
book mainly discusses technologies that have an impact on the macro
architecture and thus the entire system since these technologies have much
wider implications.

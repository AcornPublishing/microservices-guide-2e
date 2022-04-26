# 14 Recipe: REST with the Netflix Stack {#chapter-netflix}

This chapter provides the following content:

* Overview of the Netflix microservices stack

* Details about service discovery with Eureka, routing with Zuul, load
balancing with Ribbon, and resilience with Hystrix

* The advantages and disadvantages of the Netflix stack

In this way, the reader can assess the suitability of these
technologies for a concrete project and a microservices
system with these technologies.

#### Where does the Netflix stack come from?

[Netflix](https://www.netflix.com/) has developed a new platform for
online video streaming to meet the high performance and scalability
requirements in this area. The result was one of the first
microservices architectures.

Later, Netflix released its technologies as open source projects.
Therefore, the Netflix stack is one of the first stacks to implement
microservices.

#### License & Technology

The components of the Netflix stack are open source and are under the
very liberal Apache license. The Netflix projects are practically all
based on Java. They are integrated into Spring Cloud, making it much
easier to use them together with Spring Boot (see
[section 5.3](#section-technisch-mikro-spring-boot)).

## 14.1 Example {#section-netflix-beispiel}

The example for this chapter can be found at
<https://github.com/ewolff/microservice>. It consists of three
microservices.

* The *catalog* microservice manages the information about the items.

* The *customer* microservice stores the data of the customers.

* The *order* microservice can accept new orders. It uses the catalog
and the customer microservice.

#### Architecture of the Example

{id="fig-netflix-architektur"}
![Fig. 14-1: Architecture of the Netflix Example](images/netflix-architektur.png)

Each of the microservices has its own web interface with which users
can interact. Among each other, the microservices communicate via
REST. The order microservice requires information about customers and
items from the other two microservices.

In addition to the microservices, there is also a Java application
that displays the Hystrix dashboard, with which the monitoring of the
Hystrix circuit breakers can be visualized.
[Figure 14.2](#fig-netflix-docker) shows the entire example at the
level of the Docker containers.

#### Building the Example

[Section 0.4](#section-einleitung-quick-start) explains which software
has to be installed for starting the example.

First the code has to be downloaded with `git
clone https://github.com/ewolff/microservice.git`. Then the code has
to be compiled with `./mvnw clean package` (macOS, Linux) or `mvnw.cmd
clean package` (Windows) in the directory `microservice-demo`.
See [appendix B](#anhang-maven) for more details on Maven and how to
trouble shoot the build.
Afterwards the Docker containers can be built with `docker-compose
build` in the directory `docker` and started with `docker-compose up
-d`.
See [appendix C](#anhang-docker) for more details on Docker,
docker-compose and how to troubleshoot them.
Subsequently, the Docker containers are available on the Docker
host.

<https://github.com/ewolff/microservice/blob/master/HOW-TO-RUN.md> in
detail explains the steps that need to be performed to build and run
the example.

#### Docker Containers and Ports

{id="fig-netflix-docker"}
![Fig. 14-2: Docker Containers in the Netflix Example](images/netflix-docker.png)

The Docker containers communicate via an internal network. Some Docker
containers can also be used via a port on the Docker host. The Docker
host is the computer on which the Docker containers run.

* The three microservices *order*, *customer*, and *catalog* each run
in their own Docker containers. Access to the Docker containers is
only possible within the Docker network.

* In order to be able to use the services from the outside, *Zuul*
provides routing. The Zuul container can be accessed from outside
under port 8080 and forwards requests to the microservices. If the
Docker containers are running locally, the URL is
<http://localhost:8080>. At this URL, there is also a web page
available which includes links to all microservices, Eureka, and the
Hystrix dashboard.

* *Eureka* serves as service discovery solution. The dashboard is
available at port 8761. This port is also accessible at the Docker
host. For a local Docker installation, the URL is
<http://localhost:8761>.

* Finally, the *Hystrix dashboard* runs in its own Docker container
that can also be accessed under port 8989 on the Docker host, for
example at <http://localhost:8989>.

## 14.2 Eureka: Service Discovery {#section-netflix-eureka}

Eureka implements service discovery. As already discussed in
[section 13.3](#section-synchron-herausforderungen), for synchronous
communication microservices have to find out at which port and IP
address other microservices can be accessed.

Essential characteristics of Eureka are:

* Eureka has a *REST interface*. Microservices can use this interface
to register or request information about other microservices.

* Eureka supports *replication*. The information from the Eureka
servers is distributed to other servers. This enables the system to
compensate for the failure of Eureka servers. In a distributed system,
service discovery is essential for communication between
microservices. Therefore, service discovery must be implemented in
such a way that a failure of one server does not cause the entire
service discovery to fail.

* Due to *caches on the client* the performance of Eureka is very
good. In addition, availability is improved because the information is
stored on the client, thus compensating for server failure. The server
only sends information about new or deleted microservices to the
client and not information about all registered services, making
communication very efficient.

* Netflix supports *AWS* (Amazon Web Services), i.e. the Amazon Cloud.
In AWS, servers run in availability zones. These are basically
separate data centers. The failure of an availability zone does not
affect other availability zones. Several availability zones form a
region. A region is located in a geographical zone. For example, the
data centers for the region called EU-West-1 is located in 
Ireland. Eureka can take regions and availability zones into account
and, for example, offer a microservice instance from the same
availability zone to a client as a result of the service discovery in
order to increase speed.

* Eureka expects the microservices to regularly send *heartbeats*. In
this way, Eureka detects crashed instances and excludes them from the
system. This increases the probability that Eureka will return service
instances that are available. However, it may happen that a
microservice instance is returned even though it is no longer running.
But whether a microservice instance is crashed is obvious once it is
used so that a
different instance can be used in that case.

#### Servers

The Netflix Eureka project is available for download at
[GitHub](https://github.com/Netflix/eureka/). So you can build the
project and 
get both the server and the client.

The Spring Cloud project also supports Eureka. The Eureka server can
even be started as a
Spring Boot application. For this, the main class which also has the
annotation `@SpringBootApplication` has additionally to be annotated
with `@EnableEurekaServer`. In `pom.xml` in the `dependencyManagement`
section the Spring Cloud dependencies have to be imported, and a
dependency on the library
`spring-cloud-starter-eureka-server` has to be inserted. In addition,
a configuration in the `application.properties` file is necessary. The
project `microservice-demo-eureka-server` provides all of that
and implements an Eureka server.

At first glance, it does not seem to make much sense to build the
Eureka server by yourself in this way, especially since the
implementation essentially consists of an annotation. But 
the Eureka server can be treated like all the other microservices. The
Spring Cloud Eureka server is a JAR file and, like all other
microservices, can be stored and started in a Docker image. It is also
possible to secure it like a Java web application with Spring
security, for example, and configure logging and monitoring as with
all other microservices.

{id="fig-netflix-eureka", width=75%}
![Fig. 14-3: Eureka Dashboard](images/netflix-eureka.png)

Eureka provides a dashboard (see [figure 14-3](#fig-netflix-eureka)).
It displays an overview of the microservices which are registered with
Eureka. This includes the names of the microservices and the URLs at
which they can be accessed. However, the URLs only works in the Docker
internal network so that the links in the dashboard do not work. The
dashboard is accessible on the Docker host at port 8761 i.e.
<http://localhost:8761/> if the example runs locally.

#### Client

Each microservice is a Eureka client and must register with the Eureka
server in order to inform the Eureka server about the name of the
microservice, and the IP-address and port at which it can be reached.
Spring Cloud simplifies the configuration for clients.

#### Registration

The client has to have a dependency on `spring-cloud-starter-eureka`
to add the necessary libraries. The main class which has
the annotation `@SpringBootApplication` additionally has to be
annotated with `@EnableEurekaClient`. An alternative is
`@EnableDiscoveryClient`. In contrast to `@EnableEurekaClient`, the
annotation `@EnableDiscoveryClient` is generic. Thus it also works
with Consul (see [chapter 15](#chapter-consul)).

{linenos=off}
~~~~~~~~
spring.application.name=catalog
eureka.client.serviceUrl.defaultZone=http://eureka:8761/eureka/
eureka.instance.leaseRenewalIntervalInSeconds=5
eureka.instance.metadataMap.instanceId=${spring.application.name}:${random.value}
eureka.instance.preferIpAddress=true
~~~~~~~~

In the file `application.properties` shown above appropriate settings must be entered for the application to register.

* `spring.application.name` contains the name under which the
application registers at the Eureka server.

* `eureka.client.serviceUrl.defaultZone` defines which Eureka server
is to be used.

* The setting `eureka.instance.leaseRenewalIntervalInSeconds` ensures
that the registration information is replicated every five
seconds and thus is replicated faster than the default setting. This
makes new microservice instances usable more quickly. In production,
this value should not be set so low so that there is not too much
network traffic.

* `eureka.instance.metadataMap.instanceId` provides each microservice
instance with a random ID, for instance, to be able to discriminate
between two instances for load balancing.

* Due to `eureka.instance.preferIpAddress` the services register with
their IP address and not with their host name. This avoids problems
that arise because host names cannot be resolved in the Docker
environment.

During registration, the name of the microservice is automatically
converted to uppercase letters. Thus `order` is turned into `ORDER`.

#### Other Programming Languages

For programming languages other than Java, a library must be used to
access Eureka. There are some libraries that implement Eureka clients
for certain programming languages. Of course, Eureka provides a REST
interface which can also be used.

#### Sidecars

To use the Netflix infrastructure with other programming languages, it
is also the possibility to use a *sidecar*. A sidecar is an
application written in Java that uses the Java libraries to talk to
the Netflix infrastructure. The application uses the sidecar to
communicate with the Netflix infrastructure. So the sidecar is just a
helper for the real application. That way the application can be
written in any programming language. Essentially the sidecar is the
interface to the Netflix infrastructure.

In this way e.g. Eureka can support other programming languages, but this
requires an additional process that consumes additional resources.

Netflix itself offers [Prana](https://github.com/Netflix/Prana/wiki)
as sidecar. Spring Cloud also provides an implementation of
such a
[sidecar](http://projects.spring.io/spring-cloud/spring-cloud.html#_polyglot_support_with_sidecar).

#### Access to other Services

In the example, Ribbon (see [section 14.4](#section-netflix-ribbon))
implements the access to other services in order to implement
load balancing across multiple instances. Thus, the Eureka API is only
used via Ribbon to find the information about the other microservices.

## 14.3 Router: Zuul {#section-netflix-zuul}

Zuul is the routing solution that is part of the Netflix stack. Zuul
is responsible for forwarding external calls to the correct
microservice.

#### Zuul vs. Reverse Proxy

The routing could also be provided by a Reverse Proxy. This is a web
server that is configured to forward incoming calls to other servers.

Zuul has a feature that a reverse proxy is lacking. It has dynamic
filters. Depending on the properties of the HTTP request or external
configurations, Zuul can forward certain calls to specific servers or
execute logic for logging, for example. A developer can write custom
code for the routing
decision. The code can even be dynamically
loaded as Groovy code at runtime. In this way, Zuul ensures maximum
flexibility.

Zuul filters can also be used to implement central functionalities
such as logging of all requests. A Zuul filter can implement the
login, send information about the current user with the HTTP requests,
and thereby implement authentication. Zuul can thus take over typical
functionalities of an API gateway (see
[section 13.3](#section-synchron-herausforderungen)).

#### Zuul in the Example

In the example, Zuul is configured as a proxy and does not contain any
special code. Zuul forwards access to a URL such as
<http://localhost:8080/order> to the microservice called ORDER. Such a
forwarding works for all microservices registered in Eureka. Zuul
reads the names of all microservices from Eureka and forwards the
requests.

{id="fig-netflix-zuul-konzept"}
![Fig. 14-4: Routing with the Zuul Proxy](images/netflix-zuul-konzept.png)

Of course, Zuul "reveals" which microservices the microservices system
is made up of. However, Zuul can be reconfigured by routes and filters
in such a way that completely different microservices can be accessed
under the same URL.

Zuul can also deliver static content. In the example, Zuul provides
the web page from which the various microservices can be accessed.

## 14.4 Load Balancing: Ribbon {#section-netflix-ribbon}

Microservices have the advantage that each microservice can be scaled
independently of the other microservices. To do this, it is necessary
that the call to a microservice can be distributed to several
instances by a load balancer.

#### Central Load Balancer

Typically, a single load balancer is used for all calls. Therefore, a
single load balancer, which processes all requests from all
microservices, can also be used for an entire microservices system.
However, such an approach leads to a bottleneck since all network
traffic must be routed through this single load balancer. The load
balancer is also a single point of failure. If the load balancer
fails, all network traffic stops functioning and the entire
microservices system fails.

{id="fig-netflix-zentraler-loadbalancer"}
![Fig. 14-5: Central Load Balancer](images/netflix-zentraler-loadbalancer.png)

Decentralized load balancing would be better. For this, each
microservice must have its own load balancer. If a load balancer
fails, only one microservice will fail.

#### Client-side Load Balancing

The idea of client-side load balancing can be implemented by a
"normal" load balancer such as Apache httpd or nginx. A load balancer
is deployed for each microservice. The load balancer must obtain the
information about the currently available microservices from the
service discovery.

{id="fig-netflix-client-seitiger-loadbalancer"}
![Fig. 14-6: Client-side Load Balancer](images/netflix-client-seitiger-loadbalancer.png)

It is also possible to write a library that distributes requests to
other microservices to different instances. This library must read the
currently available microservice instances from the service discovery
and then for each request select one of the instances. This is not
particularly hard to implement. This is the way
[Ribbon](https://github.com/Netflix/ribbon/wiki) works.

#### Ribbon API

Ribbon offers a relatively simple API for load balancing.

{linenos=off, lang="java"}
~~~~~~~~
private LoadBalancerClient loadBalancer;
  // Spring injects a LoadBalancerClient
ServiceInstance instance = loadBalancer.choose("CUSTOMER");
url = String.format("http://%s:%s/customer/",
 instance.getHost(), instance.getPort());
~~~~~~~~

Spring Cloud injects an implementation of the interface
`LoadBalancerClient`. First, a call to the `LoadBalancerClient`
selects an instance of a microservice. This information is then used
to fill a URL to which the request can be sent.

Ribbon supports various strategies for selecting an instance. Thus,
approaches other than a simple round robin are feasible.

#### Ribbon with Consul

As part of the Netflix stack, Ribbon supports Eureka as a service
discovery tool, but it also supports Consul. In the Consul example
(see [chapter 15](#chapter-consul)) the access to the microservices is
implemented identical to the Netflix example.

#### RestTemplate

Spring contains the `RestTemplate` to easily implement REST calls. If
a `RestTemplate` is created by Spring and annotated with
`@LoadBalanced`, Spring makes sure that a URL like `http://order/` is
forwarded to the order microservice. Internally, Ribbon is used for
this. <https://spring.io/guides/gs/client-side-load-balancing/> shows
how this approach can be implemented with a `RestTemplate`.

## 14.5 Resilience: Hystrix {#section-netflix-hystrix}

With synchronous communication between microservices, it is important
that the failure of one microservice does not cause other
microservices to fail as well. Otherwise, the unavailability of a single
microservice can cause further microservices to gradually fail until
the entire system is no longer available.

The microservices may, of course, return errors because they cannot
deliver reasonable results due to a failed microservice. However, it
must not happen that a microservice waits for the result of another
microservice for an infinite period of time and thereby becomes
unavailable itself.

#### Resilience Patterns

The book "Release It!"[^releaseit] describes different patterns with
which the resilience of a system can be increased.
Hystrix implements some of these patterns.

[^releaseit]: Michael T. Nygard: Release It!: Design and Deploy Production-Ready Software, Pragmatic Bookshelf, 2nd Edition, 2017, ISBN 978 1 68050 239 8

* A *timeout* prevents a microservice from waiting too
long for another microservice. Without a timeout, a thread can block
for a very long time because, for example, the thread does not get a
response from another microservice. If all threads are blocked, the
microservice will fail because there are no more threads available for
new tasks. Hystrix executes a request in a separate thread pool.
Hystrix controls these threads and can terminate the request to
implement the timeout.

* *Fail Fast* describes a similar pattern. It is better to
generate an error as quickly as possible. The code can check at the
beginning of an operation whether all necessary resources are
available. This may include enough disk space, for example. If
this is not the case, the request can be terminated immediately with
an error. This reduces the time that the caller has to block a thread
or other resources.

* Hystrix can use its own thread pool for each type of request. For
example, a separate thread pool can be set up for each called
microservice. If the call of a particular microservice takes too long,
only the thread pool for that particular microservice is emptied,
while the others still contain threads. This will limit the impact of
the problem and is called a *bulkhead*. This term was coined in
analogy to a watertight bulkhead in a ship which divides the ship into
different segments. If a leak occurs, only part of the ship is flooded
with water so that the ship does not sink.

* Finally, Hystrix implements a *circuit breaker*. This is a fuse
analogous to the ones used in the electrical system of a house. There,
a circuit breaker is used to cut off the current flow if there is a
short circuit. This prevents for example a fire from breaking out. The
Hystrix circuit breaker has a different approach. If a system call
results in an error, the circuit breaker is opened and does not allow
any further calls to pass through. After some time, a call is allowed
to pass through again. Only when this call is successful, the circuit
breaker is closed again. This prevents a faulty microservice from
being called. This saves resources and avoids blocked threads. In
addition, the circuit breakers of the different clients are closing
one by one so that a failed and recovered microservice only gradually
has to handle the full load. This reduces the probability that it will
fail again immediately after starting up.

#### Implementation

[Hystrix](https://github.com/Netflix/Hystrix/) offers an
implementation of most resilience patterns as a Java library.

The Hystrix API requires command objects instead of simple method
calls. These classes supplement the method call with the necessary
Hystrix functionalities. When using Hystrix with Spring Cloud, it is
not necessary to implement commands. Instead, the methods are
annotated with `@HystrixCommand`. It activates Hystrix for this
method. The attributes of the annotation configure Hystrix.

{linenos=off, lang="java"}
~~~~~~~~
@HystrixCommand(
  fallbackMethod = "getItemsCache",
  commandProperties = {
    @HystrixProperty(
      name = "circuitBreaker.requestVolumeThreshold",
      value = "2") })
public Collection<Item> findAll() {
...
  this.itemsCache = pagedResources.getContent();
...
  return itemsCache;
}
~~~~~~~~

The listing shows the access from the order microservice to the
catalog microservice. `circuitBreaker. requestVolumeThreshold`
specifies how many calls in a time window must cause errors for the
circuit breaker to open. In addition, the `fallbackMethod` attribute
of the annotation configures the method `getItemsCache()` as a
fallback method. `findAll()` stores the data returned by the catalog
microservice in the instance variable `itemsCache`. The
`getItemsCache()` method serves as a fallback and reads the result of
the last call from the instance variable `itemsCache` and returns it.

{linenos=off, lang="java"}
~~~~~~~~
private Collection<Item> getItemsCache() {
  return itemsCache;
}
~~~~~~~~

The reasoning behind this is that it is better that the service
continues to work with outdated data than that the service does not work
at all. This can lead to orders being charged at an outdated price.
However, this is probably the better option compared to accepting no
orders at all.

In
general if a service fails, a default value can also be used or an
error can be reported. Reporting an error is the right solution if
incorrect data cannot be accepted under any circumstances.
Which approach is correct is
in the end a decision that depends on the domain logic.
It should
only be avoided that in case of an error the REST call burdens the
server or blocks the client for too long.

#### Monitoring

The state of the circuit breakers provides a good overview of the
state of the system. An open circuit breaker is an indication of a
problem. So Hystrix is a good source of metrics. Hystrix provides
information about the circuit breaker state via HTTP as a stream of
JSON data.

#### Hystrix Dashboard

The Hystrix dashboard can display this data on a web page and thereby
shows what is happening in the system at the moment (see
[figure 14-7](#fig-netflix-hystrix)).

{id="fig-netflix-hystrix"}
![Fig. 14-7: Hystrix Dashboard](images/netflix-hystrix.png)

The upper area shows the state of the circuit breaker for the
functions `getOne()`, `findAll()`, and `price()`. The circuit breakers
of all three functions
are `closed`. So there are no errors at the moment. The dashboard also
shows information about the average latency of the requests and
current throughput.

Hystrix executes the calls in a separate thread pool. The state of
this thread pool is also shown on the dashboard. It contains ten
threads and is currently processing no requests.

#### Other Monitoring Options

The Hystrix metrics are also available via the Spring Boot mechanisms
(siehe [section 5.3](#section-technisch-mikro-spring-boot)) and can be
exported to other monitoring systems. Thereby, Hystrix metrics can be
seamlessly integrated into an existing monitoring infrastructure.

#### Turbine

The metrics of a single microservice instance are not particularly
meaningful. Microservices can be scaled independently. This means that
many instances can exist for each microservice. This means that the
Hystrix metrics of all instances must be displayed together.

This can be done with
[Turbine](https://github.com/Netflix/Turbine/wiki). This tool queries
the HTTP data streams of the Hystrix servers and consolidates them
into a single stream of data displayed by the dashboard. Spring Cloud
offers a simple way to implement a Turbine server, see for instance
<https://github.com/ewolff/microservice/tree/master/microservice-demo/microservice-demo-turbine-server>.

## 14.6 Recipe Variations {#netflix-variationen}

Netflix is only one technological option for implementing synchronous
microservices. There are various alternatives to the technologies of
the Netflix stack.

* The Zuul project is not maintained that much any more. An
  alternative might be [Zuul2](https://github.com/Netflix/zuul). It is
  based on [asynchronous
  I/O](https://medium.com/netflix-techblog/zuul-2-the-netflix-journey-to-asynchronous-non-blocking-systems-45947377fb5c)
  so it consumes less resources and is more stable. However, Spring
  Cloud [won't support
  Zuul2](https://github.com/spring-cloud/spring-cloud-netflix/issues/1498). So
  another alternative might be [Spring Cloud
  Gateway](https://spring.io/projects/spring-cloud-gateway). But the
  approach with Apache for routing shown in [chapter
  15](#chapter-consul) and Kubernetes in [chapter
  17](#chapter-kubernetes) is probably even better.

* Netflix does not invest in Hystrix that much any more. They suggest
  to use [resilience4j](https://github.com/resilience4j/resilience4j)
  instead which is a very similar Java library that also support
  typical resilience patterns.

* The *Consul example* ([chapter 15](#chapter-consul)) uses Consul
instead of Eureka for service discovery and Apache httpd instead of
Zuul for routing. However, this project also uses Hystrix for
resilience and Ribbon for load balancing. Consul also supports DNS and
can thus handle any programming language as implemented in the Consul
DNS example. Consul Template offers the possibility to configure
services with Consul by filling a configuration file template with the data
from Consul. In the example, Apache httpd is configured this way.
Eureka has quite a few advantages over Consul. Apache httpd as a web
server is familiar to many developers and might therefore be the
less risky compared to Zuul. On the other hand, Zuul provides dynamic
filters that Apache httpd does not support.

* *Kubernetes* (see [chapter 17](#chapter-kubernetes)) and a *PaaS*
such as *Cloud Foundry* (see [chapter 18](#chapter-paas)) offer
service discovery, routing, and load balancing. At the same time the
code remains independent of the infrastructure. Nevertheless, the
examples in those chapters uses Hystrix for resilience, too. These
solutions require the use
of a Kubernetes or PaaS environment. Thus, it is no longer possible to
just deploy some Docker containers on a Linux server.

* Functionalities such as load balancing and resilience can be
implemented with an HTTP proxy instead of Ribbon. This is a further
development of the sidecar concept. An example is
[Envoy](https://github.com/lyft/envoy). This proxy implements
some resilience patterns. Envoy is also part of Istio (see [section
23.3](#section-service-mesh-how) and is used as a sidecar in Istio
application. Istio is a service mesh that supports many technologies
for the operation of microservice system.
With a proxy, the application itself can be
kept free of these aspects. Apache httpd or nginx also can at least
implement load balancing.
So they could also provide basic features of a sidecar.

* Asynchronous communication (see [chapter 10](#chapter-asynchron))
seems at first sight to be a contradiction to communication via a
synchronous protocol like REST. But *Atom* (see
[chapter 12](#chapter-atom)) can be combined with concepts from this
chapter. Atom uses REST, so the microservices only need to implement
other types of REST resources. A combination with messaging systems
like *Kafka* (see [chapter 11](#chapter-kafka)) is also conceivable.
However, in this case, the system not only has the complexity of the
messaging system, but must also offer a REST environment.
  
* *Frontend integration* ([chapter 7](#chapter-frontend)) works on a
different level than REST and can be combined with the Netflix stack.
In particular, integration with *links and JavaScript*
([Chapter 8](#chapter-links-javascript)) is possible without any
problems. With *ESI* (see [chapter 9](#chapter-esi)) Varnish instead
of Zuul implements routing. So Varnish would have to extract the IP
addresses of the microservices from Eureka. However, this is not
possible without further ado.

## 14.7 Experiments {#section-netflix-experimente}

* Supplement the system with an additional microservice.
  * A microservice that is used by a call center agent to create
    notes for a call can be used as an example. The call center
    agent should be able to select the customer.
  * Of course, you can copy and modify one of the existing
    microservices.
  * Register the microservice in Eureka.
  * The customer microservice must be called via Ribbon. Then the
    microservice will be found automatically via Eureka. Otherwise, the
    microservice must be looked up explicitly in Eureka.
  * Package the microservice in a Docker image and add the image
    to `docker-compose.yml`. There you can also determine the name of
	the Docker container.
  * Create a link in `docker-compose.yml` from the container with the
  new service to the container `eureka`.
  That way the microservice can register at the Eureka server.
  * The microservice must be accessible from the homepage. To do this,
  you have to create a link similar to the other links in the file
  `index.html` in the Zuul
  project. Zuul automatically sets up the routing for the microservice
  as soon as the microservice is registered in Eureka.

* Try scaling and load balancing.
  * Increase the number of instances of a service, for instance with
  `docker-compose up --scale customer=2`.
  * Use the Eureka dashboard to determine whether two customer
  microservices are running. It is available at port 8761, for example
  at  <http://localhost:8761/> when Docker is running on the local
  computer. 
  * Observe the logs of the order microservice with `docker logs -f
  ms_order_1` and have a look  whether different instances of the
  customer microservice are called. This should be the case because
  Ribbon is used for load balancing. For this, you have to trigger
  requests to the order application e.g. a simple reload of the
  starting page.

* Simulate the failure of a microservice.
  * Watch the logs of the order microservice with `docker logs
    -f ms_order_1` and have a look how the catalog microservice is
	called. For this, you have to trigger requests to the order
	application. For example you can reload the starting page.
  * Find the IP address of the order microservice with the help of the
  Eureka dashboard at port 8761, for example <http://localhost:8761/>
  when Docker is running locally. 
  * Open the Hystrix dashboard at port 8989 on the Docker host,
    for example at <http://localhost:8989/> when Docker is running on
	the local computer. 
  * Using this IP address enter the URL of the Hystrix JSON data
  stream in the Hystrix dashboard. This can for instance be 
    <http://172.18.0.6:8080/actuator/hystrix.stream>. The Hystrix dashboard
	should show closed circuit breakers like in
	[figure 14-7](#fig-netflix-hystrix).
  * Shut down all catalog instances with `docker-compose up --scale 
    catalog=0`.
  * Watch the log of the order microservices during the next calls.
  * Also observe the Hystrix dashboard. The circuit breaker will only
  open when multiple calls have failed. 
  * When the circuit breaker is open, the order microservice should
  work again since now the fallback is activated.
  Then a cached value is used.

* Only the access to the catalog microservice is safeguarded with
Hystrix. In the order microservice the class `CustomerClient`in package
`com.ewolff.microservice.order.clients` implements the access to the
customer microservice.
Extend the access to the customer microservice using Hystrix.
For this, use the class `CatalogClient`
from the same package as example.

* Extend the Zuul setup by a fixed route. In `application.yml` in
directory `src/main/resource` in project
`microservice-demo-zuul-server` add for example the following to make
the INNOQ homepage appear at <http://localhost:8080/innoq>:


{linenos=off, lang="yaml"}
~~~~~~~~
zuul:
  routes:
    innoq:
      path: /innoq/**
      url: http://innoq.com/
~~~~~~~~
	  
* Add a filter to the Zuul configuration. A tutorial dealing with
Zuul filters can be found at
<https://spring.io/guides/gs/routing-and-filtering/>.

* Create your own microservice that, for example, only returns simple
HTML. Integrate it into Eureka and deploy it as part of the Docker
compose environment. If it is registered in Eureka, it can be
addressed immediately from the Zuul proxy at a URL like
<http://localhost:8080/mymicroservice> .

## 14.8 Conclusion {#section-netflix-fazit}

The Netflix stack provides a variety of projects to build microservice
architectures. The stack solves the typical challenges of synchronous
microservices as follows:

* *Service discovery* is offered by Eureka. Eureka focuses on Java
with the Java client, but also offers a REST API and libraries for
other languages. Eureka can therefore also be used with other
languages.

* For *resilience* Hystrix is the de facto standard for Java
and covers this area very well. Non-Java applications can use Hystrix
via a sidecar at most.
There are ports of Hystrix for other languages like e.g. Go (see
[section 5.4](#section-technisch-mikro-go)).
Hystrix is independent from the other technologies and can therefore
be used on its own.

* Ribbon implements client-side *load balancing*, which has many
advantages. Since Ribbon is a Java library, other technologies are
difficult to use with Ribbon. Especially in the area of load
balancing, there are numerous classic
load balancers that provide alternatives.
Ribbon relies on Eureka for service discovery but can also use Consul.

* *Routing* is solved by Zuul. Zuul's dynamic filters are very
flexible, but many developers are rather familiar with reverse proxies
based on web servers like Apache httpd or nginx. In this case, a
reverse proxy might be the safer option. Additional features such as SSL
termination, request throttling, or similar things may also be
required, which Zuul does not offer. 
[Section 15.3](#section-consul-apache) shows how Apache httpd can be
used with Consul to provide routing.
Zuul requires Eureka to find the microservices.

The servers in the Netflix stack are written in Java so that the
servers from the Netflix stack and the microservices can be packed
into JAR files. Thanks to Spring Cloud, they are also uniformly
configurable. Also the handling of metrics and logs is identical.
This uniformity can be an advantage for operation.

#### Advantages

* Eureka together with client-side caching is very fast and resilient.

* Zuul is very flexible because of its filters.

* Client-side load balancing avoids single points of failure or bottlenecks.

* Hystrix is very mature and the de facto standard for Java.

#### Challenges

* The Netflix stack implements many solved problems (e.g. reverse
  proxy, service discovery) anew.

* The technologies focus on Java and therefore limits technology freedom.

* The code depends on the Netflix stack (Ribbon, Hystrix, but also
  Eureka due to `@EnableDiscoveryClient` and the client API).

# 15 Recipe: REST with Consul and Apache httpd {#chapter-consul}

This chapter shows an implementation for a synchronous microservices
system with Consul and the Apache httpd server.

Essential contents of the chapter are:

* Consul is a very powerful service discovery technology.

* Apache httpd can be used as a load balancer and router for HTTP
  requests in a microservices system.

* Consul Template can create a configuration file for the Apache httpd server
  that includes information about all registered microservices. Consul
  Template configures and restarts
  Apache httpd when new microservice instances are started.

#### Where does Consul come from?

Consul is a product of the company
[Hashicorp](https://www.hashicorp.com/) which offers various products
in the field of microservices and infrastructure. Of course, Hashicorp
offers also commercial support for Consul.

#### License and Technology

[Consul](https://www.consul.io/) is an open source product. It is
written in Go and is licensed under the
[Mozilla Public License 2.0](https://github.com/hashicorp/consul/blob/master/LICENSE). The
code is available at [GitHub](https://github.com/hashicorp/consul).

## 15.1 Example {#section-consul-beispiel}

The domain structure is identical to the example in the Netflix
chapter ([chapter 14](#chapter-netflix)) (see
[figure 15-1](#fig-consul-architektur)) and consists of three
microservices.

{id="fig-consul-architektur"}
![Fig. 15-1: Architecture of the Consul Example](images/netflix-architektur.png)

* The *catalog* microservice manages the information such a price or
  name for
  the items that can be ordered.

* The *customer* microservice stores customer data.

* The *order* microservice can accept new orders. It uses the catalog
  and customer microservice.

#### Architecture of the Example

The example in this chapter uses [Consul](https://www.consul.io/) for
service discovery and [Apache httpd server](https://httpd.apache.org/)
for routing the HTTP requests.

An overview of the Docker containers is shown in
[figure 15-2](#fig-consul-beispiel). The three microservices provide
their UI and REST interfaces at the port 8080. They are only
accessible within the network between the Docker containers. Consul
offers port 8500 for the REST interface and the HTML UI as well as UDP
port 8600 for DNS requests. These two ports are also bound to the
Docker host. So these ports are also accessible at the Docker host
and thus from other computers. The Docker host also provides the
Apache httpd at port 8080. Apache
httpd forwards calls to the microservices in the Docker network so
that the microservices can also be accessed from outside.

{id="fig-consul-beispiel"}
![Fig. 15-2: Overview of the Consul Example](images/consul-beispiel.png)

#### Building the Example

[Section 0.4](#section-einleitung-quick-start) describes which
software has to be installed for starting the example.

First download the code with `git
clone https://github.com/ewolff/microservice-consul.git`. Then the
code has to be translated with `./mvnw clean package` (macOS, Linux)
or `mvnw.cmd clean package` (Windows) in directory
`microservice-consul-demo`.
See [appendix B](#anhang-maven) for more details on Maven and how to
trouble shoot the build.
Afterwards the Docker containers can be
built in the directory `docker` with `docker-compose build` and started
with `docker-compose up -d`.
See [appendix C](#anhang-docker) for more details on Docker,
docker-compose and how to troubleshoot them.
Subsequently, the Docker containers are
available on the Docker host.

When the Docker containers are running on the local computer, the
following URLs are available:

* <http://localhost:8500> is the link to the Consul dashboard.

* <http://localhost:8080> is the URL of the Apache httpd server. It
  can display the
  web UI of all microservices.

<https://github.com/ewolff/microservice-consul/blob/master/HOW-TO-RUN.md>
describes the necessary steps for building and running the example in
detail.

## 15.2 Service Discovery: Consul {#section-consul-consul}

[Consul](http://www.consul.io) is a service discovery technology. It
ensures that microservices can communicate with each other. Consul has
some features that set it apart from other service discovery
solutions.

* Consul has an *HTTP REST API* and in addition supports
  *DNS*. [DNS](http://www.zytrax.com/books/dns/) (Domain Name System)
  is the system that maps host names such as www.innoq.com to IP
  addresses in the Internet. In addition to returning IP addresses, it
  can also return ports at which a service is available.
  This is a feature of the SRV DNS records.

* With [Consul Template](https://github.com/hashicorp/consul-template)
  Consul can generate configuration files. The files may contain IP
  addresses and ports of services registered in Consul. *Consul
  Template* also provides Consul's service discovery service to
  systems which cannot access Consul via the API. The systems only
  have to use some kind of configuration file, which they often
  already do anyway.

* Consul can perform *health checks* and exclude services from service
  discovery when the health check fails. For example, a health check
  can be a request to a specific HTTP resource to determine whether
  the service can still actually process requests. A service may still
  be able
  to accept HTTP requests, but it may not be able to process them
  properly due to a database failure. The service can signal this
  through the health check.

* Consul supports *replication* and can thus ensure high
  availability. If a Consul server fails, other servers with
  replicated data take over and compensate for the failed server.

* Consul also supports *multiple data centers*. Data can be replicated
  between data centers to further increase availability and protect
  Consul against the failure of a data center.  The search for
  services can be limited to the same data center. Services in the
  same
  data center usually deliver higher performance.

* Finally, Consul can be used not only for service discovery, but also
  for the *configuration* of services.  Configuration has different
  requirements.
  Availability is important for service discovery. Faulty information
  can be tolerated. If the service is accessed and is not available,
  it does not matter much. You can simply use another instance of the
  service. However, if services are configured incorrectly, this can
  lead to errors. So correct information is much more important for
  configuration than for service discovery.

#### Consul Dashboard

Access to the information about registered microservices is provided
by Consul in its dashboard (see
[figure 15-3](#fig-consul-dashboard)). 
it shows all service that have registered at the Consul server and
also the result of their health checks. In the example all
microservices are healthy and can accept requests.
In addition to the registered
services, it displays the servers on which Consul is running and the
contents of the configuration database. Access to the dashboard is
possible at port 8500 on the Docker host.

{id="fig-consul-dashboard"}
![Fig. 15-3: Consul Dashboard](images/consul-dashboard.png)

#### Reading Data with DNS

It is also possible to access the data from the Consul server via
DNS. This can be done, for example, with the tool `dig`.  `dig
@localhost -p 8600 order.service.consul.` returns the IP address of
the order microservice if the Docker containers run on the local
computer `localhost`.  Communication to the Consul DNS server takes
place via the UDP
port 8600.  `dig @localhost -p 8600 order.service.consul. SRV` returns
in addition to the IP address also the port at which the service is
available. DNS SRV records are used for this purpose. They are part of
the DNS standard and allow to specify a port for services in addition
to the IP address.

#### Consul Docker Image

The example uses a Consul Docker image directly from the manufacturer
Hashicorp.  It is configured so that Consul only stores the data in
the main memory and runs on a single node. This simplifies the setup
of the system and reduces the resource consumption.  This
configuration is of course unsuitable for a production environment
because data can be lost and a failure of the Consul Docker container
would bring the entire system to a standstill. In production, a
cluster of Consul servers should be used and the data should be stored
persistently.

## 15.3 Routing: Apache httpd {#section-consul-apache}

The [Apache httpd server](https://httpd.apache.org/) is one of the
most widely used web servers. There are modules that adapt the server
to different usage scenarios.  In the example, modules are configured
that turn Apache httpd into a reverse proxy.

#### Reverse Proxy

While a conventional proxy can be used to process traffic from a
network to the outside,
a reverse proxy is a solution for inbound network connections. It
can forward external requests to specific services. This means that
the entire
microservices system can be accessible under one URL, but can use
different microservices internally.

The concept of a reverse proxy has already been explained in [chapter
14.3](#section-netflix-zuul).

#### Load Balancer

In addition, Apache httpd serves as a load balancer. httpd distributes
network traffic to multiple instances to make the application
scalable. In the example, there is only one Apache httpd, which
functions simultaneously as reverse proxy and load balancer for
requests from the outside. The
requests the microservices send to each other are not handled by this
load balancer. For the communication between the microservices, the
library Ribbon is used, as already in the Netflix example (see
[section 14.4](#section-netflix-ribbon)).

One of the strengths of this solution is that it uses a well-proven
software, which teams often have already gained experience with. As
microservices place high demands on the operation and infrastructure,
such a conservative choice is advantageous to avoid the learning and
effort of another technology. Instead of Apache httpd you can, of
course, also use for example [nginx](https://nginx.org/).

There are also approaches like [Fabio](https://github.com/eBay/fabio)
which are written specifically for the load balancing of microservices
and are easier to use and configure.

## 15.4 Consul Template {#section-consul-template}

For each microservice Apache httpd must have an entry in its
configuration file. For this
[Consul Template](https://github.com/hashicorp/consul-template) can be
used. In the example, the `00-default.ctmpl` file is used as a
template to create the Apache httpd configuration. It is written in
the [Consul Templating Language](https://github.com/hashicorp/consul-template#templating-language).
For each microservice, it creates an entry that distributes the load
between the instances and redirects external requests to these
instances.

#### The Template

The essential part is:

{linenos=off}
~~~~~~~~
{{range services}}

<Proxy balancer://{{.Name}}>
{{range service .Name}}  BalancerMember http://{{.Address}}:{{.Port}}
{{end}}
</Proxy>
  ProxyPass        /{{.Name}} balancer://{{.Name}}
  ProxyPassReverse /{{.Name}} balancer://{{.Name}}

{{end}}
~~~~~~~~

The Consul Template API functions are enclosed in `{{` and `}}`. For
each service a configuration is generated with `{{range service}}`. It
contains a reverse proxy that is configured with the `<Proxy>`
element and the `Name` of the microservice. This element contains the
microservice instances as `BalancerMember`
with the `Address` and `Port` of each instance
for distributing the load between the microservice instances. The end
is formed by
`ProxyPass` and `ProxyPassReverse` which likewise belong to the
reverse proxy.

#### Starting the Consul Template

The Consul Template is started with the following section from the
`Dockerfile` in the directory `docker/apache`:

{linenos=off, lang="dockerfile"}
~~~~~~~~
CMD /usr/bin/consul-template -log-level info -consul consul:8500 \\
  -template "/etc/apache2/sites-enabled/000-default.ctmpl:/etc/apache2/sites-enabled/000-default.conf:apache2ctl -k graceful"
~~~~~~~~
  
Consul Template executes the command `apache2ctl -k graceful` if there
are new services or services have been removed and the configuration
has therefore been changed. This causes Apache
httpd to read the updated configuration and restart. However, open
connections are not closed, but remain open, until communication is
terminated. If no Apache httpd is running yet, one will be
started. Thus, Consul Template takes control of Apache httpd and
ensures that one instance of Apache httpd is always running in the
Docker container.

To do this, Consul Template must run in the same Docker container as
Apache httpd. This contradicts the Docker philosophy that only one
process should run in one container. However, this cannot be avoided
in the concrete example because these two processes are so closely
related.

#### Conclusion

Consul Template can ensure that a microservice can be
reached from outside as soon as it has registered with Consul. To do
this, the Apache httpd server does not need to know anything about the
service discovery or Consul. It receives the information in its
configuration and restarts.

## 15.5 Consul and Spring Boot {#section-consul-spring-boot}

The Consul integration in Spring Boot is comparable to the integration
of Eureka (see [section 14.2](#section-netflix-eureka)). There is a
configuration file `application.properties`. Here is the relevant
section:

{linenos=off, lang="dockerfile"}
~~~~~~~~
spring.application.name=catalog
spring.cloud.consul.host=consul
spring.cloud.consul.port=8500
spring.cloud.consul.discovery.preferIpAddress=true
spring.cloud.consul.discovery.instanceId=\${spring.application.name}:
\${spring.application.instance_id:\${random.value}}
~~~~~~~~

The section configures the following values:

* `spring.application.name` defines the name under which the
  application is registered in Consul.

* `spring.cloud.consul.host` and `spring.cloud.consul.port` determine
  at which port and on which host Consul can be accessed.

* Via `spring.cloud.consul.discovery.preferIpAddress` the services
  register with their IP address and not with the host name. This
  circumvents problems that arise because host names cannot be
  resolved in the Docker environment.

* `spring.cloud.consul.discovery.instanceId` assigns an unambiguous ID
  to each microservice instance, for example, for discriminating
  between instances for load balancing.

#### Code Dependencies

In addition, a dependency to `spring-cloud-starter-consul-discovery`
has to be inserted in `pom.xml` for the build. Moreover, the main
class of the application which has the annotation
`@SpringBootApplication` has also to be annotated with
`@EnableDiscoveryClient`.

#### Health Check with Spring Boot Actuator

Finally, the microservice must provide a health check.  Spring Boot
contains the Actuator module for this, which offers a health check as
well as metrics. The health check is available at the URL
`/health`. This is exactly the URL Consul requests. So it is
enough to add a dependency to `spring-boot-starter-actuator` in
the `pom. xml`. Specific health checks may need to be developed if the
application depends on additional resources.

#### Consul and Ribbon

Of course, a microservice must also use Consul to communicate with
other microservices. For this, the Consul example uses the Ribbon
library analogous to the Netflix example (see
[section 14.4](#section-netflix-ribbon)). Ribbon has been modified in
the Spring Cloud project so that it can also deal with Consul. Because
the microservices use Ribbon, the rest of the code is unchanged
compared to the Netflix example.

## 15.6 DNS and Registrator {#section-consul-dns}

The microservices have code dependencies to the Consul API for
registering.  This is not necessary.
[Registrator](https://github.com/gliderlabs/registrator) can register
Docker containers with Consul without the need for code.  When the
Docker containers are configured in such a way that they use Consul as
DNS server, the lookup of other microservices can also occur without
code dependencies.
This eliminates any dependencies on Consul in the project.

#### Structure of the Example

{id="fig-consul-dns"}
![Fig. 15-4: Consul with DNS](images/consul-dns.png)

[Figure 15-4](#fig-consul-dns) shows an overview of the approach.

* *Registrator* runs in a Docker container. Via a socket, Registrator
  collects information from the Docker daemon about all newly launched
  Docker containers. The Docker daemon runs on the Docker host and
  manages all Docker containers.

* The *DNS interface of Consul* is bound to the UDP port 53 of the
  Docker host. This is the default port for DNS.

* The *Docker containers* use the Docker host as DNS server.

The `dns` setting in the `docker-compose.yml` is configured to use the
IP address in the 
environment variable `CONSUL_HOST` as the IP address of the DNS
server. Therefore, the IP
address of the Docker host has to be assigned to `CONSUL_HOST` before
starting `docker-compose`. Unfortunately, it is not possible to
configure DNS access from the Docker containers in a way that does not
use this environment variable.

Registrator registers every Docker container started with
Consul. Thus, not just the microservices but also the Apache httpd
server or Consul itself can be found
among the services in Consul.

Consul registers the Docker containers with `.service.consul` added to
the name.  In `docker-compose.yml`, `dns_search` is set to
`.service.consul` so that this domain is always searched. In the end,
the order microservice uses the URLs `http://msconsuldns_customer:8080/`
and `http://msconsuldns_catalog:8080/` to access the customer and
catalog microservices. Docker compose creates the prefix `msconsuldns`
for the name of the Docker containers to separate the project from
other projects. This name is also used by Consul Template to configure
the Apache httpd for routing.

In this setup, Consul is responsible for load balancing. If there are
several instances of a microservice, Registrator registers them
all under the same name. Consul then returns one of the instances for
each DNS request.

The executable example can be found at
<https://github.com/ewolff/microservice-consul-dns> and the
instructions for starting it at
<https://github.com/ewolff/microservice-consul-dns/blob/master/HOW-TO-RUN.md>.

As a result of this approach, the microservices no longer contain any
code dependencies on Consul. Therefore, with these technologies, it is
no problem to implement microservices with a programming language
other than Java.

#### Configuration is also Possible in a Transparent Manner

[Envconsul](https://github.com/hashicorp/envconsul) also enables
configuration data to be read from Consul and made available to the
applications as environment variables. In this way, Consul can also
configure the microservices without that they have to include
Consul-specific code.

## 15.7 Recipe Variations {#section-consul-variationen}

Consul is very flexible and can be used in many different ways.

#### Combination with Frontend Integration

Like other approaches Consul can be combined with frontend integration
(see [chapter 7](#chapter-frontend)). SSI (Server-side Includes) with
Apache httpd is especially simple to combine since an Apache
httpd is already present in the system.

#### Combination with Asynchronous Communication

Synchronous communication can be combined with asynchronous
communication (see [chapter 10](#chapter-asynchron)). However, one
type of communication should suffice normally.  Atom or other
asynchronous approaches via HTTP (see [section 12](#chapter-atom)) are
easy to integrate into an HTTP-based system like Consul.

#### Other Load Balancers

Instead of Apache httpd, a server like nginx or a load balancer like
HAProxy can, of course, also be used for routing of requests from the
outside. Ribbon can also be 
replaced by such a load balancer so that also the internal load
balancing is
then configured with Consul Template. In this case only one type of
load balancer is used for load balancing and routing. Unlike the
Java library Ribbon, Apache httpd or nginx can be used with any
programming language. Each microservice then has its own httpd
or nginx instance so that no bottleneck or single point of failure is
created.

#### Service Meshes

[Chapter 23](#chapter-service-mesh) discusses services meshes. They
provide a lot of useful features e.g. for resilience, monitoring,
tracing and logging. Istio is an example of a service mesh. A service
mesh injects proxies into the communication between the
microservices. Istio supports Consul to achieve that. So with Istio,
Consul can be extended to become a complete platform for the operation
of microservices.

## 15.8 Experiments {#section-consul-experimente}

* Supplement the Consul system without DNS with an additional microservice.
  * A microservice that is used by a call center agent to create
    notes for a call can be used as an example. The call center
    agent should be able to select the customer.
  * Of course, you can copy and modify one of the existing
    microservices.
  * Register the microservice in Consul.
  * Ribbon has to be used to call the customer microservice. Ribbon
    does  the lookup of the 
    microservice in Consul. Otherwise, the
    microservice must be searched explicitly in Consul.
  * Package the microservice in a Docker image and reference the
    image in `docker-compose.yml`. There you can also specify the name
    of the Docker container.
  * Create in `docker-compose.yml` a link from the container with the
    new service to the container `consul`.
  * The microservice must be accessible from the homepage. To do this
    you have to create a link in the file `index.html` in the Docker
    container `apache`. Consul Template automatically sets up the
    routing for the microservice in Apache as soon as the microservice
    is registered in Consul.

* Supplement the DNS Consul system (see
  [section 15.6](#section-consul-dns)) with an additional
  microservice.
  * A microservice that is used by a call center agent to create
    notes for a call can be used as an example. The call center
    agent should be able to select the customer.
  * The call to the customer microservice has to use the host name
    `msconsuldns_customer`.
  * Of course, you can copy and modify one of the existing microservices.
  * A registration in Consul is not necessary since Registrator
    automatically registers each Docker container.
  * Package the microservice in a Docker image and reference the
    image in `docker-compose.yml`. There you can also specify the name
    of the Docker container.
  * Configure the DNS server for the new microservice in
    `docker-compose.yml` similar to the other microservices.
  * The microservice must be accessible from the homepage. To do this
    you have to create a link in the file `index.html` in the Docker
    container `apache`. Consul Template automatically sets up the
    routing for the microservice in Apache as soon as the microservice
    is registered in Consul.

* Currently, the Consul installation is not a cluster and is therefore
unsuitable for a production environment. Change the Consul
installation so that Consul runs in the cluster and the data from the
service registry is saved to a hard disk. To do this, the
configuration has to be changed in the `Dockerfile` and several
instances of Consul have to be started. For more information, see
<https://www.consul.io/docs/guides/bootstrapping.html>.

* Consul can also be used for saving the configuration of a Spring Boot
  application, see
  <https://cloud.spring.io/spring-cloud-consul/#spring-cloud-consul-config>. Use
  Consul to configure the example application with Consul.

* Replace Apache httpd with nginx, another web server, or e.g.
 HAProxy. To do this, you need to create an appropriate Docker
 image or search for a matching Docker image in the
 [Docker hub](https://hub.docker.com). In addition, the web server
 must be provided with reverse proxy extensions and configured with
 Consul Template.  Additional documentation about Consul Template can
 be found on the
 [Github web page](https://github.com/hashicorp/consul-template).
 For many systems there are also
 [Consul Template examples](https://github.com/hashicorp/consul-template/tree/master/examples).

* Try scaling and load balancing.
  * Increase the number of instances of a service, for example with
  `docker-compose up --scale customer=2`.
  * Use the Consul dashboard to check if two customer microservices
  are running. It is available under port 8500, for example at
  <http://localhost:8500/> when Docker is running on the local
  computer.
  * Observe the logs of the order microservice with `docker logs
    -f msconsul_order_1` and see if different instances of the customer
    microservice are called. This should be the case because Ribbon is
    used for load balancing. To do this, you must trigger requests to
    the order application e.g. by reloading the homepage.

* Add your own health check for one of the microservices. Check
  whether load balancing actually excludes a service when the health
  check is no longer successful.

## 15.9 Conclusion {#section-consul-fazit}

Setting up a microservices system with Consul is another option
for a synchronous system.  This infrastructure meets the typical
challenges of synchronous microservices as follows.

* *Service discovery* is covered by Consul.  Consul is very
  flexible. Due to the DNS interface and Consul Template it can be
  used with many technologies. This is particularly important in the
  context of microservices. While a system might not need to use a variety of
  technologies from the start, in the long term it is advantageous to
  be able to integrate new technologies.
  
* Consul is *more transparent* to use than Eureka. The Spring
  Cloud applications still need special Consul configurations. But Consul
  offers a configuration in Apache format for Apache httpd so at least
  in this case Consul is transparent.

* If Registrator is used to register the microservices and Consul is
used as DNS server, Consul is *fully transparent* and can be used
without any code dependencies.  With Envconsul, Consul can even configure
the microservices without code dependencies.

* Consul can be used to *configure* the microservices. In this way,
  with only one technological approach, both service discovery and
  configuration can be implemented.

* *Resilience* is not implemented in this example.

* *Routing* with Apache httpd is a relatively common approach. This
  reduces the technological complexity, which is quite high in a
  microservices system anyway. With the large number of new
  technologies and a new architectural approach, it is certainly
  helpful to cover at least some areas with established approaches.

* *Load balancing* is implemented with Ribbon like in the Netflix
  example (see [section 14.4](#section-netflix-ribbon)). However, it is
  no problem to provide each microservice instance with an Apache
  httpd, for example, which is configured by Consul Template in such a
  manner that it provides load balancing for outbound calls. For Consul
  DNS Consul even implements load balancing transparently
  with the DNS server.
  So in that case no additional technology for load balancing ist
  needed.

#### Comparision to Netflix

The technology stack from this example has the advantage that it also
supports heterogeneous microservices systems because there are no more
code dependencies on Consul if using DNS. Consul as
a service discovery technology is much more powerful than Eureka with
the DNS interface it provides and Consul Template.
Apache is a standard reverse proxy that is widely used. It is
therefore probably more mature than Zuul. Also Zuul is not really
supported any more while Apache is still one of the most broadly used
web servers. For resilience the stack does not really offer a good
solution. However it can still be combined with libraries such as
Hystrix.

The main benefit of the Consul technology stack is its independence
from a concrete language and environment. The Netflix stack is based
on Java and it is hard to integrate other languages. Also Netflix has
discontinued several of the projects i.e. Hystrix and Zuul. So there
are quite a few disadvantages of the Netflix stack but not really a
lot of advantages. The Consul stacks is therefore usually preferable
over the Netflix stack.

#### Advantages

* Consul does not have a Java focus but supports many different
  technologies.

* Consul also supports DNS.

* Consul Template can even configure many services (Apache httpd)
transparently via configuration files.

* Entirely transparent registration and service discovery with
Registrator and DNS are possible.

* The use of well established technologies such as Apache httpd
  reduces the risk.

#### Challenges

* Consul is written in Go. Therefore, monitoring and deployment differ
  from Java microservices.

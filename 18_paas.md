# 18 Recipe: PaaS with Cloud Foundry {#chapter-paas}

This chapter introduces PaaS (Platform as a Service) as a runtime
environment for microservices.

The text answers the following questions:

* What is a PaaS and how does it differ from other runtime
environments?

* Why is a PaaS suited for microservices?

As concrete example for a PaaS the chapter introduces the use of Cloud
Foundry.

## 18.1 PaaS: Definition {#section-paas-definition}

In the cloud there are fundamentally different services offered.

#### IaaS

An IaaS (Infrastructure as a Service) offers virtual computers on
which software has to be installed. Thus, IaaS is a simple solution
which essentially corresponds to classical virtualization. The
decisive difference is the billing model, which for IaaS bills only
the actually used resource per hour or per minute.

#### SaaS

SaaS (Software as a Service) denotes a cloud offer where software can
be rented e.g. for word processing or financial accounting. For
software development, for example, version controls or continuous
integration servers can be purchased as SaaS.

#### PaaS

PaaS stands for Platform as a Service. PaaS offers a platform on
which custom software can be installed and run. The developer only
provides the application to the PaaS. The PaaS makes the application
executable. Unlike Docker (see [chapter 5](#chapter-docker)) and
Kubernetes (see [chapter 17](#chapter-kubernetes)), the operating
system and the software installed on it is not under the developer's
control. For the microservices examples, the `Dockerfile` specifies
that an Alpine Linux distribution and a specific version of the Java
Virtual Machine (JVM) should be used. This is no longer necessary with
a PaaS. The JAR file contains the executable Java application and thus
everything that the PaaS needs.

To start the application in the runtime environment, the PaaS can
create a Docker container. But the decision which JVM and Linux
distribution to use is up to the PaaS. The PaaS must be prepared to
run different types of applications. .NET applications and Java
applications each require their own virtual machine, while Go
applications do not. The appropriate environment must be created by
the PaaS.
Nowadays PaaS are flexible enough to support different environment and
even roll your own environment. Therefore you can usually define which
JDK should be used.

#### PaaS Restricts Flexibility and Control

Developers have less control over the Docker images. However, the
question is whether a developer should spend time with the
selection of the JVM and the Linux distribution in the first place. Often, these issues
lie with operations anyway. Some PaaS offer the possibility to
configure the Linux distribution or JVM. Often, even complete runtime
environments can be defined by the user. Nevertheless, flexibility is
limited.

To run existing applications, the PaaS might not be flexible enough.
For example, an application might need a specific JVM version that the
PaaS does not support.
Microservices, however, are usually newly developed, so
this disadvantage does not play a great role.

#### Routing and Scaling

The PaaS must forward requests from the user to the application. So
routing is a feature of a PaaS. Similarly, PaaS can usually scale
applications individually, ensuring scalability.

#### Additional Services

Many PaaS can provide the application with additional services such as
databases.
Also typical features for operation such as support for
analyzing log data or monitoring are often part of a PaaS.

#### Public Cloud

PaaS are offered in the public cloud. The developers only have to
deploy their application into the PaaS and then the application runs
on the Internet. This is a very simple way to provide Internet
applications. In the public cloud, there are further advantages. If
the application is under high load, it can scale automatically.
Scalability is virtually unlimited as the application's public cloud
environment can provide lots of resources.

#### PaaS in Your Own Data Center

The situation is somewhat different when the applications run in your
own data center. The use of PaaS is still very easy for the developers. But
the PaaS must be installed. This can be a very complicated process,
which outweighs the advantages somewhat. However, the PaaS only needs
to be installed once. After that, developers can use the PaaS to
install a variety of applications and bring them into production.
Operations only needs to ensure that the PaaS functions reliably.

Particularly in the case of operations departments that still have
many manual processes and where the provision of resources takes a
long time, PaaS can considerably accelerate the rollout of
applications without the need for major changes to the organization or
processes.

#### Macro Architecture

As already shown in [chapter 16](#chapter-plattformen),
microservices platforms have an impact on the macro architecture.
While a system such as Kubernetes (see
[chapter 17](#chapter-kubernetes)) can run any kind of Docker
container, a PaaS works at the level of applications. A PaaS is
therefore more restrictive. For example, all Java applications will
probably be standardized to one or a few Java versions and Linux
distributions. Programming languages that are not supported by the PaaS
simply cannot be used to implement microservices. Monitoring and
deployment are determined by the selection of the PaaS. Therefore, a
PaaS creates an even higher standardization of the macro architecture
than it is the case in a Kubernetes environment.

Modern PaaS can be customized and are quite flexible. In that case the
way the PaaS is customized and which technology options are supported
can serve as a way to define the macro architecture. Then the macro
architecture is not defined by the PaaS supplier but by whoever
customizes the PaaS.

## 18.2 Cloud Foundry {#section-paas-cloudfoundry}

[Cloud Foundry](https://www.cloudfoundry.org/) serves as PaaS
technology for the example in this book. There are the following
reasons for this:

* Cloud Foundry is an *open source project* involving a number of
companies. Cloud Foundry is managed by a *foundation*, in which Cloud
Foundry providers such as Pivotal, SAP, IBM, and Swisscom are organized.
This ensures broad support, and in addition, there are already many
PaaS based on Cloud Foundry.

* Cloud Foundry can be easily installed as a Pivotal Cloud Foundry for
Local Development on a *laptop* to set up a local PaaS for developers
to test microservices systems.

* There are many public cloud providers who have an offering based on
Cloud Foundry. An overview can be found at:
  <https://www.cloudfoundry.org/how-can-i-try-out-cloud-foundry-2016/>.

* Finally, Cloud Foundry can be installed in your own data center.
[Pivotal Cloud Foundry](https://pivotal.io/platform) is for example on
option for this.

#### Flexibility

Cloud Foundry is a very flexible PaaS.

* Cloud Foundry supports applications in *different programming
languages*. A buildpack must be available for the chosen
programming language. The buildpack creates the Docker image from the
application, which is then executed by Cloud Foundry. The
[list of buildpacks](https://docs.cloudfoundry.org/buildpacks/) shows
which buildpacks can be downloaded from the Internet.
 
* In a Cloud Foundry system, *modified or self-written buildpacks* can
be installed. This enables support for additional programming
languages or the adaptation of existing support to your own needs.

* The *configuration of the buildpacks* can, for example, change
memory settings or make other adjustments. Thus, an existing or
self-written buildpack can be adapted to the needs of the
microservice.

* It is also possible to deploy
[Docker containers](https://docs.cloudfoundry.org/adminguide/docker.html)
in a Cloud Foundry environment. However, in this case it is important
to pay attention to the special features of Docker under Cloud
Foundry. Ultimately, this makes it possible to run virtually any
software with Cloud Foundry.

## 18.3 The Example with Cloud Foundry {#section-paas-beispiel}

The microservices in this example are identical to the examples from
the previous chapters (see [section 14.1](#section-netflix-beispiel)).

* The *catalog* microservice administrates the informationen
concerning the goods.

* The *customer* microservice stores the customer data.

* The *order* microservice can receive new orders. It uses the catalog
and customer microservice via REST.

* In addition, there is the *Hystrix dashboard*, a Java application
for visualizing the monitoring of the Hystrix circuit breaker.

* Finally, there is a web page *microservices*, which contains links
to the microservices and thus facilitates the entry into the system.

{id="fig-paas-beispiel"}
![Fig. 18-1: The Microservices System in Cloud Foundry](images/paas-beispiel.png)

#### Starting Cloud Foundry

[Section 0.4](#section-einleitung-quick-start) describes which
software has to be installed for starting the example.

A detailed description how the example can be built and started is
provided at
<https://github.com/ewolff/microservice-cloudfoundry/blob/master/HOW-TO-RUN.md>.

The Cloud Foundry example is available at
<https://github.com/ewolff/microservice-cloudfoundry>.
Download the code with `git clone
https://github.com/ewolff/microservice-cloudfoundry.git`.

To start the system the application is first compiled with Maven. To
do so, you have to execute `./mvnw clean package` (macOS, Linux) or
`mvnw.cmd clean package` (Windows) in the sub directory
`microservice-cloudfoundry-demo`.
See [appendix B](#anhang-maven) for more details on Maven and how to
trouble shoot the build.

The example is supposed to be started on a local Cloud Foundry
installation. The required installation is described at
<https://pivotal.io/pcf-dev>. Upon the start of the Cloud Foundry
environment the Paas should be assigned enough memory, for
example, with `cf dev start -m 8086`. After the login with `cf login
-a api.local.pcfdev.io --skip-ssl-validation` the environment should
be usable.

#### Deploying the Microservices

Now deploy the microservices. Execute `cf push`
in the sub directory `microservice-cloudfoundry-demo`. This command
evaluates the file `manifest.yml`, with which the microservices for
Cloud Foundry are configured. `cf push catalog` deploys a
single application such as `catalog`.


{lang="yaml"}
~~~~~~~~
---
memory: 750M
env:
  JBP_CONFIG_OPEN_JDK_JRE: >
   [memory_calculator:
    {memory_heuristics:
     {metaspace: 128}}]
applications:
- name: catalog
  path: .../microservice-cloudfoundry-demo-catalog-0.0.1-SNAPSHOT.jar
- name: customer
  path: .../microservice-cloudfoundry-demo-customer-0.0.1-SNAPSHOT.jar
- name: hystrix-dashboard
  path: .../microservice-cloudfoundry-demo-hystrix-dashboard-0.0.1-SNAPSHOT.jar
- name: order
  path: .../microservice-cloudfoundry-demo-order-0.0.1-SNAPSHOT.jar
- name: microservices
  memory: 128M
  path: microservices
~~~~~~~~

In detail, the following parts of the configuration can be distinguished.

* Line 2 ensures that each application is provided with 750 MB RAM.

* Line 3--7 changes the memory distribution so that enough memory
for the Java bytecode is available in the meta space of the JVM.

* Lines 8 to 16 configure individual microservices. At the same time
the JAR files to be deployed are specified. For each of the
microservices, the common settings in lines 2-7 apply. The paths to
the JARs
are abbreviated to increase the clarity of the listing.

* Finally, lines 17--19 deploy the application `microservices`, which
displays a static HTML page with links to the microservices. In the
directory `microservices` an HTML file `index.html` is stored and
an empty file `Staticfile`, which marks the content of the directory
as static web application.

Cloud Foundry uses the Java buildpack to create Docker containers,
which are then started. At <http://microservices.local.pcfdev.io/> the
static web page is provided which allows the user to use the
individual
microservices.

As you can see, the configuration to run the microservices on Cloud
Foundry is very simple.

#### DNS for Routing or Service Discovery

The microservices themselves have no code dependencies to Cloud
Foundry. DNS is used for service discovery. The order microservice
calls the catalog and customer microservices. To do this, it uses the
host names `catalog.local.pcfdev.io` and `customer.local.pcfdev.io`,
which are derived from the names of the services. The
`local.pcfdev.io` domain is the default, but can be customized in the
Cloud Foundry configuration. `catalog.local.pcfdev.io`,
`order.local.pcfdev.io`, and `customer.local.pcfdev.io` are also the
hostnames used in the web browser for the links in the HTML UI of the
microservices. Behind this is a routing concept in Cloud Foundry,
which makes the microservices accessible from outside and implements
load balancing.

With `cf logs`, it is possible to have a look at the microservices
logs. `cf events` returns the last entries. At
<https://local.pcfdev.io/> a dashboard with basic information about
the microservices and with an overview of the logs is available (see
[figure 18-2](#fig-paas-cloudfoundry-dashboard)).

{id="fig-paas-cloudfoundry-dashboard", width=75%}
![Fig. 18-2: Cloud Foundry Dashboard](images/paas-cloudfoundry-dashboard.png)

With `cf ssh catalog` the user can login into the Docker container in
which the microservice `catalog` runs. This allows the user to examine
the
environment more closely.

#### Using Databases and other Services

It is possible to provide the microservices with additional *services*
such as databases. `cf marketplace` shows the services in the
marketplace. These are all services that are available in the Cloud
Foundry installation. From these services, instances can then be
created and made available to the applications.

#### Example for a Service from the Marketplace

`p-mysql` is the name of the MySQL service provided by the local Cloud
Foundry installation that can be used to run the examples.
With `cf marketplace -s p-mysql` you can obtain an overview of the
different offerings for the service `p-mysql`. `cf cs p-mysql 512mb
my-mysql` generates a service instance named `my-mysql` with the
configuration `512mb`. The command `cf bind-service` can make the
service available to an application.

With `cf ds my-mysql` the service can be deleted again.

#### Using Services in Applications

The application must be configured with the information for accessing
the service. For this, Cloud Foundry uses environment variables that
contain server address, user account and password. The application
must read this information. There are different possibilities for this
in the respective programming languages. The
[buildpack documentation](https://docs.run.pivotal.io/buildpacks/)
contains more information.

A configuration using environment variables can also be used for
settings provided during the installation of the microservice. In the
`manifest.yml`, variables can be set that can be read by deployed
applications.

#### Services for Asynchronous Communication

Some services add asynchronous communication to Cloud Foundry. For
example, the local Cloud Foundry installation offers RabbitMQ and
Redis as services. These are both technologies that can send messages
asynchronously between microservices. Other Cloud Foundry offerings
can provide additional MOMs as services.

## 18.4 Recipe Variations {#section-paas-variationen}

This chapter refers to the PaaS concept. Cloud Foundry is not the only
available PaaS.

* [OpenShift](https://www.openshit.com/) supplements Kubernetes with
support for different programming languages to thereby automate the
generation of the Docker containers.

* [Amazon Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/)
  is only available in the Amazon Cloud. It can install applications
  in virtual machines and scale these virtual machines. Thus, Elastic
  Beanstalk represents a simplification compared to the IaaS approach.
  However, since Beanstalk is based on IaaS and some additional
  features, Beanstalk rests on a very stable foundation. In Beanstalk
  applications, additional services from the Amazon offer can be used.
  These include, for example, databases, but also MOMs. Thus, elastic
  Beanstalk benefits from the numerous components available in the
  Amazon Cloud. 

* [Heroku](https://www.heroku.com/) is only available in the public
cloud. Similar to Cloud Foundry it has buildpacks for supporting
different programming languages and a marketplace for additional
services.

## 18.5 Experiments {#section-paas-experimente}

* Supplement the Kubernetes system with an additional microservice.
  * As example you can take a microservice which can be used by a call
  center agent for making notes about a conversation. The call
  center agent should be able to select the customer. 
  * Calling the customer microservice has to use the host name
    `customer.local.pcfdev.io`.
  * Of course, you can copy and modify one of the existing microservices.
  * Enter the microservice in the file `manifest.yml`.
  * The deployment of a Java application can be derived quite easily
    from the existing `manifest. mf`. For other languages the
    [documentation of the buildpacks](https://docs.cloudfoundry.org/buildpacks/)
    might be helpful.

* Get acquainted with the possibilities of running Cloud Foundry.
Start the example and have a look at the logs with `cf logs`. Log into
the Docker container of an application with `cf ssh` and see the
latest events of a microservice with `cf events`.

It is also possible to use the Cloud Foundry infrastructure in other
areas of the application. However, these experiments are harder to do.

* Replace the integrated database in the microservices with MySQL. For
information on how to start MySQL with Cloud Foundry, see
[section 18.3](#section-paas-beispiel). However, the code and
configuration of the microservices must be changed in order for the
applications to use MySQL. There is a
[Guide](https://spring.io/guides/gs/accessing-data-mysql/) for that.

* Cloud Foundry also provides [RabbitMQ](https://www.rabbitmq.com/) in
the marketplace. This MOM can be used for asynchronous communication
between microservices. A
[guide](https://spring.io/guides/gs/messaging-rabbitmq/) shows how to
use RabbitMQ with Spring. So you can port the example from
[chapter 11](#chapter-kafka) to RabbitMQ and then run it with a
RabbitMQ instance created by Cloud Foundry.

* An Alternative are
  [user-provided service instances](https://docs.cloudfoundry.org/devguide/services/user-provided.html).
  With this approach, a microservice can obtain information about the
  Kafka instance with Cloud Foundry mechanisms. The Kafka instance is
  not running under the control of Cloud Foundry. Change the Kafka
  example from [chapter 11](#chapter-kafka) so that it uses a
  user-provided service Kafka instance. This concerns the
  configuration of the port and host for the Kafka server. 

* Change one of the microservices and use
[Blue/Green Deployment](https://docs.cloudfoundry.org/devguide/deploy-apps/blue-green.html)
to deploy the change so that the microservice does not fail during the
deployment. The Blue/Green Deployment first creates a new environment
and then switches to the new version so that no downtime occurs.

## 18.6 Serverless {#section-paas-serverless}

PaaS deploy applications. Serverless goes even further and enables the
deployment of individual functions. Thereby, Serverless allows even
smaller deployments than with PaaS. A REST service can be divided into
a variety of functions: one per HTTP method and resource.

The advantages of Serverless are similar to those of PaaS. A high
degree of abstraction and thus relatively simple deployment. In
addition, Serverless functions are only activated when a request is
made so that no costs are incurred if no requests are processed.
They can also scale very flexibly.

Serverless technologies include
[AWS Lambda](https://aws.amazon.com/lambda/),
[Google Cloud Functions](https://cloud.google.com/functions/),
[Azure Functions,](https://azure.microsoft.com/services/functions/) and
[Apache OpenWhisk](https://developer.ibm.com/code/open/apache-openwhisk/).
OpenWhisk allows you to install a Serverless environment in your own
data center.

As with a PaaS, Serverless also includes support for operation.
Metrics and log management are provided by the cloud provider.

#### REST with AWS Lambda and the API Gateway

With AWS Lambda, REST services can be implemented. The API gateway in
the AWS cloud can call Lambda functions. A separate function can be
implemented for each HTTP operation. Instead of a single PaaS
application, there are many Lambda functions.

A technology like
[Amazon SAM](http://docs.aws.amazon.com/lambda/latest/dg/deploying-lambda-apps.html) 
can make the large number of Lambda functions 
quite easy to use. So even if there are a lot of functions to deploy, it
is hardly any more effort for the developers than if they implement
the REST methods in a class. Often Lambda functions can also
significantly reduce the cost of running a solution.

#### Glue Code

In another area, Lambda functions are also very helpful. In response
to an event in the Amazon Cloud, a Lambda function can be called.
[S3](https://aws.amazon.com/s3/) (Simple Storage Service) offers
storage for large files in the Amazon Cloud. When a new file is
uploaded, a Lambda function can converted it to another
format. However, these are
not real microservices, but rather glue code to complement
functionalities in Amazon services.

## 18.7 Conclusion {#section-paas-fazit}

For synchronous microservices, Cloud Foundry's solutions are very
similar to those of Kubernetes.

* *Service discovery* also works via DNS. It is therefore transparent
for client and server. In addition, no code needs to be written for
the registration.

* *Load balancing* is also transparently implemented by Cloud Foundry.
If several instances of a microservice are deployed, the requests are
automatically
distributed to these instances.

* For the *routing* of external requests, Cloud Foundry relies on DNS
and a distribution of the requests to the various microservice instances.

* For *resilience*, the Cloud Foundry example uses the Hystrix
library. Cloud Foundry itself does not offer a solution in this area.

In addition, a PaaS provides a standardized runtime environment for
microservices and can therefore be an important antidote to the high
level of operational complexity that microservices bring. Thus, a PaaS
enforces a standardization that is often desirable in the context of
macro architecture (see [chapter 2](#chapter-mikro-makro)).

Compared to Docker ([chapter 5](#chapter-docker)) or Kubernetes
([chapter 17](#chapter-kubernetes)) a PaaS provides less flexibility. This
can also be a strength due to the standardization that goes hand in
hand with reduced flexibility. Modern PaaS also offer the possibility
of adapting the environment with concepts such as buildpacks or even
of running Docker containers with arbitrary applications.

#### Benefits

* PaaS solve typical problems of microservices (load balancing, routing,
service discovery).

* Cloud Foundry introduces no code dependencies.

* PaaS also cover operation and deployment.

* PaaS enforce standardization and thereby the definition of a macro
  architecture.

* Developers only have to deliver applications. Docker is hidden.

#### Challenges

* Cloud Foundry requires a complete switch of the operation approach.

* Cloud Foundry is very powerful, but thus also very complex.

* Cloud Foundry provides a high degree of flexibility, however,
  compared to Docker containers this flexibility still has its limits.

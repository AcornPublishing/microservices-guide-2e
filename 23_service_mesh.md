# 23 Recipe: Service Mesh Istio {#chapter-service-mesh}

Service meshes are a type of infrastructure that supports typical
challenges of microservices.

In this chapter, readers learn:

* What service meshes are.

* How service meshes solve most of the challenges of microservice
  with no impact on the code.

* What Istio as the most popular service mesh provides and how
  it can be used.


## 23.1 What is a Service Mesh? {#section-service-mesh-what}

The last chapters have shown the technical challenges associated with
microservices. [Chapter 4](#chapter-docker) explained that Docker
simplifies the packaging and deployment of microservices. [Chapter
17](#chapter-kubernetes) has shown that Kubernetes provides service
discovery, load balancing and routing for
microservice. Kubernetes works with any type
of microservice and has no impact on the code. However, the support for
resilience is limited.

To fully support the operation of microservices, infrastructure for
monitoring ([chapter 20](#chapter-monitoring)), log analysis ([chapter
21](#chapter-log-analyse)) and tracing ([chapter
22](#chapter-tracing)) also has to be provided.

So while a platform like Kubernetes provides a lot of features, it
is still missing a few features.

#### Istio

*Istio* supports these features for the operation of microservices. It
also supports:

* For operation, Istio integrates technologies for monitoring, log
analysis and tracing.
The data for these features is measured by Istio. So there is no need
to add any code to the microservice. That way, Istio transparently
solves the main issues
for the operation of microservice systems.

* *Mutual TLS* (Transport Layer Security) adds encryption to the
  communication between the microservices. Also it distributes
  certificates to the microservice. That way, each microservice can be
  authenticated. So even if an attacker is able to communicate with a
  microservice, he / she won't have such a certificate and the attack
  can easily be defended against.
  Istio also supports other security features like authorization.
  
* Istio adds advanced features for *routing* to Kubernetes. That
  allows
  e.g. A/B testing. Using this approach, the requests of some user are
  forwarded to one
  version of the system while the requests of others user are
  forwarded to a different version. That allows to find out which
  version is more
  attractive or generates more revenue. It is also possible to send
  the traffic to the new and old version of a microservices at the
  same time to make sure that the new version behaves like the old
  version (mirroring).

* *Resilience* is supported. A timeout can be added to a request and
  requests can be retried. Also circuit breaker can be used to protect
  microservices from overload.

Istio does not solve all challenges of microservices.
A different infrastructure such as Kubernetes has to
provide basic features e.g. for deployment and service
discovery.

Istio is licensed under the liberal Apache license. It allows users
to freely modify and distribute the code. Google and IBM started the
project in partnership with the team for the Envoy proxy at Lyft.


## 23.2 Example {#section-service-mesh-example}

The example in this chapter contains the same microservices as the
example in the Atom
chapter (see [section 12.2](#section-atom-beispiel)).

{id="fig-service-mesh-ueberblick"}
![Fig. 23-1: Overview of the Service Mesh Example](images/service-mesh-ueberblick.png)

[Figure 23-1](#fig-service-mesh-ueberblick) shows the structure of the
example:

* The *Ingress Gateway* is provided by Istio. It forwards HTTP request to
  the microservices. It is similar to a Kubernetes Ingress (see
  [section 17.2](#section-kubernetes-beispiel). However, the Istio
  Gateway supports Istio's features mentioned above like monitoring or
  advanced routing.
  
* *Apache httpd* provides a static HTML page that serves as the
  homepage for the example. The page has links to each
  microservice. Apache httpd is configured by a Kubernetes
  deployment. That 
  deployment creates a Kubernetes replica set with one Kubernetes
  pod. A
  Kubernetes service and a route for the Ingress gateway ensure access to
  the Apache httpd.
  
* *Order, shipping* and *invoicing* are microservices. Shipping and
  invoicing poll data about the orders from the order
  microservice. They use REST and Atom as
  described in [chapter 12](#chapter-atom). The microservices each
  have a deployment, a Kubernetes service and a route for the Ingress
  gateway like the Apache httpd mentioned above.
  
* All three microservices use the same *Postgres database*. However,
  they use different database schemas. So each microservice can change
  its schema and therefore its domain model without any impact on the
  other microservices. The Postgres database is installed by a
  Kubernetes deployment. A Kubernetes
  service ensures access to it.
  
So for load balancing, service discovery and deployment the example
relies on Kubernetes with the concepts described in [chapter
17](#chapter-kubernetes).

The reasons for choosing this example in this chapter are:

* The example uses REST. Some of Istio's features support
  HTTP and HTTP/2 which REST uses.
  
* It is based on an asynchronous architecture. A synchronous REST
  system would be an even better example
  for Istio's features. However, the asynchronous architecture
  benefits
  resilience and solves a core challenge for microservices.

#### Run the Example

[Section 0.4](#section-einleitung-quick-start) describes which
software has to be installed for starting the example.

The code for the example is accessible at
<https://github.com/ewolff/microservice-istio>. The
[documentation](https://github.com/ewolff/microservice-kubernetes/blob/master/HOW-TO-RUN.md)
explains in detail how to install the required software and how to run
the example.

The following steps are necessary for running the example:

* [Minikube](https://github.com/kubernetes/minikube) is a minimal
Kubernetes version. It has to be installed. Instructions for this can
be found at <https://github.com/kubernetes/minikube#installation>.

* [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/)
is a command line tool for handling Kubernetes. It also has to be
installed. Its installation is described at
<https://kubernetes.io/docs/tasks/tools/install-kubectl/>.

* [Download](https://istio.io/docs/setup/kubernetes/download-release/)
and [install](https://istio.io/docs/setup/kubernetes/quick-start/)
Istio.  It is enough to install Istio *without* mutual TLS
authentication between sidecars.

* Change to the directory `microservice-istio-demo` and run `./mvnw clean
package` or `mvnw.cmd clean package` (Windows) to compile the Java
code. [Appendix B](#anhang-maven) discusses Maven in more detail and
contains hints on how to trouble shoot the build.

* Configure Docker so that it uses the Kubernetes cluster. This is
required to install
the Docker images: `minikube docker-env`(MacOS or Linux) or
`minikube.exe docker-env`(Windows) tells you how to achieve that.

* Run `docker-build.sh` in the directory
`microservice-istio-demo`. It builds the images and uploads them into the
Kubernetes cluster. [Appendix C](#anhang-docker) discusses the
`docker`command line tool in more detail and shows some ways to
trouble shoot it. If you are not on a system with a shell that can run
the script, you can issue the commands in the script manually.

* Make sure that the Istio containers are automatically injected when
the pods are started: 
`kubectl label namespace default istio-injection=enabled`

* Deploy the infrastructure for the microservices using `kubectl apply
-f infrastructure.yaml` in the directory
`microservice-kubernetes-demo`. This creates the Apache httpd server,
the Postgres server and the Istio gateway. Also it adds a route from
the Istio gateway to the static HTML pages stored in the Apache httpd
server.

* Then you can deploy the microservices using `kubectl apply -f
  microservices.yaml`. This configuration contains the deployment,
  services and
  routes for the microservices order, invoicing and shipping.
  
* To use the demo, you need to figure out the URL of the Istio
  gateway. The shell script `ingress-url.sh` does that.
  
* Open the URL in a web browser. 
  The static overview page with links to the microservices should be
  displayed.
  You should be able to enter an order
  in the order microservice. After a while, the shipping and invoice
  should appear in the other microservices. The microservices poll
  every 30 seconds so there might be a slight delay.
  
If you run into any problems, please refer to the [online
documentation](https://github.com/ewolff/microservice-istio/blob/master/HOW-TO-RUN.md). It
contains an even more extensive documentation and some strategies for
trouble shooting. 

#### Adding another Microservice

There is another microservice in the sub directory
`microservice-istio-bonus`. This microservice has a completely
separate build. A separate build
might be the better option compared to the shared build of the other
microservices. Microservice should be separately
deployable. Because the build is the first step in a deployment
pipeline, the build should ideally be separate, too.

To add the microservice you can do the following:

* Change to the directory `microservice-isitio-demo` and run `./mvnw clean
package` or `mvnw.cmd clean package` (Windows) to compile the Java
code.

* Run `docker-build.sh` in the directory
`microservice-istio-bonus`. It builds the images and uploads them into
the Kubernetes cluster.

* Deploy the microservice with `kubectl apply -f bonus.yaml`.

* You can remove the microservice again with `kubectl delete -f bonus.yaml`.

#### Adding a Microservice with Helm

The configuration of the microservice in `bonus.yaml` is very
similar to the configuration of the other microservices. It makes
little sense to repeat the common parts of the configuration for every
microservice. [Helm](https://helm.sh/) is a package manager for
Kubernetes that supports templates for Kubernetes configurations. Helm
can make it easier to add a new microservice.

* First you need to [install
  Helm](https://docs.helm.sh/using_helm/#installing-helm) into the
  Kubernetes cluster.
  
* The directory `spring-boot-microservice` contains a Helm chart for
  the microservices in this project. With Helm, there is no need to run `kubectl
  apply -f bonus.yaml`. Instead, `helm install --set name=bonus
  spring-boot-microservice/` is enough to deploy the microservice. For
  a different microservice just a different `name` has to be
  provided. So the Helm chart provides a generic mechanism to deploy
  microservices. A Helm deployment is also called a "release".
  
{linenos=off}
~~~~~~~~
[~/microservice-istio]helm install --set name=bonus  spring-boot-microservice/
NAME:   waxen-newt
LAST DEPLOYED: Wed Feb 13 14:25:52 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME   TYPE      CLUSTER-IP     EXTERNAL-IP  PORT(S)         AGE
bonus  NodePort  10.108.31.211  <none>       8080:30878/TCP  5s

==> v1beta1/Deployment
NAME   DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
bonus  1        1        1           1          5s

==> v1alpha3/VirtualService
NAME   AGE
bonus  1s

==> v1/Pod(related)
NAME                    READY  STATUS   RESTARTS  AGE
bonus-55d854b9d9-sn4pm  2/2    Running  0         4s
~~~~~~~~

* A name is automatically assigned to the Helm release. In this example,
  the name is `waxen-newt` and it printed out by Helm. `helm list`
  provides a list of releases
  that are currently installed.

* You can remove the Helm release by using this assigned name
  e.g. `helm delete waxen-newt`.

Helm offers a very easy to deploy microservices that match the
structure of the other microservices. In a system with many
microservices, Helm charts simplify deployment of microservices and
also make the deployments more uniform.

#### Using the Additional Microservice

The bonus microservice is not included in the static web page. So
there is 
no link to it. However, the bonus microservice can also be accessed
via the Ingress gateway. If the Ingress gateway's URL is
<http://192.168.99.127:31380/>, you can access the bonus microservice
at <http://192.168.99.127:31380/bonus>.

Note that the bonus microservice does not show any revenue for the
orders. This is because it requires a field `revenue` in the data the
order microservice provides. That field is currently not included in
the data. This shows that adding a new microservice might
require changes to a common data structure. Such changes might also
impact the other microservices because the other microservices use the
shared
data structure, too.

## 23.3 How Istio Works {#section-service-mesh-how}

If take a closer look at the `microserivces.yaml` file, you will
notice that the deployment for the microservices has no Istio specific
information. This is an advantage. It makes it
easier to use Istio. Also it means that Istio works with any type of
microservices no matter what programming language or which frameworks
are used.

However, Istio supports features like resilience
and monitoring as mentioned. Somehow Istio needs to collect
information about the microservices.

#### Sidecar

The idea behind a *sidecar* is to add another Docker container to each
Kubernetes pod. Actually, if you list the Kubernetes pods with
`kubectl get pods`, you will notice that for each pod it says `2/2`: 

{linenos=off}
~~~~~~~~
[~/microservice-istio/microservice-istio-demo]kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
apache-7f7f7f79c6-jbqx8      2/2     Running   0          8m51s
invoicing-77f69ff854-rpcbk   2/2     Running   0          8m43s
order-cc7f8866-9zbnf         2/2     Running   0          8m43s
postgres-5ddddbbf8f-xfng5    2/2     Running   0          8m51s
shipping-5d58798cdd-9jqj8    2/2     Running   0          8m43s
~~~~~~~~

So while there is just one Docker container configured for each
Kubernetes pod, there are in fact two Docker container running. One
container contains the 
microservice. The other contains the sidecar that enables the
integration into the Istio infrastructure.

Istio automatically injects these containers into each pod. During the
installation described above, `kubectl label namespace default
istio-injection=enabled` marked the `default` namespace so that Istio
injects sidecars for each pod in that namespace. Namespaces are a
concept in Kubernetes to separate Kubernetes resources. With Istio,
there is the `default` namespace that contains all Kubernetes
resources provided by the user. The namespace `istio-system` contains
all Kubernetes resources that belong to Istio itself.

#### Proxy

Just injecting a sidecar is not enough. The Istio sidecar has to get
information and metrics about the microservice.
Istio routes all outgoing and incoming traffic through a
*proxy*. This [proxy](https://github.com/istio/proxy) is an extended
version of the [Envoy proxy](https://www.envoyproxy.io/). All network
traffic passes through the proxy. So the proxy
can modify and measure all traffic. That makes the service mesh
transparent and enables its features.

{id="fig-service-mesh-proxy"}
![Fig. 23-2: Proxy and Data Plane in Istio](images/service-mesh-proxy.png)

[Figure 23-2](#fig-service-mesh-proxy) shows that all traffic from one
microservice container (e.g. invoicing) to another container
(e.g. order) is routed through two proxies. So it seems the microservice
is communicating directly with the other
microservice but in fact the communication is intercepted by the
proxies.

The Istio proxy can handle any TCP based protocol. However, it has specific
support for HTTP 1.1, HTTP 2 and gRPC. For those protocols it can
determine whether an operation was successful or not by evaluating
e.g. the HTTP status code.

#### Data Plane

The proxies are the part of Istio that is also called the *data
plane*. It is responsible
for the exchange of data between microservices.

#### Control Plane

To do anything meaningful with the intercepted traffic, Istio uses
the *control plane*. It is responsible for configuring
the proxies in the data plane and processing data from the proxies.

{id="fig-service-mesh-control-plane"}
![Fig. 23-3: Control Plane in Istio](images/service-mesh-control-plane.png)

[Figure 23-3](#fig-service-mesh-control-plane) shows that the control
plane consists three components:

* *Pilot* converts the routing information in Istio into configuration
  for the proxies. This enables e.g. A/B testing. It also supports
  resilience with
  e.g. timeouts or circuit breakers. Also it abstracts away service
  discovery mechanisms of the underlying platform. This enables Istio
  to support other infrastructure besides Kubernetes. So for example,
  Istio also supports Consul as a basic infrastructure instead of
  Kubernetes.
  
* For each request, *Mixer* evaluates policies. These are used e.g. to
  determine which kinds of metrics should be measured for a request and
  which components should receive them. But the policies might also
  define which microservice can call a specific microservice or they
  can enforce quotas. 

* *Citadel* manages certificates and cypher keys to enable
  encryption. It
  also support authentication based on the identity of a
  microservice and
  authentication for end users. An end user is authenticated
  with a token. Access to specific microservices can be limited to
  specific users. There is a [hands-on
  example](https://istio.io/docs/tasks/security/authn-policy/) for this.

So by injecting the proxy into the network traffic between
microservices, Istio transparently enables many features that
otherwise would need
to be implemented in the microservice.

## 23.4 Monitoring with Prometheus and Grafana {#section-service-mesh-monitoring}

As discussed in [chapter 20](#chapter-monitoring), a microservice
system should include a monitoring infrastructure that collects
monitoring information from all microservices and makes them
accessible. This is required to keep track of the metrics for the huge
number of
distributed microservices. Based on these metrics, alarms and analysis
can be implemented.

{id="fig-service-mesh-monitoring"}
![Fig. 23-4: Monitoring in Istio](images/service-mesh-monitoring.png)

[Figure 23-4](#fig-service-mesh-monitoring) shows how monitoring is
supported by Istio:

* The *proxy* collects metrics such as the duration of a request on
  the client and
  server side, the status code and the number of requests.
  
* This information is collected by *Mixer*.

* *[Prometheus](https://prometheus.io/)* (see [chapter
  20](#chapter-monitoring)) stores these metrics
  them. The metrics can be analyzed with Prometheus.
  
* *[Grafana](https://grafana.com/)* provides more advanced tools for
  the analysis of the metrics.
  
With a default Istio installation, all of these components are
configured, installed and ready to be used. There is no additional
effort to enable monitoring for the microservices system.

Of course, this approach only supports metrics that the proxy can
measure. This
include all the information about the request like its duration or the
status code. Also information about
the Kubernetes infrastructure e.g. CPU utilization can be
measured. However, data about the internal state of the microservice
is not measured. To get that data, the microservice would need to
report it to the monitoring infrastructure.

The metrics are sent asynchronously. It is not necessary to send them
synchronously. It is fine if they are received and processed at a
later point in time. By doing more communication asynchronously, the
additional latency of the synchronous communication can be limited.

#### Monitoring in the Example

The example uses only the metrics provided by the proxies and the
Kubernetes infrastructure for monitoring. This is different
from the example in [chapter 20](#chapter-monitoring). The example in
that chapter relies on the microservices to provide the metrics.
However, Istio's approach used in this chapter can be a
good alternative.

* It has no impact on the code of the microservice whatsoever. Metrics
  are only
  measured by the proxy. So any microservice will report the same
  metrics -- no matter which programming language they are written in
  or which framework they use.

* The metrics give a good impression about the state of the
  microservice. The metrics show the performance and reliability that
  a
  user or client would see. This is enough to ensure that service
  level
  agreements and quality of service is met. With an
  infrastructure like Kubernetes and Istio, activities such as restarts
  of failing microservices are automated. So some metrics that would make
  an administrator restart a service are not that important any more.
  
The metrics provided by Istio might be enough to manage a
microservices system. In that case, there is no need for any macro
architecture rule for monitoring (see [section
2.3](#section-mikro-makro-betrieb)). Still it is possible to
operate the system.

#### Prometheus in the Example

Metrics only make sense if there is load on the system. The shell
script `load.sh` uses the tool `curl` to request a certain URL 1,000
times. You can start one or multiple instances of this script with the
URL of the homepage of the shipping microservice to create some
load on that service.

The script `monitoring-prometheus.sh` uses `kubectl` to create a proxy
for the Prometheus server on localhost. That way, Prometheus can be
accessed with the URL <http://localhost:9090/>. You can use metrics
like `istio_requests_total` or `istio_request_bytes_count` as an
example for 
the metrics Istio reports to Prometheus. The metrics are
multi-dimensional i.e. they can be summed up by HTTP status code,
source or destination.

{id="fig-service-mesh-prometheus", width=65%}
![Fig. 23-4: Prometheus with Istio Metrics](images/service-mesh-prometheus.png)

For example, [Figure 23-4](#fig-service-mesh-prometheus) shows the
bytes count for requests. There are different destinations in the
graph: the order microservice and also the Istio component that
measures the telemetry data.
The destination is one dimension of the data.
These metrics could be summed up by dimensions such as the destination
to understand
which destination receives how much traffic. 

#### Grafana in the Example

For more advanced analysis of the data, Istio provides an
installation of Grafana. The easiest way to use the Grafana
installation is to start the shell script
`monitoring-grafana.sh`. It creates a proxy on localhost for the
Grafana installation. You can use the URL
<http://localhost:3000/> to access the Grafana installation then.

{id="fig-service-mesh-grafana", width=65%}
![Fig. 23-5: Grafana with Istio Dashboard](images/service-mesh-grafana.png)

The Grafana installation in Istio provides predefined
dashboards. [Figure 23-5](#fig-service-mesh-grafana) shows an example
of the Istio service dashboard. It uses the shipping
microservices. The dashboard shows metrics like the request
volume, the success
rate and the duration. This gives a great overview about the
state of the service.

There are not just dashboards for the microservices.
The Istio performance dashboard provides a general overview about the
state of the Kubernetes cluster with metrics like memory
consumption or CPU utilization. The Istio mesh dashboard shows a
global metric about the number of request the service mesh processes
and their success rates.

## 23.5 Tracing with Jaeger {#section-service-mesh-tracing}

For tracing, Istio uses [Jaeger](https://www.jaegertracing.io/). This
tool is similar to [Zipkin](https://zipkin.io/) (see
[chapter 22](#chapter-tracing)). Jaeger supports collecting
tracing information with the Zipkin format but also with the
[OpenTracing](https://opentracing.io/) standard. Jaeger is easier to
deploy on Kubernetes. There are official templates to do the
deployment on that infrastructure.
So it is a more natural choice for a Kubernetes infrastructure.

Tracing solves a common problem in microservices systems.
A request to a microservice might result in other requests.
Tracing helps to understand these dependencies. That facilitates root
cause analysis.

To
understand which incoming request caused which outgoing requests,
Jaeger relies on HTTP header. The values in the headers of the
incoming requests have to be added to any outgoing request.
This means that tracing cannot be transparent to the
microservices. They have to include some code to forward the tracing
headers from the incoming request to each outgoing request. For
Jaeger, these headers are `x-request-id`,
`x-ot-span-context`, `x-b3-traceid`, `x-b3-spanid`,
`x-b3-parentspanid`, `x-b3-sampled` and `x-b3-flags`.

#### Tracing in the Example

The example uses [Spring Cloud
Sleuth](https://spring.io/projects/spring-cloud-sleuth). This is a
powerful library that supports many features for
tracing. The example in [chapter 22](#chapter-tracing) used
Spring Cloud Sleuth to measure and report the tracing information to
Zipkin. However, for the example in this chapter, Spring
Cloud Sleuth just needs to forward the HTTP headers. So in
`application.properties` the parameter
`spring.sleuth.propagation-keys` containes the HTTP headers that must
be
forwarded.
The `x-b3-*` header are automatically forwarded by Spring Cloud Sleuth
so 
just the `x-request-id` and `x-ot-span-context` header have to be
configured.

The example in [chapter 22](#chapter-tracing) relies on
Spring Cloud Sleuth to also publish the trace data to the Zipkin. This
is not necessary for the example in this chapter. The Istio proxies
forward the information.

For other languages, different means to forward the HTTP headers would
be required. So concerning the macro architecture (see [section
2.3](#section-mikro-makro-betrieb)), the only necessary rule is to
forward the
HTTP headers. How the microservices implement this feature is up to
them. It would be the easiest to use Spring Boot and Spring
Cloud Sleuth.
However, there is no need for a strict rule to use the same
technology. This
makes sure that technology freedom is preserved.

To see the tracing information, use `tracing.sh` to start a
proxy on localhost. Then you access the Jaeger UI at
<http://localhost:16686/>. 

{id="fig-service-mesh-tracing", width="50%"}
![Fig. 23-6: Jaeger Trace](images/service-mesh-tracing.png)

[Figure 23-6](#fig-service-mesh-tracing) shows an example of a
trace. It is a request to the shipping microservice. The
user started a poll for new data on the order microservice. Then the
service contacted the Istio Mixer to make sure the policies are enforced.

The trace just shows the interaction of two microservices (shipping
and order). It is not
very complex. Most of the time is spent in the order microservice. So
the trace does not provide a lot of useful information to improve
performance. The only way to make this request faster is to make the
order microservice faster. However, the trace does not show where the
order microservice spends its time.

This lack of potential to optimize based on the trace is a result of
the
architecture. Almost all requests are handled by a single
microservice. The trace in the figure is one of the few
exceptions but even that example only includes two
microservices. Therefore, the trace does not provide a lot of valuable
information. However, that also means that the distributed nature of
the system has not such a huge impact and the system is still
easy to deal with.

{id="fig-service-mesh-tracing-dependencies", width="50%"}
![Fig. 23-7: Jaeger Dependencies](images/service-mesh-tracing-dependencies.png)

[Figure 23-7](#fig-service-mesh-tracing-dependencies) shows a
different type of information Jaeger provides: The dependencies
between the microservices. Shipping and invoicing use order to
receive the information about the latest orders. Order reports metrics
to Mixer. And finally order is accessed by the Istio
gateway when external requests are forwarded to it. This information
about dependencies
might be useful to get an overview about the architecture of
the system.

## 23.6 Logging {#section-service-mesh-logging}

Mixer can forward information about each HTTP request to a logging
infrastructure. That information can be used to analyze e.g. the
number of
requests to certain URLs and status codes.
A lot of statistics for web sites rely on this kind of
information.

The format in the logs can be configured. It may contain any
information Mixer received from the request.

However, Istio does not provide an infrastructure to handle logs. To
make use of the logging information, some storage for the
logs such as Elasticsearch is needed. Also an analysis frontend
like Kibana is necessary. This is not part of a standard Istio
installation.

To learn about how Istio supports logs, there is an
[example](https://istio.io/docs/tasks/telemetry/metrics-logs/). It
shows how log information can be composed from Mixer's data. This
example outputs the logs to stdout and not to a log infrastructure.

Also there is an
[example](https://istio.io/docs/tasks/telemetry/fluentd/) that shows
how [Fluentd](https://www.fluentd.org/) can be used to collect the
logs provided by Istio from all microservices. The logs are stored in
Elasticsearch and evaluated with Kibana.

#### Logs in the Example

For the example, a custom log
infrastructure was
set up. This infrastructure uses Elasticsearch to store logs and
Kibana to analyze them -- just like the example in [chapter
21](#chapter-log-analyse).

{id="fig-service-mesh-logging"}
![Fig. 23-8: Loggging in the Example](images/service-mesh-logging.png)

[Figure 23-8](#fig-service-mesh-logging) shows how logging is
implemented in the example. Each microservice must directly
write JSON data to the Elasticsearch server. So there is no need to
write any log files which makes the system easier to
handle. Also there is no need to parse the log
information. Elasticsearch can directly process it.

The example uses the [Logback](https://logback.qos.ch/) Java
library. The [Logback
Elasticsearch
Appender](https://github.com/internetitem/logback-elasticsearch-appender)
forwards the logs to Elasticsearch. The configuration in the file
`logback-spring.xml` defines what information the microservices log.

However, concerning the macro architecture (see [section
2.3](#section-mikro-makro-betrieb)), there would only be a rule that
requires microservices to log JSON to the Elasticsearch
server. Probably there would also be a rule about which information
should be included in a log entry e.g. the host, severity and of
course the
message. Not just Java microservices but also microservice written in
a different language or using a
different library could also conform to such a macro architecture rule.

It would be possible to make the Istio infrastructure log to the same
Elasticsearch instance, too. However, for the example it was decided
that this is not necessary. With the current system, it is easy
to find problems in the implementation by searching for log entries
with severity error. Also logging each HTTP request adds little value. 
Information about the HTTP requests is probably already included in
the logs of the microservices.
This
is probably the case for systems in production environments, too.

Generally speaking, Istio's logging support has the advantage that
developers do not have to care about these logs at all. Also the logs
are
uniform no matter what kind of technology is used in the microservices
and how they log. Enforcing a common logging approach and logging
format takes some effort. This is in particular true for microservices
that use
different technologies. So while Istio's logs might not include
information from inside the microservices, they are easy to get. Such
a log might be better than no log at all or no uniform log.

## 23.7 Resilience {#service-mesh-resilience}

Resilience means that a microservice should not fail if other
microservices fail. It is important to avoid failure cascades that
could bring down the complete microservices system.

#### Measuring Resilience with Istio

Failure cascades can happen if a called microservice returns an
error. It could be even worse if the called microservices does return
successfully but it takes very long. In that case, resources such as
threads might be blocked while waiting for a reply. In the worst case,
all threads end up blocked and the calling microservice fails.

Such scenarios are hard to simulate. Usually the networks is
reasonably reliable. It would be possible to implement a stub
microservice that returns errors but that would require some
effort.

However, Istio controls the network communication through the
proxies. It is therefore possible to add delays and errors to specific
microservices. Then the other microservices can be checked to see if
they are resilient against the delays and failures.

{linenos=off, lang="yaml"}
~~~~~~~~
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-fault
spec:
  hosts:
  - order.default.svc.cluster.local
  http:
  - fault:
      abort:
        percent: 100
        httpStatus: 500
    route:
    - destination:
        host: order.default.svc.cluster.local
~~~~~~~~

The listing above shows the content of the file `fault-injection.yaml`
from the example. It makes 100% of the calls to the order microservice
fail with HTTP status 500. You can apply it to the example with
`kubectl apply -f
fault-injection.yaml` and remove it again with `kubectl delete -f
fault-injection.yaml`.

Actually, the microservice will still work after
applying the fault injection. If you add a new order to the system, it
will not be propagated to shipping and invoicing, though. You can make
those microservices poll the order microservice by pressing the pull
button in the web UI of shipping and invoicing. In that case, an
error will be shown. So the system is already quite resilient because
it uses asynchronous communication.
The shipping and invoicing microservices still work. The only
exception is the polling of the order microservice.
If the shipping microservice would
call the order microservice synchronously e.g. to fulfill a request,
the shipping service
would fail after the fault injection if no additional logic is
implemented to handle such a failure.

{linenos=off, lang="yaml"}
~~~~~~~~
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-delay
spec:
  hosts:
  - order.default.svc.cluster.local
  http:
  - fault:
      delay:
        fixedDelay: 7s
        percent: 100
    route:
    - destination:
        host: order.default.svc.cluster.local
~~~~~~~~

It is also possible to inject a delay as the listing above shows. You
can apply it to the system with `kubectl apply -f
delay-injection.yaml` and remove it again with `kubectl delete -f
delay-injection.yaml`. If you make the shipping microservice poll
the order microservice, it will take longer but it will work
fine. Otherwise the system just works normally. So also for delays the
asynchronous communication solves the resilience problem.

It is also possible to limit delays and faults e.g. by HTTP headers. So
you can run specific test requests in the production system
and make them fail or hit a delay to understand how the production
system would handle such problems. The normal requests would be
unaffected.

#### Implementing Resilience with Istio

[Section 14.5](#section-netflix-hystrix) has described several
resilience patterns like timeout or circuit breaker. They can be
implemented with a library like Hystrix as described in that
section. However, this
would require changes to the code and can limit the free choice of
technologies.

Because Istio adds proxies to the communication between the
microservices, it is possible to add a circuit breaker without
changing the code. 

{linenos=off, lang="yaml"}
~~~~~~~~
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: order-circuit-breaker
spec:
  host: order.default.svc.cluster.local
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        http2MaxRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 1
      interval: 1m
      baseEjectionTime: 10m
      maxEjectionPercent: 100
~~~~~~~~

The listing above shows a configuration of a circuit breaker for
Istio. It is available in the file `circuit-breaker.yaml` and can be
added to the example using `kubectl apply -f circuit-breaker.yaml`. It
can be removed with `kubectl delete -f circuit-breaker.yaml`. The
configuration has the following settings:

* A maximum of one TCP connection is allowed for the service (`maxConnections`).

* There may be just one request per connection
  (`maxRequestsPerConnection`).

* In total just one
  HTTP 1.1 (`http1MaxPendingRequests`) and one HTTP 2
  (`http2MaxRequests`) request might be pending.
  
* Each minute each microservice instance is checked (`interval`). If it
  has returned
  one error (`consecutiveErrors`) i.e. an HTTP status 5xx or a
  timeout, it is excluded from traffic
  for ten minutes (`baseEjectionTime`). All instances of the
  microservice might be excluded 
  from traffic in this way (`maxEjectionPercent`).

The goal of the circuit breaker is to protect microservices from
too much load. That is the reason for limiting the maximum number of
connection. Also the number of pending requests is limited. If the
microservice is too slow to handle all traffic, this protects the
microservice from overload. And if an instance has already failed, it
is excluded from the work. That gives the instance a chance to
recover.

The limits in the example are very low to make it easy to trigger the
circuit breaker. In a production environment, the values should
be higher. If you use the `load.sh` script to access the
order microservice's web UI, you will just need to run a few instances
in parallel to receive 5xx error codes that are returned by the
circuit breaker.

If the circuit breaker does not accept a request because of
the defined limits, the calling microservice will receive an
error. So the calling microservice is not protected from a failing
microservice. Quite the contrary: To protect the called microservice
instances, the circuit breaker might increase the number of failed
requests.

##### Retry and Timeout

If a called microservice fails, the calling microservice should not
fail. Istio provides two measures to achieve this:

* *Retries* repeat the failed requests. If the failure is transient,
  this can make the request succeed. However, it also adds load to the
  called microservices. A circuit breaker might be useful to
  protect it from overload. A request is considered failed if e.g. an
  HTTP status 5xx is returned.
  
* *Timeouts* make sure that the calling microservice is not blocked
  for too long. Otherwise if all threads are blocked, the calling
  microservice might not be able
  to accept any more requests.

{linenos=off, lang="yaml"}
~~~~~~~~
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-retry
spec:
  hosts:
  - order.default.svc.cluster.local
  http:
  - retries:
      attempts: 20
      perTryTimeout: 5s
    timeout: 10s
    route:
    - destination:
        host: order.default.svc.cluster.local
~~~~~~~~

The listing above shows a part of the file `retry.yaml`. It configures
retries and timeouts for the order microservice. Calls to the order
microservice are retried up to 20 times. For each retry, a timeout of
five seconds is configured. However, there is also a total timeout of
10 seconds. So if the retries don't succeed after 10 seconds, the call
will fail. The Istio's default timeout is 15 seconds.

The rest of the file is not shown in the listing. It is very similar
and adds retries to the Istio gateway for the
order microservice. The Istio gateway routes user requests from
outside the Kubernetes cluster to the right microservice.

You can configure the order microservice to fail for 50% of all
requests with `kubectl apply -f failing-order-service.yaml`. 
This is implemented in the code and triggered by the instance variable
`FAILRANDOMLY`. It is set to `true` in the Kubernetes configuration.
If you
access the web page of the order microservice or make one of the other
microservices poll the latest orders, you will notice that about half
of the time the requests won't work.

You can configure Istio to retry the requests with `kubectl apply -f
retry.yaml`. The system should behave as if the microservice just works
normally. The retries are transparent. It is also possible to
change the settings for retries and timeout [per
request](https://istio.io/docs/concepts/traffic-management/#fine-tuning).

You can remove the retries with `kubectl delete -f retry.yaml`. The
failing microservice can be set to normal with `kubectl apply -f
microservices.yaml`.

#### Resilience: Impact on the Code

Istio's circuit breaker makes it more likely that a call to a
microservice fails. The same is true for a timeout. Retries can reduce
the number of failures. But even with Istio's resilience features,
there will still be calls to
microservices that fail. So while retry, timeout and circuit breaker
do not require any changes to the code, the code still needs to take
of care of failed requests. Those result in a response with HTTP
status code 5xx. How failures are handled, is a part of the domain
logic. For example if the warehouse management is down, it might not
be
possible to determine the availability of certain items. Whether an
order should be accepted even though the availability of the ordered
items is unknown, is a business decision.
Maybe it is fine to accept the order and deal with the unavailable
items. Maybe this would disappoint customers or violate contracts and
is therefore not an option.
So there must be code that
handled an HTTP status code 5xx and determines whether the order
should still be processed.

## 23.8 Challenges {#section-service-mesh-challenges}

Istio has a lot of advantages but also poses a few challenges.

#### Complex Infrastructure

Istio is a huge complex system. It adds a lot of
functionalities to Kubernetes. It covers monitoring, tracing, logging,
security and many more features. So it adds another set of technologies
to the stack that might be hard to understand in every
detail. However, Istio provides a complete solution for many
challenges of microservices systems. So while Istio might appear to be
complex, the question is whether there are any simpler
alternatives with the same set of features. If you run on Kubernetes
and end up building an Istio-like infrastructure yourself, you might
be building a solution that is even more complex and that you
need to maintain. It is possible to strip down the Istio
installation to the [bare
minimum](https://istio.io/docs/setup/kubernetes/minimal-install/) or
to exclude specific features that might not be needed.

#### Mixer and Proxy: Increased Latency 

The call to the proxy adds to the latency.
The call goes through the
loopback device. So it is not a true network call. It just
talks to the local machine with a virtual network round trip. 

For each call, Istio also adds a synchronous call to Mixer. This is
necessary to check the latest version of the policies against the
call. The call to Mixer adds to the latency. However, for a system with
asynchronous communication like in the example, this is not a huge
problem. Asynchronous call don't need to be handled in a timely manner
because no one waits for the results. For
synchronous systems, the latency might be a problem even without the
additional call to Mixer.

However, the creators of Istio are well aware of this
problem. Therefore, caching and other measures are implemented to make sure
that the call to Mixer is fast enough. For example, the metrics are
transferred asynchronously to limit how much data is transferred
synchronously.

The trace in [figure
23-6](#fig-service-mesh-tracing) includes the round trip to Mixer. In
that case it is neglectable. However, that might be different under
high load. Mixer's design is the result of [experiences at
Google](https://istio.io/blog/2017/mixer-spof-myth/). Mixer should
actually decrease latency and increase availability compared to a
service mesh without Mixer. That is because Mixer is designed with
high availability and low latency in mind. It shields the proxies from
dealing with the rest of the service mesh. However, a solution based
on a library such as Hystrix does not add any latency or availability
at
all. So from a performance perspective, a library might be better.

#### Focus on REST

Istio is only useful if most of the communication goes through the
proxies. This is the case for TCP communication. However, Istio
has specific support for HTTP, gRPC and WebSockets. For other traffic,
it cannot parse the protocol. Therefore, the proxy cannot determine
whether a request was handled successfully or not. As the example in
this chapter shows, some features are very useful if the system
is designed to do asynchronous communication via REST. For example,
tracing and the support for resilience are not that big an advantage
for any asynchronous systems. There are no huge call tree to be
traced. Resilience is already covered by the asynchronous
communication.

If the communication relies on a messaging technology like Kafka (see
[chapter 11](#chapter-kafka), a lot of communication happens through
the messaging system. So the Istio proxies
cannot understand it and handle it as well as REST calls. Istio might
still provide a few
advantages i.e. for web traffic from users. Prometheus and Grafana
can
show other metrics for example from the messaging system. But the
advantage is not as large as for a purely REST-based systems.

#### Limited Information

The service mesh uses proxy to collect information about the network
traffic between the microservices. This enables the service mesh to
work independently from the technologies inside the pods. However, it
also limits the information available to the service mesh. For
monitoring, the information about the requests and the Kubernetes
infrastructure might be enough. For tracing, the microservices 
have to forward some HTTP headers. So tracing is not fully transparent
for the microservices. For logging Istio can just generate a log entry
for each HTTP request. For resilience, circuit breaker, timeout and
retries leave not a lot to be desired. Still the business logic must
decide what should happen if a service is not available.

Of course it is possible to make the microservice log more information
or provide more metrics.
But in that case, the microservices would need to be
modified.
So if you don't want to change the microservices, Istio provides
limited information that might be enough for some areas like
monitoring but might lack in other areas like logging.

## 23.9 Benefits {#section-service-mesh-benefits}

Istio solves a lot of challenges of microservice architectures. It covers
resilience, tracing, monitoring, security and logging. All of these
challenges are solved independently from the programming language or 
frameworks. So Istio does not limit the technology freedom. This
is clearly preferably over the other alternatives presented in this
book. Those alternatives limit the choice of technology or at least
have to
be supported differently for each used programming language.

Also Istio works well with Kubernetes (see [chapter
17](#chapter-kubernetes)) that complements Istio  by
solving deployment, clustering. load balancing and service
discovery. Compared to a Kubernetes cluster without Istio, not a lot
changes as far as deployment and configuration of the microservices is
concerned.

Istio's architecture and implementation is based on Google's
experience from operating its huge infrastructure. So while it is a
relatively new project, the concepts and previous implementations
have proven to be successful in a very huge and complex environment.
Also Istio uses technologies like Prometheus or Grafana etc that are
quite mature.

Service meshes are gaining a lot of popularity. Istio is probably the
most popular implementation. A popular technology is usually a
technology that has little risk because the community presents a huge
market and will always aim at keeping the technology alive and
viable. It is supported by Google and IBM, two of the leading
companies in the IT area.

## 23.10 Variations {#section-service-mesh-variations}

There are alternatives to Istio on Kubernetes:

* Istio can also run with [Docker and
Consul](https://istio.io/docs/setup/consul/). So even if you are not
using Kubernetes, you can still use Istio's features. See [chapter
15](#chapter-consul) for more information about Consul.

* [Linkerd](https://linkerd.io/) is another service mesh. It runs on
  Kubernetes and is part of the portfolio of the [Cloud Naive
  Computing Platform](https://cncf.io/). 

* [Aspen
  Mesh](https://aspenmesh.io/) is a distribution of
  Istio. 

* Cloud providers also offer service meshes. Azure has the [Service
  Fabric
  Mesh](https://docs.microsoft.com/en-us/azure/service-fabric-mesh/service-fabric-mesh-overview)
  and Amazon Web Services the [AWS App
  Mesh](https://aws.amazon.com/app-mesh/). Google provides support for
  [Istio in the Google Cloud](https://cloud.google.com/istio/).


## 23.11 Experiments

Istio provides an extensive documentation. You can use it to
familiarize yourself with features of Istio that this chapter does not
discuss: 

- Istio provides support for security. See [the security
  task](https://istio.io/docs/tasks/security/) for some hands-on
  exercises
  for this features. The exercises cover encrypted communication,
  authentication and authorization.

- Istio also supports advanced routing. For example, [the traffic
  shifting
  task](https://istio.io/docs/tasks/traffic-management/traffic-shifting/) 
  shows how to do A/B testing with Istio. 
  The [mirroring
  task](https://istio.io/docs/tasks/traffic-management/mirroring/)
  shows
  mirroring. With mirroring, two versions of the microservice receive
  the traffic. Mirroring can be used to make sure that the new and the
  old version behave in the same way.
  
- Istio can also create graphs of the [dependencies between the
  microservices](https://istio.io/docs/tasks/telemetry/servicegraph/). Istio
  even provides a specific tool to visualize the service mesh called
	  Kiali, see the [Kiali
  task](https://istio.io/docs/tasks/telemetry/kiali/). 

## 23.12 Conclusion

Service meshes solve many challenges for microservices systems. They
support monitoring, tracing, logging and resilience. While they
provide a lot of features, service meshes
do not limit the free technology choice for implementing
microservices. As the example in this chapter shows, this works quite
well for
monitoring. For tracing the microservices need to be modified. 
For logging Istio is of little use. Besides providing the information,
Istio also includes tools like Prometheus, Grafana, Jaeger or Kiali to
visualize and store information about the microservice system. So Istio
is a pretty complete
solution for the operation of microservices.

Istio's concepts are based on Google's experience with large container
systems. So the idea of a service mesh has proven itself. Istio
integrates most of the
technologies and only adds some glue. So it is actually not a very
risky technology. The components
Istio uses are mature and used in a lot of systems.

However, Istio is not just powerful but also complex. To solve this,
Istio can be
stripped down. It is possible to exclude each part of the system from
the installation. The question is whether there is any simpler
alternative. The challenges Istio solves must be dealt with in a
microservices system. Istio's complexity is probably not a problem of
Istio but rather of microservices.

Istio adds an overhead to the communication between
microservices. That might impact performance but is probably
acceptable. Microservices communicate through the network
anyway. Adding an additional, relatively small overhead to that
communication is
not a fundamental change.

#### Advantages

* Complete solution for the main challenges of microservice
* No impact on the technology used by the microservice
* No changes to the code of the microservices necessary
* Concept has proved itself at Google
* Good integration with Kubernetes

#### Challenges

* Huge and complex
* Adds overhead to the communication between the microservice

# 17 Recipe: Docker Containers with Kubernetes {#chapter-kubernetes}

This chapter describes Kubernetes, a runtime environment for Docker
containers. The reader gets to know the following points in this
chapter:

* Kubernetes can run Docker containers in a cluster and comprises a
complete infrastructure for microservices.

* Kubernetes does not introduce code dependencies into the example.

* Also MOMs or other tools can be run on Kubernetes.

## 17.1 Kubernetes {#section-kubernetes-kubernetes}

[Kubernetes](https://kubernetes.io/) is increasingly gaining
importance as runtime environment for the development and the
operation of microservices.

#### Licence and Community

Kubernetes is an open source project and under the Apache licence. It
is managed by the Linux foundation and was originally created at Google.
An extensive ecosystem has emerged around Kubernetes, offering various
extensions.

#### Kubernetes Versions

There are different Kubernetes variants.

[Minikube](https://github.com/kubernetes/minikube) is a version of
Kubernetes for installing a test and development system on a laptop. But
there are many more
[versions](https://kubernetes.io/docs/getting-started-guides/), which
can either be installed on servers or directly used as cloud offerings.
[kops](https://github.com/kubernetes/kops) is a tool which enables the
installation of a Kubernetes cluster in many different types of
environments like AWS (Amazon Web Services).
[Amazon Elastic Container Service for Kubernetes (Amazon EKS)](https://aws.amazon.com/eks/)
provides Kubernetes clusters on AWS. Google Cloud also supports
Kubernetes with the
[Google Container Engine](https://cloud.google.com/container-engine).
Microsoft Azure provides the
[Azure Container Service](https://azure.microsoft.com/en-us/services/container-service/)
and IBM Bluemix the
[IBM Bluemix Container Service](https://console.ng.bluemix.net/docs/containers/container_index.html).

#### Features

As mentioned above, Kubernetes offers a platform based on Docker that
supports some important features such as:

* Kubernetes runs Docker containers in
a *cluster* of nodes. Thus, Docker containers can use all resources in
the cluster.

* In the event of a failure, Docker containers can be restarted. This
is even possible if the original node on which the container was
running is no longer available. In this way, the system
achieves *fail-safety*.

* Kubernetes also supports *load balancing* and can distribute the load
between multiple nodes.

* Finally, Kubernetes supports *service discovery*. Microservices that
run in Docker containers can easily find each other and communicate
with each other via Kubernetes.

* Since Kubernetes works on the level of Docker containers, the
microservices have *no code dependencies* to Kubernetes. This is
not only elegant, but also means that a Kubernetes system supports
virtually all programming languages and frameworks for the
implementation of microservices.

#### Kubernetes Concepts

In addition to the already discussed Docker concepts (see
[chapter 5](#chapter-docker)) Kubernetes introduces several additional
concepts.

* *Nodes* are the servers on which Kubernetes runs. They are organized
in a cluster.

* A *Pod* is multiple Docker containers which together provide a
service. For example, this can be a container with a microservice
together with a container for log processing. The Docker containers in
a pod can share Docker volumes and thus efficiently exchange data.
All containers belonging to one pod always run on one node.
So to scale the system to more nodes, more instances of the pod need
to be started and distributed on the nodes.

* A *replica set* ensures that always a certain number of instances of
a pod runs. This allows the load to be distributed to the pods. In
addition, the system is fail-safe. If a pod fails, a new pod is
automatically started.

* A *deployment* generates a replica set and provides the required
Docker images.

* *Services* make pods accessible. The services are registered under one
name in the DNS and have a fixed IP address under which they can be
contacted throughout the cluster. In addition, services enable routing
for requests from the outside.

* Kubernetes is *declarative*. I.e. the configuration defines a
  desired state. Kubernetes makes sure that the system fits the
  desired state. So a replica set actually defines the number of pods
  that should be running and it is up to Kubernetes to make the
  actually number of running pods match that number. If the
  configuration of the replica set changes, the number of running pods
  is adjusted accordingly. And if some of the pods crash, enough pods
  are started to match the declaration in the replica set.

## 17.2 The Example with Kubernetes {#section-kubernetes-beispiel}

The services in this example are identical with the examples presented
in the two preceding chapters (see
[section 14.1](#section-netflix-beispiel)).

{id="fig-kubernetes-beispiel"}
![Fig. 17-1: The Microservices System in Kubernetes](images/kubernetes-beispiel.png)

* The *catalog* microservice manages the information about the
items. It provides an HTML UI and a REST interface.

* The *customer* microservice stores the customer data and also
provides an HTML UI and a REST interface.

* The *order* microservice can receive new orders. It provides an HTML
UI and uses the REST interfaces of the catalog and customer
microservice.

* There is also an *Apache web server* for facilitating access to the
individual microservices. It forwards the
calls to the respective services.

[Figure 17-1](#fig-kubernetes-beispiel) shows how the microservices
interact. The order microservice communicates with the catalog and the
customer microservice. The Apache httpd server communicates with all
other microservices to display the HTML
UIs.

In addition, the microservices are accessible from the outside via
node ports. On each node in the cluster request to a specific port
will be forwarded to the service. However, the port numbers are
assigned by Kubernetes. So there is no port number in the figure. Also
a load balancer is set up by the Kubernetes service to distribute the
load across Kubernetes nodes.

#### Implementing Microservices with Kubernetes

{id="fig-kubernetes-beispiel-microservice"}
![Fig. 17-2: A Microservice in Kubernetes](images/kubernetes-beispiel-microservice.png)

[Figure 17-2](#fig-kubernetes-beispiel-microservice) shows the
interaction of the Kubernetes components for a microservice.
A *deployment* creates a *replica set* with the help of Docker images.
The *replica set* starts one or multiple pods. The *pods* in the
example only comprise a single Docker container in which the
microservice is running.

#### Service Discovery

A *service* makes the replica set accessible. The service provides the
pods with an IP address and a DNS record. Other pods communicate with
the service by reading the IP address from the DNS. Thereby,
Kubernetes implements service discovery with DNS. In addition,
microservices receive the IP addresses of other microservices via
environment variables. Thus, they could also use this information to
access the service.

#### Fail-Safety

The microservices are so fail-safe because the replica set ensures
that a certain number of pods is always running. So if a pod fails, a
new one is started.

#### Load Balancing

Load balancing is also covered. The number of pods is determined by
the replica set. The service implements load balancing. All pods can
be accessed with the same IP address which the service defines.
Requests go to this IP address, but are distributed to all instances.
The service implements this functionality by interfering with the IP
network between the Docker containers. Since the IP address is
cluster-wide unique, this mechanism works even if the pod is moved
from one node to another.

Kubernetes does not implement load balancing at the DNS level. If it
did, a different IP addresses would be returned for the same service
name for each DNS lookup so that the load would be distributed during
DNS access. However, such an approach presents a number of challenges.
DNS supports caching. If a different IP address is to be returned each
time a DNS access occurs, the caching must be configured accordingly.
However, often problems still occur because caches are not invalidated
in time.

#### Service Discovery, Fail-Safety, and Load Balancing without Code Dependencies

For load balancing and service discovery no special code is necessary.
A URL like http://order:8080/ suffices. Accordingly, the microservices
do not use any special Kubernetes APIs.
There are no code dependencies to Kubernetes, and no specific
Kubernetes libraries are used.

#### Routing with Apache httpd

In the example, Apache httpd is configured as a reverse proxy. It
therefore routes access from the outside to the correct microservice.
Apache httpd leaves load balancing, service discovery, and fail-safety
to the Kubernetes infrastructure.

#### Routing with Node Ports

Services also offer a solution for routing, i. e. external access to
microservices. The service generates a node port. Under this port the
services are available on every Kubernetes node. If the pod that
implements the service is not available on the called Kubernetes node,
Kubernetes forwards a request to the node port to another Kubernetes
node where the pod is running. In this way, an external load
balancer can distribute the load to the nodes in the Kubernetes
cluster. The requests are simply distributed to the node port of the
service on all nodes in the 
cluster. The service type `NodePort`
is used for this purpose, which must be specified when creating the
service.

#### Routing with Load Balancers

Kubernetes can create load balancers in a Kubernetes production
environment. In an Amazon environment, for example, Kubernetes
configures an ELB (Elastic Load Balancer) to access the node ports in
the cluster. The service type `LoadBalancer` is used for this purpose.

{id="fig-kubernetes-routing"}
![Fig. 17-3: Kubernetes Service Type LoadBalancer in der Amazon Cloud](images/kubernetes-routing.png)

The services in the example are of the type `LoadBalancer`. However,
if they run on Minikube, they are treated as services of type
`NodePort` because Minikube cannot configure and provide load
balancers.

#### Routing with Ingress

Kubernetes offers an extension called
[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/),
which can configure and alter the acesss of services from the
Internet. Ingress can also implement load balancing, terminate SSL or
implement virtual hosts. This behavior is implemented by an Ingress
controller.

In the example, the Apache web server forwards requests to the
individual microservices. This could also be done with a Kubernetes
Ingress. Then the routing would be done by a part of the Kubernetes
infrastructure which might work better e.g. in the cluster and is
easier to configure. The configuration is done in Kubernetes YAML
files and supports other Kubernetes parts like e.g. services. However,
the example contains a few static web pages that the Apache web server
provides. So the Apache web server has to be configured anyways. Using
an Ingress is therefore not a huge advantage.

## 17.3 The Example in Detail {#section-kubernetes-detail}

[Section 0.4](#section-einleitung-quick-start) describes which
software has to be installed for starting the example.

The example is accessible at
<https://github.com/ewolff/microservice-kubernetes>. <https://github.com/ewolff/microservice-kubernetes/blob/master/HOW-TO-RUN.md>
explains in detail how to install the required software for running the example.

The following steps are necessary for running the example:

* [Minikube](https://github.com/kubernetes/minikube) as minimal
Kubernetes installation has to be installed. Instructions for this can
be found at <https://github.com/kubernetes/minikube#installation>.

* [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/)
is a command line tool for handling Kubernetes and also has to be
installed. Its installation is described at
<https://kubernetes.io/docs/tasks/tools/install-kubectl/>.

* The script `docker-build.sh` generates the Docker images for
  the microservices and uploads them into the public Docker hub. This
  step is optional since the images are already available on the
  Docker hub. It only has to be performed when changes were introduced
  to the code or to the configuration of the microservices.
  Before starting the script, the Java code has to be compiled with
 `./mvnw clean package` (macOS, Linux) or `mvnw.cmd clean package`
(Windows) in directory `microservice-kubernetes-demo`.
  See [appendix B](#anhang-maven) for more details on Maven and how to
  trouble shoot the build.
  Then the script `docker-build.sh` creates the images with `docker
  build` , with `docker tag` they receive a globally unique name and
  `docker push` uploads them into the Docker hub. Using the public
  Docker hubs spares the installation of a Docker
  repository and thereby facilitates the handling of the example.

* The script `kubernets-deploy.sh` deploys the images from the public Docker hub in the 
  Kubernetes cluster and thereby generates the pods, the deployments, the
  replica sets, and the services. For this, the script uses the tool
`kubectl`. `kubectl run` serves for starting the image. The image is
downloaded at the indicated URL in the Docker hub. In addition, it is
defined which port the Docker container should provide. So `kubectl
run` generates the deployment, which creates the replica set and
thereby the pods. `kubectl expose` generates the service which
accesses the replica set and thus creates the IP address, node port
resp. load balancer and DNS entry.

This excerpt from `kubernetes-deploy.sh` shows the use of the tools
using the catalog microservice as example.

{linenos=off}
~~~~~~~~
#!/bin/sh
if [ -z "$DOCKER_ACCOUNT" ]; then
  DOCKER_ACCOUNT=ewolff
fi;
...
kubectl run catalog \\
 --image=docker.io/$DOCKER_ACCOUNT/microservice-kubernetes-demo-catalog:latest
  \\
 --port=80
kubectl expose deployment/catalog --type="LoadBalancer" --port 80
...
~~~~~~~~

An alternative is to use Kubernetes YAML files. They describe the
desired state of deployments and services. For example here is the
part of `microservices.yaml` for the catalog microservices.

{linenos=off, lang="yaml"}
~~~~~~~~
...

apiVersion: apps/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: catalog
  name: catalog
spec:
  replicas: 1
  selector:
    matchLabels:
      run: catalog
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: catalog
    spec:
      containers:
      - image: docker.io/ewolff/microservice-kubernetes-demo-catalog:latest
        name: catalog
        ports:
        - containerPort: 8080
        resources: {}
status: {}

---

...

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: catalog
  name: catalog
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    run: catalog
  type: LoadBalancer
status:
  loadBalancer: {}

...
~~~~~~~~

The information in the YAML file is very similar to the parameters of
the commands above. Using `kubectl apply -f microservices.yaml` all
the services and deployment would be created in the Kubernetes
cluster. The same command would be used to update the services and
deployments after any changes.

#### Some Minikube Commands

`minikube dashboard` displays the dashboard in the web browser, which
displays the deployments and additional elements of Kubernetes. This
makes it easy to understand the state of the services and deployments
([Figure 17-4](#fig-kubernetes-dashboard)).

{id="fig-kubernetes-dashboard"}
![Fig. 17-4: Kubernetes Dashboard](images/kubernetes-dashboard.png)

`minikube service apache` opens the Apache service in the web browser
and thereby offers access to the microservices in the Kubernetes
environment.

The script `kubernetes-remove.sh` can be used to delete the example.
It uses `kubectl delete service` for deleting the services, and
`kubectl delete deployments` for deleting the deployments.

## 17.4 Additional Kubernetes Features {#section-kubernetes-features}

Kubernetes is a powerful technology with many features. Here are some
examples of additional Kubernetes features:

#### Monitoring with Liveness and Readiness Probes

* Kubernetes recognizes the failure of a pod via
  [Liveness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/).
  A custom Liveness Probe can be used to determine when a container is
started anew depending on the needs of the application.

* A
[Readiness Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-readiness-probes)
indicates whether a container can process requests or not. For
example, if the application is blocked by processing a large amount of
data or has not yet started completely, the Readiness Probe can report
this state to Kubernetes. In contrast to a Liveness Probe, the
container is not restarted as a result of a failed Readiness Probe. 
Kubernetes assumes that after some time the pod will signal via the
Readiness Probe that it can handle requests. 

#### Configuration

* The configuration of applications is possible with
  [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configmap/).
  The configuration data is provided to the applications for instance
  as values in environment variables.

#### Separating Kubernetes Environments with Namespaces

* Kubernetes environments can be separated with
[Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).
Namespaces are virtual clusters so that e.g. services and deployments are
completely separated.
That allows different environment to coexist. I.e. it is possible for
multiple teams to share a cluster and use namespaces to separate their
environments. Also this makes it possible to separate the
microservices from infrastructure like databases or monitoring
infrastructure. That way users only see the services and deployments
they are interested in.

#### Applications with State

* Kubernetes can also handle applications that have state.
Applications without state can be simply restarted on another node in
a cluster. This facilitates fail-safety and load balancing. If the
application has state and therefore requires certain data in a Docker
volume, the required Docker volumes must be available on each node the
application runs on. This makes the handling of such applications more
complex. Kubernetes offers
[persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
and
[stateful sets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#stable-storage)
for dealing with this challenge.

* Another option for dealing with applications with state are
  [operators](https://coreos.com/operators). They allow the automated installation of applications with state. For example, there is the 
  [Prometheus operator](https://github.com/coreos/prometheus-operator),
  which installs the monitoring system Prometheus
  ([chapter 20](#chapter-monitoring)) in a Kubernetes cluster. It
introduces Kubernetes resources for Prometheus components such as
`Prometheus`, `ServiceMonitor`, or
  `Altermanager`. With the Prometheus operator, these key words are
used by the  Kubernetes configuration instead of pods, services, or
deployments. The operator also determines how Prometheus saves the
monitoring data, and thus solves a key challenge. 

#### Extensions with Helm

* Kubernetes offers a complex ecosystem with numerous extensions.
[Helm](https://helm.sh/) offers the possibility to
install extensions as [Charts](https://github.com/kubernetes/charts/)
and thereby assumes the functionality of a package manager for
Kubernetes. This extensibility is an important advantage of
Kubernetes.

[Section 23.2](#section-service-mesh-example) shows how Helm can be
used to simplify the deployment of a microservice. Using a Helm chart
just the name of the microservice must be defined. The Kubernetes
configuration is generated with a template and the provided name. This
makes the installation of the microservices much easier and more
uniform.

## 17.5 Recipe Variations {#section-kubernetes-variationen}

Kubernetes offers above all a runtime environment for Docker
containers and is therefore very flexible.

#### MOMs in Kubernetes

The example in this chapter uses communication with REST. Of course,
it is also possible to operate a MOM like Kafka
([section 11](#chapter-kafka)) in Kubernetes. However, MOMs store the
transmitted messages to guarantee delivery. Kafka even saves the
complete history. Reliable storage of data in a Kubernetes cluster is
feasible, but not easy. Using a MOM other than Kafka does not solve
the problem. All MOMs store messages permanently to guarantee
delivery. Thus, for reliable communication with a MOM, Kubernetes has
to store the data reliably and scalability.

#### Frontend Integration with Kubernetes

Kubernetes can be quite easily combined with frontend integration
([chapter 7](#chapter-frontend)), since Kubernetes does not make any
assumptions about the UI of the applications. Client-side frontend
integration does not place any demands on the backend. For server-side
integration, a cache or web server must be hosted in a Docker
container. However, these servers do not store any data permanently,
so they can easily be operated in Kubernetes.

#### Docker Swarm and Docker Compose

[Section 4.7](#section-docker-variationen) already introduced some
alternatives to Kubernetes for operating Docker containers in
clusters. Kubernetes offers a very powerful solution and is further
developed by many companies in the container area. However, Kubernetes
is also very complex due to its many features. A cluster with Docker
Compose and Docker Swarm can be a simpler but also less powerful
alternative. However, Docker Swarm and Compose at least also offer
basic features like service
discovery and load balancing.

#### Docker vs. Virtualization

As Kubernetes takes over cluster management, it includes features that
virtualization solutions also offer. This can also lead to operational
concerns, as reliable cluster operation is a challenge. Another
technology in this area is often viewed critically. When deciding
against Kubernetes, Docker can still be used without a scheduler (see
[section 4.7](#section-docker-variationen)). But then the Kubernetes
features for service discovery, load balancing, and routing are
missing.
They probably have to be implemented by different means.

## 17.6 Experiments {#section-kubernetes-experimente}

* Supplement the Kubernetes system with an additional microservice.
  * A microservice that is used by a call center agent to create
  notes for a call can be used as an example. The call center agent
  should be able to select the customer. 
  * For calling the customer microservice the host name
    `customer` has to be used.
  * Of course, you can copy and modify one of the existing
  microservices. 
  * Package the microservice in a Docker image and upload it into the
  Docker repository. This can be done by adapting the script
  `docker-build.sh`. 
  * Adapt `kubernetes-deploy.sh` in such a manner that the
  microservice is deployed. 
  * Adapt `kubernetes-remove.sh` in such a manner that the
  microservice is deleted. 

* <https://kubernetes.io/docs/getting-started-guides/> is an
interactive tutorial which shows how to use Kubernetes. It complements
this chapter well. Work through the tutorial to get an impression of
the Kubernetes features. 

* Kubernetes supports rolling updates. A new version of a pod is rolled out in such a way that there are no interruptions to the service. See
  <https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/>.
  Run a rolling update! To do this you need to create a new Docker
  image. The
  scripts for compiling and delivering to the Docker hub are included
  in the example. 

* Cloud providers such as Google or Microsoft offer
  Kubernetes infrastructures, see
  <https://kubernetes.io/docs/getting-started-guides/#hosted-solutions>.
  Make the example work in such an environment! The scripts can be
  used without changes because `kubectl` also supports these
  technologies. 

* Test the load balancing in the example.
  * `kubectl scale` alters the number of pods in a replica set.
  `kubectl scale -h` indicates which options there are.
  For example, scale the replica set `catalog`. 
  * `kubectl get deployments` shows how many pods are running in the respective
    deployment. 
  * Use the service. For example `minikube service
  apache` opens the web page with links to all microservices.
  Select the order microservice and get the orders displayed.
  * `kubectl describe pods -l run=catalog` displays the running pods.
  There you can also find the IP address of the pods in a line which
  starts with `IP`. 
  * Log into the Kubernetes node with `minikube ssh`. To read out the
  metrics you can then use a command like
  `curl 172.17.0.8:8080/metrics`. You have to adapt the IP
  address. This way you can display the metrics of the catalog pods
which Spring Boot creates. For example, the metrics contain the number
of requests that have been responded with an HTTP 200 status code
(OK). If you now
use the catalog microservice via the web page, each pod should process
some of the requests and therefore the metrics of all pods should go
up.
  * Also use `minikube dashboard` for observing the information in the
  dashboard. 

* The example currently uses the public Docker hub. Install your own
  [Docker registry](https://docs.docker.com/registry/). Save the
  Docker images of the example in the registry and deploy the example
  from this registry.

* At <https://github.com/GoogleCloudPlatform/kubernetes-workshops>
  you can find material for a Kubernetes workshop to further
  familiarize yourself with this system.

* Port the Kafka example (see [chapter 11](#chapter-kafka)) or
  the Atom example (see [chapter 12](#chapter-atom)) to
  Kubernetes. This shows how asynchronous microservices can also run
  in a Kubernetes cluster. Kafka stores data, which can be difficult
in a Kubernetes system. Explore how to run a Kafka cluster in a
  Kubernetes system in production.

* Use `kubectl logs -help` for familiarizing yourself with the log
administration in Kubernetes. Take a look at the logs of at least two
microservices.

* Use [kail](https://github.com/boz/kail) for displaying the logs of
some pods.

## 17.7 Conclusion {#section-kubernetes-fazit}

Kubernetes solves the challenges of synchronous microservices as follows.

* DNS offers *service discovery*. Thanks to DNS, microservices can be
used from any programming language and even transparently. However,
DNS only provides the IP address
so the port must be known. No code is required
to register the services. When you start the service, a DNS record is
created automatically.

* *Load balancing* is ensured by Kubernetes by distributing the
traffic for the IP address of the Kubernetes service to the individuals
pods on the IP level. This is transparent for callers and for the
called microservice.

* *Routing* is covered by Kubernetes via the load balancer or node
ports of the services. This is also transparent for the microservices.

* *Resilience* is offered by Kubernetes via the restarting of
containers and load balancing. In addition, a library such as Hystrix
can be useful, for instance, for implementing timeouts or circuit
breakers. A proxy like [Envoy](https://github.com/lyft/envoy) can be
an alternative to Hystrix.
Envoy is also part of Istio (see [section
23.3](#section-service-mesh-how) and implements resilience for Istio.

Within one package, Kubernetes offers complete support for
microservices including service discovery, load balancing, resilience,
and scalability in the cluster. In this way, Kubernetes solves many
challenges arising during the operation of a microservices
environment. The code of the microservices remains free of these
concerns. No dependencies on Kubernetes are introduced into the code.

This is attractive, but it also represents a fundamental change. While
Consul or the Netflix stack also run on virtual machines or even bare metal, Kubernetes
requires everything to be packed in Docker containers. This can be a
fundamental change compared to an existing mode of operation and can
make the migration to this environment harder.

#### Advantages

* Kubernetes solves most typical challenges of microservices (load balancing, routing,
  service discovery).

* The code has no dependencies on Kubernetes.

* Kubernetes also covers operation and deployment.

* The Kubernetes platform enforces standards and thereby the
  definition of a macro architecture.

#### Challenges

* A complete change of operation is required to use Kubernetes
  instead of other log or deployment technologies.

* Kubernetes is very powerful, but thus also very complex.

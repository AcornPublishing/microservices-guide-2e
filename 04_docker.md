# 4 Docker {#chapter-docker}

This chapter provides an introduction to Docker and covers
the following topics:

* After studying the chapter, the reader is able to run the examples
provided in the following chapters in a Docker environment.

* Docker and microservices are nearly synonymous. This chapter
explains why Docker fits so well to microservices.

* Docker facilitates software installation. Important for this is the
`Dockerfile`, which describes the installation of the software in a
simple way.

* Docker Machine and Docker Compose support Docker on server systems
and complex software environments with Docker.

* The chapter lays the foundation for an understanding of technologies
  such as Kubernetes ([chapter 17](#chapter-kubernetes)) and Cloud
Foundry ([chapter 18](#chapter-paas)) which are based on Docker.

#### Licences and Projects

Docker is under the Apache 2.0 licence. It is developed by the company
[Docker, Inc](https://www.docker.com/), among others. Some core
components such as for instance [Moby](https://github.com/moby/moby)
are under an Open Source licence and thus allow also other developers
to implement systems similar to Docker. Docker is based on Linux
containers which isolate processes in Linux systems from each
other. The
[Open Container Initiative](https://www.opencontainers.org/) ensures
via standardization the compatibility of the different container
systems.

## 4.1 Docker for Microservices: Reasons {#section-docker-gruende}

[Chapter 1](#chapter-microservices) defined microservices as
separately deployable units. The separate deployment does not only
result in a decoupling at the architectural level, but also in regards
to technology choice, robustness, security, and scalability.

#### OS Processes for Microservices?

If microservices are supposed to have all these characteristics, the question
arises how they can be implemented. Microservices must be scalable
independently of each other. In the event of a crash, a microservice
must not be allowed to make other microservices unavailable, too, and
thus endanger the robustness of the whole system. Therefore,
microservices must at least be separate processes.

*Scalability* can be guaranteed by multiple instances of a process. When
an application is started, the operating system generates a process
and allocates resources such as CPU or memory to it. So more processes
can use more resources.
But processes are limited concerning
scaling. If the processes all run on one server, then only a limited
amount of hardware resources are available.
Instead the microservices should run in a cluster. 
Kubernetes ([chapter 17](#chapter-kubernetes)) and Cloud
Foundry ([chapter 18](#chapter-paas)) support running microservices in
a cluster.

With processes, *robustness* is guaranteed to a certain extent because
the crash of one process does not affect the other processes. However,
a server failure still causes a large number of processes and thus
microservices to fail. But there are also other problems. All
processes share one operating system. It must provide the libraries
and tools for all microservices. Each microservice must be compatible
with the operating system version. It is difficult to configure the
operating system to support all microservices. In addition, the processes
must coordinate in such a way that each process has its own network
port. If there is a large number of processes, it becomes increasingly
harder to find unused ports. Also it hard to figure out which ports
are used by which process.

#### Virtual Machines: Too Heavy-Weight for Microservices

Instead of a process, each microservice can run in its own virtual
machine. Virtual machines are simulated computers that all run on the
same physical hardware. For the operating system and application,
virtual machines look exactly like a physical server. Through
virtualization, the microservice has its own operating system
installation. Thus, the configuration of the operating system can be
adapted to the specific microservice and there is also complete
freedom in choosing the network port.

However, a virtual machine has a substantial overhead:

* The virtual machine must give the operating system the illusion of
running directly on the hardware. This leads to an overhead.
Therefore, *performance* is poorer than with physical hardware.

* Each microservice has its own instance of the operating system. This
consumes a lot of memory in the *RAM*.

* Finally, the virtual machine has virtual disks with a complete
operating system installation. This means that the microservice
occupies a lot of *hard disk space*.

So virtual machines have an overhead. This makes their operation
expensive. In addition, operations has to manage a large number of
virtual servers. This is complicated and time-consuming.

The ideal solution would be a light-weight alternative to
virtualization, which possesses the isolation of virtual machines, but
consumes as little resources as processes do and is similarly easy to
operate.

## 4.2 Docker Basics {#section-docker-grundlagen}

Docker represents a light-weight alternative to virtualization. While
Docker does not provide as much isolation as a virtualization, it is
practically as light-weight as a process.

* Instead of having a complete virtual machine of their own, Docker
containers *share* the *kernel* of the operating system on the Docker
host. The Docker host is the system on which the Docker containers
run. The processes from the containers therefore appear in the process
table of the operating system on which the Docker containers are
running.

* The Docker containers have their *own network interface*. In this
way, the same port can be used in each Docker container, and each
container can use any number of ports. The network interface is in a
subnet where all Docker containers are accessible. The subnet is not
accessible from the outside. This is at least the standard
configuration of Docker. The Docker network configuration offers many
other alternatives. To still allow external access to a Docker
container from the outside, ports of a Docker container can be
mapped to ports on the Docker host. When mapping the ports of the
Docker containers to the ports of the Docker host, be careful because
each port of the Docker host can only be mapped to one port of a Docker
container.

{id="fig-docker-dateisysteme", width=75%}
![Fig. 4-1: File System Layers in Docker](images/docker-dateisysteme.png)

* Finally, the file system is optimized. There are *layers in the file
system*. When a microservice reads a file, it goes through the layers
from top to bottom until it finds the data. The containers can share
layers. [Figure 4-1](#fig-docker-dateisysteme) shows this more
precisely. The file system layer at the bottom represents a simple
Linux installation with the Alpine Linux distribution. Another layer
is the Java installation. Both applications share these two layers
which are stored only once on the hard disk, although both
microservices use them. Only the applications are stored in file
system layers that are exclusively available to a single container.
The lower layers cannot be changed. The microservices can only write
to the top layer. The reuse of the layers reduces the storage
requirements of the Docker containers.

It is easily possible to start hundreds of containers on a laptop.
This is not surprising. After all, it is also possible to start
hundreds of processes on a laptop. Docker has no significant overhead
compared to a process. Compared to virtual machines, however, the
performance benefits are outstanding.

#### One Process per Container

Ultimately, Docker containers are highly isolated processes due to
their own network interface and file system. Therefore, only one
process should run in a Docker container. Running more than one
process in a Docker container contradicts the idea of separating
processes from each other by means of Docker containers. Because only
one process is supposed to run in a Docker container, there should be
no background services or daemons in Docker containers.

#### Docker Image and Docker Registry

File systems of Docker containers can be exported as Docker images.
These images can be passed on as files or can be stored in a Docker
registry. Many repositories such as
[Nexus](https://www.sonatype.com/nexus-repository-sonatype) and
[Artifactory](https://www.jfrog.com/open-source/#artifactory) can store
and provide Docker images just like compiled software and libraries.
This makes it easy to exchange Docker images with a Docker registry
for installation in production. The transfer of images from and to the
registry is optimized. Only the updated layers are transferred.

#### Supported Operating Systems

Docker is originally a Linux technology. For operating systems such as
macOS and Windows, Docker installations are available that allow Linux
Docker containers to be started. For this purpose, a virtual machine
with a Linux installation is running in the background. This is
transparent for the user. It seems as if the Docker containers are
running directly on a computer.

For Windows, there are in addition Windows Docker containers since
Windows Server 2016. Linux
applications run in a Linux Docker container, Windows applications in
a Windows Docker container.

#### Operating Systems for Docker

Docker changes the requirements for operating systems.

* Only one process is supposed to run in one *Docker container*. This
means that only as much of the operating system is required as is
needed to run that process. For a Java application, this is the Java
Virtual Machine (JVM), which requires some Linux libraries that are
loaded at runtime. A shell is not necessary, for example. Therefore,
there are distributions like [Alpine Linux](https://alpinelinux.org/),
which are just a few megabytes in size and only contain the most
important tools. They are an ideal basis for Docker containers. The
programming language Go can create statically linked programs. In that
case nothing else has to be available in the Docker container besides
the program itself. Then no Linux distribution is required at all.
  
* The *Docker host* on which the Docker containers run only has to run
Docker containers. Many Linux tools are therefore superfluous.
[CoreOS](https://coreos.com/) is a Linux distribution that can run
little more than Docker containers and for example considerably
simplifies operating system updates of an entire cluster. CoreOS can
also serve as a basis for Kubernetes (see also
[chapter 17](#chapter-kubernetes)). Another example is
[boot2docker](http://boot2docker.io/), which Docker Machine installs 
(see [section 4.4](#section-docker-machine)) on servers to run Docker
containers on these servers. Also this Linux distribution like CoreOS
can basically only run Docker containers.

#### Overview

{id="fig-docker-ueberblick"}
![Fig. 4-2: Overview of Docker](images/docker-ueberblick.png)

[Figure 4-2](#fig-docker-ueberblick) shows an overview of the Docker
concepts.

* The *Docker host* is the machine on which the Docker containers run.
It can be a virtual machine or a physical machine.

* *Docker containers* run on the Docker host.

* The containers typically contain a *process*.

* Each container has its own *network interface* with its own IP
address. This network interface is only accessible from the Docker
internal network. However, there are also ways to allow access from
outside this network.

* In addition, each container has its own *file system* (see
[figure 5-1](#fig-docker-dateisysteme)).

* When a container is started, the *Docker image* creates the first
version of the Docker file system. When the container has been
started, the image is extended by another layer into which the
container can write its own data.

* All Docker containers share the *kernel* of the Docker host.

#### Does It Always Have to be Docker?

Docker is a very popular option for deploying microservices. However,
there are alternatives. Two alternatives were already mentioned in
[section 4.1](#section-docker-gruende): virtual machines or processes.

#### Microservices as WARs in Java Application Servers

However, it is also conceivable to deploy several microservices as WAR
files in a Java application server or Java web server. WARs contain a
Java web application. They can be deployed separately. However, the
deployment of a WAR may require the server to be restarted. Since
microservices should only be deployable separately (see
[chapter 1](#chapter-microservices)), microservices can be implemented
as WARs. Then, however, compromises are made with regards to
robustness. A memory leak in a microservice can lead to failure of all
microservices. The microservice would allocate more and more memory
until an *OutOfMemoryError* is thrown and
the entire Java application server crashes.

Separate scalability is also difficult to implement since each server
contains all microservices, and therefore only all microservices can
be scaled together. This makes the scaling more complex than in cases
where each server contains only one microservice because unneeded
microservices are also scaled. Of course it would be possible to run
each WAR on a separate Java web servers and have multiple instances of
each of these Java web servers. But then the WARs no longer run
together on one Java web server.

And finally, all microservices run in
one operating system process, which is a compromise in terms of
security. When a hacker can take over the process, he/she has access
to the entire functionality and data of all microservices.

In return, these approaches consume less resources. An application
server with several web applications requires only one JVM (Java
Virtual Machine), only one process, and only one operating system
instance. Furthermore, there is no need to introduce a new
infrastructure if application servers are already in use, which can
reduce the workload for operations.

## 4.3 Docker Installation and Docker Commands {#section-docker-installation-kommandos}

Since the examples in this book all require Docker, a Docker
installation is an essential prerequisite for actually starting the
demos and working through them. [Appendix A](#anhang-installation)
describes the installation of the necessary software for the examples.
This also includes the installation of Docker and Docker Compose.

Docker is controlled with the command line tool `docker`. It offers a
lot of commands to work with the images and containers. The
[reference](https://docs.docker.com/engine/reference/commandline/cli/)
contains a complete list of all commands with all options. The
[appendix C](#anhang-docker) shows an overview of the Docker commands.

## 4.4 Installing Docker Hosts with Docker Machine {#section-docker-machine}

Docker Machine is a tool that can install Docker hosts. From a
technical point of view, the installation is quite easy to do. Docker
Machine loads an ISO CD image with boot2docker from the Internet.
boot2docker is a Linux distribution and provides an easy way to run
Docker containers. Then Docker Machine starts a virtual machine with
this boot2docker image.

Particularly convenient with Docker machine is the fact that using the
Docker containers on external Docker hosts is just as easy as using
local Docker containers. The Docker command line tools only need to be
configured to use the external Docker host. Afterwards, the use of the
Docker host is transparent.

#### Overview

[Figure 4-3](#fig-docker-docker-machine) shows an overview of Docker
Machine. Docker Machine installs a virtual machine on which Docker is
installed. Docker and other tools such as Docker Compose then can use
this virtual machine as if it were the local computer.

{id="fig-docker-docker-machine"}
![Fig. 4-3: Docker Machine ](images/docker-docker-machine.png)

The command

{linenos=off}
~~~~~~~~
docker-machine create --driver virtualbox dev
~~~~~~~~

creates a
Docker host with the name `dev` with the virtualization software
Virtualbox. This requires that Virtualbox is installed on the
computer.

Afterwards

{linenos=off}
~~~~~~~~
eval "$(docker-machine env dev)"
~~~~~~~~

on Linux/macOS configures Docker in such a way that the `docker`
command line tools
use the Docker host in the virtual Virtualbox machine. If necessary,
the shell used must be specified.

{linenos=off}
~~~~~~~~
eval "$(docker-machine env --shell bash dev)"
~~~~~~~~

For Powershell on Windows the command is

{linenos=off}
~~~~~~~~
docker-machine.exe env --shell powershell dev
~~~~~~~~

and for cmd.exe on Windows it is

{linenos=off}
~~~~~~~~
docker-machine.exe env --shell cmd dev
~~~~~~~~

`docker-machine rm dev` deletes the Docker host again.

#### Docker Machine Drivers

Virtualbox is only one option. There are many more
[Docker Machine drivers](https://docs.docker.com/machine/drivers/) for
Cloud providers such as Amazon Web Services (AWS), Microsoft Azure, or
Digital Ocean. In addition, there are drivers for virtualization
technologies such as VMware vSphere or Microsoft Hyper-V. In this way
Docker Machine can easily install Docker hosts on many different
environments.

#### Advantage: Separate Environments and Docker on Servers

Docker Machine allows one to completely separate Docker systems from
each
other so that, for example, after a test, nothing remains of the
system and all resources are indeed released again. In addition,
Docker containers can thus be started very easily on a cloud or
virtual infrastructure.

Running the examples in the book directly with Docker is the easiest
option and therefore recommended. Docker Machine should only be used
for the examples if they are to run on a server or should be
completely separated from other Docker installations.

## 4.5 Dockerfiles {#section-docker-dockerfiles}

The creation of Docker images is done via files that are named
`Dockerfile`. One of Docker's strengths is that Dockerfiles are easy
to write and therefore, the rolling out of software can be automated
without any problems.

The typical components of a `Dockerfile` are:

* `FROM` defines a base image on which the installation is based.
  A base image for a microservice usually contains a Linux
distribution and basic software such as for instance the  JVM.

* `RUN` defines commands, that are executed to create the Docker
image. In essence, a `Dockerfile` is a shell script which installs the
software.

* `CMD` defines what happens when the Docker container is started.
Typically, only one process should run in one Docker container. This
is started by `CMD`.

* `COPY` copies files in the Docker image. `ADD` does the same,
however can also unpack archives and download files from a URL on the
Internet. `COPY` is simpler to understand
because it does not extract archives for instance. Also from a
security perspective it can be problematic to download software from
the Internet into Docker containers. Therefore, `COPY` should be given
preference over `ADD`.

* `EXPOSE` exposes a port of the Docker container. This can then be
contacted by other Docker containers or can be tied to a port of the
Docker host.

On the Internet a comprehensive
[reference](https://docs.docker.com/engine/reference/builder/) is
provided which contains additional details to the commands in
`Dockerfile`.

#### An Example for a Dockerfile

A simple example of a `Dockerfile` for a Java microservice looks like
this:

{linenos=off, lang="Dockerfile"}
~~~~~~~~
FROM ewolff/docker-java
COPY target/customer.jar .
CMD /usr/bin/java -Xmx400m -Xms400m -jar customer.jar
EXPOSE 8080
~~~~~~~~

* The first line defines the base image with `FROM`. It is downloaded
from the public Docker hub. Additional information about this image
can be found at
  <https://hub.docker.com/r/ewolff/docker-java/>. The image contains
  an Alpine Linux distribution and a Java Virtual Machine (JVM).

* The second line adds a JAR file to the image with `COPY`. A JAR
file (Java ARchive) contains all components of a Java application. It
has to be available in a sub directory `target` below the directory in
which the `Dockerfile` is stored. The JAR file is copied into the root
directory of the container.

* The `CMD` entry determines which process should be started when the
container is started. In this example, it is a Java process which
starts the JAR file.

* Finally, `EXPOSE` makes a port available to the outside. This is the port
under which the application is available. `EXPOSE` only means that the
container provides the port. It is then available in the internal
Docker network. Access from outside is only possible when this is
enabled at the start of the container.

The Docker image can be built with the command `docker build
--tag=microservice-customer microservice-customer`. `docker` is the
command line tool with which most functionalities of Docker can be
controlled. The created Docker image has the tag
`microservices-customer` as defined by the `--tag` parameter. The
`Dockerfile` has to be in the sub directory `microservice-customer`.
The name of this directory is the second parameter.

#### File System Layers in the Example 

[Figure 4-1](#fig-docker-dateisysteme) shows that a Docker image
consists of multiple layers. Although no layers have been defined in
`Dockerfile`, also the image `microservices-customer` contains
multiple layers. Each line of the `Dockerfile` defines a new layer.
These layers are reused. Thus, if `docker build` is called again,
Docker will go through the `Dockerfile` again. However, it will find
that all actions in `Dockerfile` have already been executed once. As a
result, nothing happens. If the `Dockerfile` was
modified in such a way that after the `COPY` another line with a
`COPY` of another file is inserted, Docker would reuse the existing
layer with the first `COPY`, but the second `COPY` and all further
lines would then create new layers. In this manner, Docker only
recreates
the layers that need to be rebuilt. This not only saves storage space
but is also much faster.

#### Problem with Caching and Layers

A `Dockerfile` for obtaining a Ubuntu installation with updates looks
like this:

{linenos=off, lang="Dockerfile"}
~~~~~~~~
FROM ubuntu:15.04
RUN apt-get update ; apt-get dist-upgrade -y -qq 
~~~~~~~~

First, a Ubuntu base image is loaded from the public Docker hub on
the Internet. The commands `apt-get update` and `apt-get dist-upgrade
-y -qq` are used to update the package index and then install all
packages with updates. The options ensure that `apt-get` does not ask
the user for permission and outputs only few messages on the console.

The two commands are separated in the line by a `;`. This causes them
to be executed one after the other. A new file system layer is created
only after both commands have been executed. This is useful for
creating more compact images with fewer layers.

However, this `Dockerfile` also has a problem. If the Docker image is
built again, no current updates will be downloaded. Instead, nothing
happens because the images are already there. Layer caching is based
only on commands. Docker does not recognize that the external package
index has changed. To ignore the existing images and force the
rebuilding of the images, the parameter `--no-cache=true` can be
passed to `docker build`.

#### Docker Multi Stage Builds

Everything that is needed to build a Docker image can also be found in
the Docker image. So all of it is available at runtime of the Docker
container.
If code is being compiled in a `Dockerfile`, the compiler is also
available at runtime. This is 
unnecessary. It can even be a security problem. If the container is
compromised, the attacker can compile code inside the container with
the original tools which might allow more attacks. It is not easy to
delete all of the build environment because nowadays there is usually
a complex tool chain for building software.

Building the software outside of Docker might also not be an
option. Docker is based on Linux. So on macOS you would need to run a
cross compiler to generate Linux binaries.

To solve this problem, Docker has
Multi Stage Builds. They make it possible to compile the program in
one phase of
the build in a Docker image and to transfer only the compiled program
to the next phase into a different Docker image.
Then the build tools are no longer available at runtime. They do not have
to be deleted and they do not have to be installed on the host
machine.

[Section 5.4](#section-technisch-mikro-go) shows a
Docker Multi Stage Build using a Go program as example.

#### Immutable Server with Docker

*Immutable server* is an idea that predates Docker.
The idea of an immutable server is that a server will never be
changed. So the software on the server will never be updated or
modified. The server will always be
completely rebuilt from scratch. In this way, the state of the server can be
reconstructed cleanly. So for each server there is an installation
script that installs all the needed software on a basic OS image.

However, immutable servers are hard to implement. It is very
cumbersome to completely reinstall
a server. The process might take minutes or hours. That is far too
much time to e.g. just change a configuration file.

This is exactly where Docker helps. Because of the
optimizations, only the necessary steps are taken so that
*immutable servers* can also be an option from this perspective. A
`Dockerfile` describes how to create a Docker image starting from a
base image. With each build, it will seem as if the complete Docker image is
being created. Behind the scenes, however, optimizations
ensure that only what is really necessary is built.
E.g. if in the very last step a configuration file is added and just
that configuration file has been modified, Docker is smart enough to
reuse the results of all other installation steps and just add the new
configuration file. This will just take a few seconds.

So Docker is conceptually as clear as immutable server but much more
efficient for actually implementing them.

#### Docker and Tools such as Puppet, Chef or Ansible

Besides immutable servers, there are other ways to handle the
installation of software. *Idempotent installation* means that an
installation script provides the same results no matter how often they
run. For an idempotent installation, there are no steps like 'install
the Java package' but rather a definition of the desired state:
'ensure that the Java package is installed'. So if the installation is
run on a fresh OS, Java would be installed. If the installation is run
on a system that already has Java installed, nothing happens.

Idempotent installation is in particular useful to enable updates. For
each update the script would check if all software is installed in the
correct version. If that is not the case, the correct version is
installed. Tools such as Puppet, Chef, or Ansible support the concept
of idempotent installation.

A `Dockerfile` is a very easy way to install software. The use of
tools such as Puppet, Chef, or Ansible for the installation of software
in a Docker image is possible, but does not make a lot of sense. In
particular, it is
not necessary to use the update functionalities of these tools, since
the image is typically freshly built with the *immutable server*
approach. This approach is easier than writing Puppet, Chef or Ansible
scripts because defining the desired state is usually quite
complex. The `Dockerfile` only describes how to build an image, while
the other tools must also enable updates of the servers and are
therefore more difficult to use.

## 4.6 Docker Compose {#section-docker-docker-compose}

A typical microservice systems contains more than a single Docker
container. As explained in [chapter 1](#chapter-microservices),
microservices are modules of a system. So it would be good to have a
way to start and run several containers together for starting all the
modules the system consists of in one go. This can be done with
[Docker Compose](https://docs.docker.com/compose/).

#### Service Discovery with Docker Compose Links

Actually
coordinating a system of multiple Docker containers requires more than
just starting multiple Docker containers. It also requires
configurations for the virtual network with which the Docker
containers communicate with each other. In particular, containers must
be able to find each other in order to communicate. In a Docker
Compose environment, a service can simply contact another service via
a Docker Compose link and then using the service name as host name.
So it could use a URL like `http://order/` to contact the order
microservice.

Docker Compose links offer some kind of service discovery, i.e. a way
for microservices to find other microservices. Synchronous
microservices require a form of service discovery (see also
[section 13.3](#section-synchron-herausforderungen)).

Docker Compose links extend Docker links. Docker links only allow
communication. Docker Compose links also implement load balancing and
set the start order so that the dependent Docker containers start
first.

#### Ports 

In addition, Docker Compose can bind ports from the containers to the
ports of the Docker host where the Docker containers run.

#### Volumes 

Docker Compose can also provide volumes. These are file systems that
can be shared by multiple containers. This allows containers to
communicate by exchanging files.

#### YAML Configuration

Docker Compose configures the interaction of the Docker containers
with a YAML configuration file `docker-compose.yml`.

The following file comes from a project for [chapter 9](#chapter-esi),
which implements Edge Side Includes as a way to compose websites from
different sources. For this purpose, three containers must be
coordinated.

* `common` is a web application that is supposed to deliver common
artifacts.

* `order` is a web application for processing orders.

* `varnish` is a web cache to coordinate the two web applications.

{lang="YAML"}
~~~~~~~~
version: '3'
services:
  common:
    build: ../scs-demo-esi-common/
  order:
    build: ../scs-demo-esi-order
  varnish:
    build: varnish
    links:
     - common
     - order
    ports:
     - "8080:8080"
~~~~~~~~

* The first line defines the used version of Docker Compose -- in this
case three.

* The second line starts the definition of the services.

* Line three defines the service `common`. The directory specified in
line four contains a `Dockerfile` with which the service can be built.
An alternative to `build` would be `image` to use a
Docker image from a Docker registry.

* The definition of the service `order` also specifies a directory
with a `Dockerfile`. No other settings are required for this service
(lines 5/6).

* The service `varnish` is also defined by a directory with a
`Dockerfile` (lines 7/8).

* The service `varnish` must have Docker Compose links to the services
`common` and `order`. Therefore, it has entries under `links`. The
`varnish` service can therefore reach the other services using the
host names `common` and `order` (lines 9-11).

* Finally, port 8080 of the service `varnish` is bound to port 8080 of
the Docker host, on which Docker containers run (lines 12-13).

#### Additional Options 

Further elements of the YAML configuration are described in the
[reference documentation](https://docs.docker.com/compose/compose-file/).
For example, Docker Compose supports volumes shared by multiple Docker
containers. Docker Compose can also configure the Docker containers
using environment variables.

#### Docker Compose Commands

Docker Compose is controlled by the command line tool
`docker-compose`. It must be started in the directory where the file
`docker-compose.yml` is stored. The
[reference documentation](https://docs.docker.com/compose/reference/overview/)
lists all command line options for this tool. The
[appendix C](#anhang-docker) shows an overview of the Docker Compose
commands. The most important ones are:

* With `docker-compose up` all services are started. The command
returns the combined standard output of all services. This is rarely
helpful, so `docker-compose up -d` is often the better choice. In this
case, the standard output is not returned. With `docker log` the
output of individual containers can be viewed.

* `docker-compose build` builds the images for the services.

* `docker-compose down` shuts down all services and deletes the
containers.

* With `docker-compose up --scale <service>=<number>` a larger number
of containers for a service can be started. In the example from the
listing, for example, `docker-compose up --scale common=2` could
ensure that two containers for the service `common` are started.

Since the examples often require the interaction of several Docker
containers, most examples have a `docker-compose.yml` file to run the
containers together.

## 4.7 Variations {#section-docker-variationen}

There are not really any fundamental alternatives to Docker:

* *Virtualization* has too much overhead.

* *Processes* are not sufficiently isolated. The required libraries and
runtime environments for all microservices must be installed in the
operating system. This can be difficult because each process occupies
one port and therefore the allocation of ports must be coordinated.

* Other *container solutions* such as [rkt](https://coreos.com/rkt) are
far less common.

#### Clusters

For production, applications should run in a cluster. Only in this way
can the system be scaled across several servers and secured against
the failure of individual servers.

Docker Compose can use [Docker Swarm Mode](https://docs.docker.com/engine/swarm/) for cluster management. Docker Swarm Mode is built into Docker.

Kubernetes is widely used for operating Docker containers in a cluster
(see [section 17](#chapter-kubernetes)). There are also other systems like
[Mesos](http://mesos.apache.org/). Mesos is actually a system for
managing batches for data analysis in a cluster, for example. But it
also supports Docker containers. There are also offers in the cloud
such as AWS (Amazon Web Services) and
[ECS](https://aws.amazon.com/documentation/ecs/) (EC2 Container
Service).

These approaches have one thing in common. How the load is distributed
in the cluster is decided by the scheduler -- that is, by Kubernetes
or Docker Swarm Mode. This means that the scheduler is of crucial
importance for fail-safety and load balancing. If a container fails, a
new container must be started. Likewise, additional containers must be
started at times of high loads, ideally on machines that are not too
busy.

Schedulers such as Kubernetes solve many challenges, especially with
synchronous microservices. It is highly recommended to use such a
platform for operating microservices. But the introduction of
microservices requires many changes. The architecture needs to be
adapted, developers need to learn new approaches and technologies, and
have to adapt the deployment pipeline and the tests. Implementing an
additional technology such as a Docker scheduler should therefore be
well thought out, as many other changes also have to be made.

#### Docker without Scheduler

An alternative is to install Docker containers directly on a server.
In this scenario, the servers are provided with classic mechanisms --
for example, with virtualization. Linux or Windows is installed on the
server. The microservice runs in a Docker container.

The only difference to the procedure without Docker is that Docker
containers are now used for deployment. But this already makes it much
easier to keep production and test environments identical. Concepts
such as *immutable servers* are also easier to implement. In addition, the technology
freedom for microservices is easier to implement. Traditional
virtualization is still responsible for high availability, scaling, and
distribution to the servers.

So this approach offers some of the advantages of Docker and reduces the
effort.

#### PaaS

Another alternative is a PaaS (see [chapter 18](#chapter-paas)). A
PaaS has a higher degree of abstraction than Docker schedulers because
only the application needs to be provided. The PaaS creates the Docker
images. Therefore, a PaaS can be the simpler and therefore better solution.
[Chapter 18](#chapter-paas) shows Cloud Foundry as a concrete example
of a PaaS.

#### Experiments

* Docker Machine can use clouds like Amazon Cloud or Microsoft Azure
with the appropriate
[drivers](https://docs.docker.com/machine/drivers/). Create an account
with one of the cloud providers. Most cloud providers provide free
capacity to a new user. Install Docker with Docker Machine and the
appropriate driver on the virtual machines in the cloud and use this
configuration to run one of the examples from the following chapters
(see [section 4.4](#section-docker-machine)).

* Create an account in the [Docker Hub](https://hub.docker.com/).
Build a Docker image, for example, based on one of the microservices
examples of the following chapters and place it in the Docker hub with
`docker push`.

* Use the
[tutorials](https://docs.docker.com/engine/swarm/swarm-tutorial/) to
familiarize yourself with the Docker Swarm Mode, which can be used to
run Docker in a cluster.


## 4.8 Conclusion {#section-docker-fazit}

Docker is a light-weight alternative for deploying and operating
microservices. A microservice with all its dependencies can be packed 
into a Docker image and can then be well isolated from other
microservices as a Docker container. Virtual machines appear too
heavy-weight by comparison, while simple processes do not provide the
necessary isolation.

Docker makes it easier to deploy the software. It is only necessary to
distribute Docker images. `Dockerfiles` are used for this purpose,
which are very easy to write. Concepts such as *immutable server* are
also much easier to implement. With Docker Compose, multiple
containers can be coordinated to thereby build and launch an entire
system of microservices in Docker containers. Docker Machine can very
easily install Docker environments on servers.

However, Docker requires rethinking regarding operation. Therefore, in some cases, alternatives might be helpful. This can be, for example, the deployment of several Java web applications on a single Java web server.

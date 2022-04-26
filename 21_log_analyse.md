# 21 Recipe: Log Analysis with the Elastic Stack {#chapter-log-analyse}

This chapter describes the analysis of log data.

* First, the chapter shows why logs are so widely used and how they
facilitate system operation.

* Logs in microservices systems have to respond to other demands
than logs in traditional systems. The chapter explains how even large
microservices systems can be provided with a suitable system for log
analysis.

* As concrete solution for the analysis of log data the chapter
introduces the Elastic stack.

Thereby, the reader gets to know how logs can be efficiently and
effectively processed in a microservices system.

## 21.1 Basics {#section-log-analyse-grundlagen}

Logs differ from metrics. Metrics represent the current state
of the system. Logs record events. These can comprise errors or
business events such as user registrations. Metrics measure for
example throughput. Experience shows that both data types require
specialized tools. Although it is technically possible to write
metrics into logs, it has proven to be a bad approach. It adds even
more data to the logs that are usually already quite huge. Also
calculating metrics from the logs takes
additional time and processing power.

#### Why Logs?

Logs are very simple. They are text files containing information about
events that happened. This has several advantages.

* *Every programming language and infrastructure* supports log files.
Thus, there is no lock-in and no restriction regarding the
technologies used.

* The data are *permanently stored* so that it is possible to trace
and analyze events long after they happened.

* Linear writing into a file is *very fast*. Therefore, logging hardly
affects system performance.

* Log files are *easy to analyze*. The data is human-readable. Even
simple tools
like `grep` or `tail` make it easy to quickly get an overview and to
analyze the data.

This simplicity is the strength of logs.

#### Logs with Microservices

For microservices the situation is dramatically different compared to
traditional systems:

* Microservice instances *come and go*. It is not enough to store
the log data in the instance. The instance and the stored log data 
can be lost.

* With microservices-based architectures there is *a large number of
systems*. Nobody can log into all these systems and analyze their log
files. This requires much too much effort. So it is not enough to use
simple tools like `grep` or `tail`.

* There can be *new microservices* whose logs also have to be
analyzed.

* This means that a *centralized storage* of the logs has to be used
for analysis. It has to collect the logs from all systems.

* Since a lot more log data has to be analyzed in a microservices
system, the
usual command line tools are not powerful enough for analysis. *More
efficient tools* for the analysis of larger data volumes are
necessary.

* When the logs are anyhow centrally stored and analyzed via a tool,
the logs should be optimized for this and therefore be *machine-readable*.
It is not necessary that they are human-readable.

#### Log Information

To simplify log analysis it is necessary to define a uniform format
for the logs.

An important part of the format is the log level.
*Error* can stand for events which have a negative impact on users.
Such a log entry can result from a complete failure of an operation.
*Warning* can stand for an event where negative effects on the user
were prevented. An example of such a scenario. A system was not
available, but the failure could be compensated by a default
value. *Info* can refer to information with a business meaning like a
new registration. *Debug* contains details that are only relevant for
developers.

Log messages can also store additional standardized information. For
example, an incoming request can have an ID that is output in each log message.
This allows the user to better understand the relationships between
the log
entries. When another microservice is called, the ID can also be
transferred so that log messages that are caused by the request in
other
microservices can be traced in the logs.
[Chapter 22](#chapter-tracing) focuses on tracing, which is
about the traceability of calls between microservices.

Also additional context information can be output. This can
include the current user or the web browser used.

#### Sending Logs Instead of Storing Them

Logs are usually written to a file. In a
microservices system an overview of all microservices is
necessary. All log information must not just be written to a local
file but also be collected on a central server.
So a process can read the local files and send them to the central
server.

An alternative can be asynchronous communication. The log data is no
longer stored in a file, but sent directly to the central server. The
transmission of log data can be separated from the transmission of
business data if the requirements in terms of reliability, speed, etc.
are different.

#### Tool: Map/Reduce

Companies like Google originally used *map/reduce* to analyze log
data. With this approach, a function is applied to each log line. This
is the map phase. In functional programming *map* means that a
function is applied to a list of items. The map function can filter
relevant data from the log line. For example, this allows one to
separate
logs of user registrations from other events. In the *reduce* step, the
individual data records are combined, for example, by counting. The
result might be a statistic of user registrations.

Map/reduce can process large amounts of data and distribute the work
well over many servers. But the results are only available after a
long processing time. That is often unacceptable.

#### Tool: Search Engines

In the meantime, other tools have been established to handle log data.
Search engines are very fast and can process text data efficiently.
They are also well suited for the analysis of large amounts of data.

Search engines are optimized to handle the addition of data
particularly well. They are less able to deal with updates.
Because log data is only added but never changed, search engines also
fit well with log data from this perspective.

#### Elasticsearch

Nowadays search engines such as [Elasticsearch](https://www.elastic.co/products/elasticsearch) can do much more than
just search text. They can also handle other data such as numbers or
geodata. For this purpose, the search engines process structured data
such as JSON documents.

Log entries also have a structure. They consist of a timestamp, a log
level, information about the process or thread, and the actual log
message. This structure should be available to the search engine to do
more efficient searches. A tool such as
[Logstash](https://www.elastic.co/products/logstash) can parse the log
data and pass it on to the search server as JSON.

However, to parse the log files and forward them as JSON to the search
engine
is actually nonsense. The microservice puts the data into a log
entry, which is then parsed again. It would be much easier if the
microservices would directly log JSON data. The
[GELF format](http://docs.graylog.org/en/2.3/pages/gelf.html) can be
used to transmit JSON logs directly over the network. There are even
GELF plug-ins for some logging frameworks so that the use of Logstash
or Filebeat can then be avoided.

If the data is sent to a server and not stored locally,
log messages are actually nothing more than events. Only the logs are
concerned with technical events, while
[chapter 10](#chapter-asynchron) deals with domain events. The
boundaries are fluid. A customer's registration can be processed with
a log system as well as a domain event, for example, to create a
special report.

## 21.2 Logging with the Elastic Stack {#section-log-analyse-elastic}

As an example for the processing of logs in a microservices system,
there is an installation of log tools in the Consul project (see
[chapter 15](#chapter-consul)). [Figure 21-1](#fig-log-analyse-elastic)
shows the structure.

{id="fig-log-analyse-elastic"}
![Fig. 21-1: Processing of Log Data](images/log-analyse-elastic.png)

* The microservice is implemented with Spring Boot and uses
  [Logback](https://logback.qos.ch/) as library for logging. 
  This library allows logging of JSON data. In a microservices
architecture, it makes sense to standardize the format of the JSON
data so that a uniform approach can be used for analysis.
But standardizing the library is not necessary, because only the format of
the data is relevant. A standardization of the library would
unnecessarily restrict the freedom of technology.

* Logback outputs the log data on the *console*. This allows the
developer to get an impression of what the application does when
testing it locally. The output on the console is also processed by
Docker. Docker's log processing can also be the basis for common
logging across different microservices.

* From the console, a *Docker Log Driver* processes the log
information and provides it, for example, for the `docker log`
command. The
[Docker Log Driver](https://docs.docker.com/engine/admin/logging/overview/#configure-the-default-logging-driver)
  can be configured in such a way that also other types of processing are possible.

* Logback also writes the log data in a JSON format to a *Docker
volume*. This volume is shared by all microservices. From there, the
data can be processed further. Actually, this volume is not
necessary because the data should be stored primarily
by the search engine on which the analyses are based. But in this way
there is also a backup of the data. An alternative would be to send
the data directly over the network to Elasticsearch.

* [Filebeat](https://www.elastic.co/products/beats/filebeat) reads the
  JSON data from the Docker volume and sends them to Elasticsearch. In
the example, there is a single Filebeat instance for all logs. When a
new microservice is added to the system, it only needs to log into a
file with a different name. Filebeat reads this file automatically, so
no additional configuration is necessary. In a production system,
a separate instance of Filebeat for each
microservice is the better approach to handle also large amounts of
  data. Concepts like Kubernetes pods can be helpful for this
because in a pod the containers can easily share volumes. For example,
a pod could host a microservice that writes logs to a volume. The
Filebeat process in the same pod can then read data from this volume
and send the data to Elasticsearch. Filebeat also adds some metadata
to the log 
entries.

* Finally,
[Elasticsearch](https://www.elastic.co/products/elasticsearch) stores
the log data.

* [Kibana](https://www.elastic.co/de/products/kibana) offers a user
interface for visualizing, searching, and analyzing the data.

## 21.3 Example {#section-log-analyse-beispiel}

[Section 0.4](#section-einleitung-quick-start) describes which
software needs to be installed to start the example.

The example project at <https://github.com/ewolff/microservice-consul>
contains support for the Elastic Stack. With `git clone
https://github.com/ewolff/microservice-consul.git` the project can be
downloaded.

At
<https://github.com/ewolff/microservice-consul/blob/master/HOW-TO-RUN.md>
installation and use of the example are discussed in more detail. At
<https://github.com/ewolff/microservice-consul/blob/master/HOW-TO-RUN.md#run-the-elastic-example>
you find a detailed description of the structure of the example with the
Elastic Stack.

To start the system with the log analysis, you first have to compile
the application with Maven in the
sub directory `microservice-consul-demo` using `./mvnw clean package`
(macOS, Linux) or `mvnw.cmd clean package` (Windows).
See [appendix B](#anhang-maven) for more details on Maven and how to
trouble shoot the build.
The
configuration for Logback is contained in the file `logback-spring.xml` in
each microservices project.

Afterwards you can create the Docker container in sub directory
`docker` with `docker-compose -f docker-compose-elastic.yml build`.
The standard Docker configuration for the example does not have a
Docker container for Kibana, Elasticsearch, or Filebeat. Changing this
requires the configuration in file `docker-compose-elastic.yml`. With
`docker-compose -f docker-compose-elastic.yml up -d` you can start the
application.
See [appendix C](#anhang-docker) for more details on Docker,
docker-compose and how to troubleshoot them.

The file `docker-compose-elastic.yml` contains in addition to the
Docker containers for microservices the Docker containers for
Filebeat, Elasticsearch and Kibana which are depicted in
[figure 21-1](#fig-log-analyse-elastic). It also contains the required
Docker volume for storing the log files from which Filebeat imports
the files into Elasticsearch.

The application is available at port 8080 on the Docker host. When the
Docker containers are running locally, this is the URL
<http://localhost:8080/>. At port 5601 Kibana is available
(<http://localhost:5601/>). When starting Kibana the name of the
relevant indices has to be given. The name is `filebeat-*`.

{id="fig-log-analyse-kibana"}
![Fig. 21-2: Analysis with Kibana](images/log-analyse-kibana.png)

Subsequently, the log data is available in Kibana
([figure 21-2](#fig-log-analyse-kibana)) and can be analyzed. The
figure shows that the timestamp and other data such as severity are
displayed as separate fields for which the user can search or filter.

## 21.4 Recipe Variations {#section-log-analyse-variationen}

It makes little sense to operate a microservices system without
centralized analysis of the log data. There are simply too many
microservices and microservices instances for manual analysis of the
log data.

The Elastic Stack is very widely used, however, there are also
alternatives.

* [Graylog](https://www.graylog.org/) also uses Elasticsearch
  for storing log data. In addition, it uses MongoDB for
  configuration and meta data. A web interface for analysis is integrated.

* [Apache Flume](https://flume.apache.org/) makes it possible to read
data from one source, to process it, and to write it into a sink. It
can read files and write in Elasticsearch so that it is a possible
alternative to Logstash. Compared with Flume or Logstash, Filebeat
does not have so many options for parsing data.

* [Fluentd](http://www.fluentd.org/) can also be an alternative for
reading, processing, sending, and storing log data.

* Cloud services like [Loggly](https://www.loggly.com/) or
  [Papertrail](https://papertrailapp.com/) spare the user the need to
  set up an infrastructure that can handle the large amounts of log
  data.

* Finally, [Splunk](https://www.splunk.com/) offers a commercial
solution for the installation in your own data center as well as in
the cloud.

* Microservices platforms can also offer log analysis
  ([chapter 16](#chapter-plattformen)). Cloud Foundry as PaaS
  ([chapter 18](#chapter-paas)) and Kubernetes
  ([chapter 17](#chapter-kubernetes)) both offer support for log data.

* A service mesh like Istio (see [chapter 23](#chapter-service-mesh))
  adds proxies to the network traffic between the microservices. They
  can log information about the traffic.
  
* The example in [chapter 23](#chapter-service-mesh) uses
  Elasticsearch and Kibana, too. But it sends JSON to Elasticsearch so
  there is no need to parse the logging information.

## 21.5 Experiments {#section-log-analyse-experimente}

The Elastic stack offers many interesting options.

* The Kibana reference documentation is a good basis to familiarize
yourself with the analysis possibilities. There are explanations for
[discovery](https://www.elastic.co/guide/en/kibana/current/discover.html)
and
[visualization](https://www.elastic.co/guide/en/kibana/current/visualize.html).

* Integrate [Logstash](https://www.elastic.co/guide/en/logstash/current/getting-started-with-logstash.html)
  into the setup. This requires a configuration to read and parse the
files from Filebeat. The linked documentation can be a start for this.

* Replace the individual Elasticsearch instance with a cluster. The
[documentation on starting with Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
can be helpful for this purpose.

* Finally, logging can be changed in such a way that the microservices
directly send log data as JSON to Elasticsearch without saving the
data beforehand. For this the
  [Logback Elasticsearch Appender](https://github.com/internetitem/logback-elasticsearch-appender)
  can be useful.

* In addition, the logs can be delivered directly by Docker using a
[Docker Log Driver](https://docs.docker.com/engine/admin/logging/overview/#configure-the-default-logging-driver).
On the Internet, you will find instructions to also in this way
implement an integration into Elastic Stack. Logstash can accept logs
from different sources such as MOMs and send them to Elasticsearch.

## 21.6 Conclusion {#section-log-analyse-fazit}

Managing log data is very important in a microservices system because
there are many microservices. The data must be collected and analyzed
centrally. Instead of a simple solution with human-readable logs, a
microservices system must have a solution that can handle large
amounts of data and makes them available for machine analysis. The
Elastic Stack can be used to create such an environment for log
analysis.

#### Advantages

* Elasticsearch can analyze large amounts of data quickly and scales
well.

* The Elastic Stack is widely used, so there is a wealth of experience
with this technology.

#### Challenges

* Log files can no longer be easily analyzed with simple command line
tools, but only with
the Elastic Stack.

* The setup of a reliable and scalable log processing is
complex due to the large amount of data.

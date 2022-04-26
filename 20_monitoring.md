# 20 Recipe: Monitoring with Prometheus {#chapter-monitoring}

This chapter focuses on the monitoring of microservices. Essential
topics are:

* This chapter describes the basics of monitoring: Which metrics need
to be monitored and what they are useful for.

* Microservices monitoring places new requirements on monitoring
tools.

* Prometheus has features that are advantageous for monitoring
microservices environments.

* Other tools are also suitable for monitoring microservices.

## 20.1 Basics {#section-monitoring-grundlagen}

Monitoring visualizes and analyses metrics that describe the state of
the system.

There are different kinds of metrics.

* *Counters* can be increased and thereby can count events. For
example, the number of registered users or the number of processed
tasks can be counted.

* *Gauges* display a value which changes over time. For example, the
memory usage or the network traffic can be measured with a gauge.

* A *histogram* measures the number of certain events.
For example, the number of processed requests can be
counted, and additionally the number of successfully processed requests
can be distinguished from the requests that failed due to different
kinds of errors.

#### Processing Metrics

{id="fig-monitoring-metriken", width=70%}
![Fig. 20-1: Metrics: Kinds and Processing](images/monitoring-metriken.png)

Often, metrics are saved in time series databases. Time series
databases are specialized in saving data which are indexed based on
time.

The metrics can be processed in different ways.

* When a metric attains a certain value, an *alert* can be triggered
to inform an operations team member who then analyzes and solves the
problem.

* With the help of a *graph* the metrics can be visualized which makes
it easier to quickly recognize changes. This allows a
human to analyze the system.

* A *dashboard* displays multiple metrics in an overview. This very
quickly conveys an impression of the state of the system. For example,
the dashboard can be displayed on a screen close to the coffee
machine so that the employees are informed about the current state of
the system while getting a coffee.

* *Complex event processing (CEP)* processes metrics and implements
analyses. For example, the metrics can be condensed to average values
over a certain interval.

* *Anomaly detection* automatically recognizes whether metrics lie
outside of the expected range and thereby indicates problems early on.

#### Different Metrics for Different Stakeholders

Different stakeholders can be interested in different metrics. For
example, system metrics focus on the level of operating system or
hardware. Network throughput or hard disk throughput fall into this
category. Operations experts are usually interested in such metrics.

Application metrics describe for example the number of different kinds
of requests or of method calls to a certain method. These metrics help
developers to do their work.

Finally, there are business metrics such as sales or the number of
registered users. Such metrics are often interesting for the product
owners or users.

Technically, a single system can save all these metrics. However,
often the
stakeholders want to use different tools. Specialized tools often offer
better analysis options or are already familiar to the stakeholders.
For example, for web analytics tools like
[Google Analytics](https://analytics.google.com/) can be used.
So when the stakeholders insist on using different tools, it is
conceivable to store the different metrics in different systems.

The metrics can be used to draw different conclusions. Product owners
can evaluate the business success, operations can plan capacities, and
developers can analyze errors.

## 20.2 Metrics for Microservices {#section-monitoring-microservices}

Regarding the processing of metrics, microservices differ from
classical systems in some important points.

#### More Services, More Metrics

There are substantially more instances of microservices than would be
the case with a deployment monolith. Therefore, there are also a lot
more metrics. Consequently, the system for processing metrics has to
scale appropriately. In addition, all metrics of all microservices
should be stored in one system to keep the overview.

#### Services Instead of Instances

For most microservices several instances are running. This is the only
way to ensure fail-safety and to handle the load. For metrics the
individual instances are not interesting, but the results of all
instances of a microservice and therefore the quality of service it
provides to the user. 
Thus average metrics of all instances of a specific microservice are
important, but not the metrics for each specific instance.

#### Away from System Metrics

In addition, the processing of metrics has to change. In a classical
system the failure of a server normally causes an alert or even leads
to the operations expert being called up from sleep. In a
microservices system this might not be necessary. Each microservice
can be scaled independently so that usually multiple instances of each
microservice are running. When a server fails, new instances can be automatically started on other servers.

Due to their robustness and resilience microservices can even
compensate the failure of other microservices. Therefore, the failure
of a server or of an instance of a microservices is less problematic
than in a traditional system.

The system can even compensate an overload by
starting new instances on new servers. This allows it to keep pace
with the higher load without manual intervention.

#### Towards Application Metrics

Overall, system metrics are not as important for microservices as for
traditional systems. However, it is important to analyze the system
from a user's point of view. For example, requests may take too long
which might affect user behavior. And this might happen despite
the fact that the system metrics are all normal. For example, there
may be an error in the software or data may
lead to the problem. If the additional waiting times have a negative
effect on revenue, this can be a reason for immediately triggering an
alert when waiting times are increased, even though the system metrics
are normal. Thus, the priority shifts from system metrics to
application and business metrics. Service level agreements (SLAs) can
be a good source of metrics. SLAs define what the customer expects
from the system in terms of response times, for example.

#### Alerts Based on Domain Logic

Whether application metrics show unusual values and whether it is
necessary to react to them is a decision that must be made by domain
experts. Typically, developers have a good insight into the
implemented domain functionality so that alerts can be defined by
operations in close collaboration with the developers. However,
operations alone cannot typically do this.

## 20.3 Metrics with Prometheus {#section-monitoring-prometheus}

[Prometheus](http://prometheus.io/) is a relatively new tool for
monitoring. It has a number of interesting features.

* Prometheus supports a *multidimensional data model*. For example,
the duration of HTTP requests per URL and per HTTP methods (e.g. GET,
POST, DELETE) can be stored. For each URL, there is a value for each of
the four HTTP methods. Prometheus can sum up the duration of all HTTP
methods for a URL, or the duration of an HTTP method for all
URLs. This allows more flexibility in the 
analysis. The evaluation is carried out using a query language. The
results cannot only be calculated, but also displayed graphically.

* Many monitoring solutions require the monitored systems to write the
metrics to the monitoring system (push model). Prometheus has a *pull
model* and collects data from an HTTP endpoint in the monitored
systems at specified intervals. This corresponds to the procedure for
asynchronous communication via HTTP depicted in
[chapter 12](#chapter-atom). For communicating with applications that
use a push model, there is the
[Push Gateway](https://github.com/prometheus/pushgateway).

* There is an
[*alertmanager*](https://prometheus.io/docs/alerting/alertmanager/).
  It can trigger alerts based on queries and can send the alert
  messages as e-mail or per [PagerDuty](https://www.pagerduty.com/).

* With *dashboard* the user can execute queries and can have the
results displayed graphically. Prometheus can also integrate into
other visualization tools, for example
into [Grafana](https://prometheus.io/docs/visualization/grafana/).

At its core, Prometheus is a multidimensional time series database and
also offers support for alerts and graphical evaluations. With the
multidimensional database, it is possible to capture the metrics of
all instances of all microservices and then sum them up for a
certain type of microservice.

#### Examples for Multidimensional Metrics

Prometheus can count the number of HTTP requests. Based on this,
Prometheus can use the built-in
[functions](https://prometheus.io/docs/querying/functions/) to
calculate and process the number of HTTP requests per second. For
example, this allows Prometheus to calculate the average for each
handler
([figure 20-2](#fig-monitoring-prometheus-graph)). The formula
`avg(rate(http_requests_total[5m])) by (handler)` is used for this
purpose.

{id="fig-monitoring-prometheus-graph", width=60%}
![Fig. 20-2: Prometheus Dashboard with Calculated Metrics](images/monitoring-prometheus-graph.png)

Prometheus can also calculate the average based on HTTP codes. All
that needs to be done is to change the formula to
`avg(rate(http_requests_total[5m])) by (code)`.
[Figure 20-3](#fig-monitoring-prometheus-graph-code) shows the result.

{id="fig-monitoring-prometheus-graph-code", width=60%}
![Fig. 20-3: Prometheus Dashboard with Other Calculated Metrics](images/monitoring-prometheus-graph-code.png)

This enables Prometheus to extract information from the metrics of all
microservices that are of interest to users. If an instance of a
microservice is very slow, this is less problematic than a general
problem affecting all instances of a microservice. Prometheus'
evaluations make it possible to draw exactly such conclusions.

## 20.4 Example with Prometheus {#section-monitoring-beispiel}

The example for microservices with Consul (see
[chapter 15](#chapter-consul)) can be monitored with Prometheus. This
only requires a Docker compose configuration with some additional
containers.

#### Starting the Environment

[Section 0.4](#section-einleitung-quick-start) describes which
software has to be installed for starting the example.

The example project can be found at
<https://github.com/ewolff/microservice-consul>. First, the code has
to be downloaded with `git clone
https://github.com/ewolff/microservice-consul.git`.

At <https://github.com/ewolff/microservice-consul/blob/master/HOW-TO-RUN.md>
instructions are provided which describe in detail how to start the example and how to install the necessary software. The approach for monitoring with Prometheus is explained in more detail at
<https://github.com/ewolff/microservice-consul/blob/master/HOW-TO-RUN.md#run-the-prometheus-example>.

For starting the system, the application first has to be compiled
with `./mvnw clean
package` (macOS, Linux) or `mvnw.cmd clean package` (Windows) in the
sub directory `microservice-consul-demo`.
See [appendix B](#anhang-maven) for more details on Maven and how to
trouble shoot the build.

Afterwards create the Docker containers with `docker-compose
-f docker-compose-prometheus.yml build` in the sub directory `docker`.
See [appendix C](#anhang-docker) for more details on Docker,
docker-compose and how to troubleshoot them.
In the normal system configuration of the Consul example there is no
Prometheus Docker container. So the configuration in the file
`docker-compose-prometheus.yml` has to be used. With `docker-compose
-f
docker-compose-prometheus.yml up -d` the application can be started.

As with the Consul demo, the application is available at port 8080,
for example at the URL <http://localhost:8080/>. On the Docker host
Prometheus can be reached at port 9090. So if the Docker containers
are running locally, the Prometheus UI
is available at <http://localhost:9090/>. For debugging,
<http://localhost:9090/targets> is useful. At this URL the state of
all jobs Prometheus uses to fetch metrics from the various
microservices is displayed.

#### Code in Spring Boot

In the example project, the order microservice has an interface to
Prometheus for monitoring data. The other microservices do not offer
such an interface.

As explained in [section 5.3](#section-technisch-mikro-spring-boot),
Spring Boot offers Sprint Boot Actuator as a way to collect metrics
about the microservices. These metrics include, for example, the
number and duration of HTTP requests. To support Prometheus, these
metrics only need to be provided in such a way that Prometheus can pull
them.

This requires the following changes to the project:

* An additional dependency to the Prometheus client library has to be
inserted in `pom.xml`.

* Unfortunately, Prometheus cannot use the usual Spring Boot Actuator
REST endpoints for metrics. Therefore, the Prometheus servlet has to
be integrated. Also the Spring Boot Actuator metrics have to be provided
to Prometheus with a `SpringBootMetricsCollector`. This class belongs
to the Prometheus client library. In the class
`PrometheusConfiguration` an instance of this class is created as a
Spring Bean to make the metrics available to Prometheus.

#### Prometheus Configuration

Offering metrics alone is not enough. Prometheus also has to pick up
the metrics from the order microservice. The sub directory
`docker/prometheus` in the example project contains a Dockerfile to
create a Prometheus installation 
in a Docker image. It is based on the Prometheus Docker
image of the Prometheus development team and only adds a custom
configuration. The configuration format is described in more detail in
the
[Prometheus configuration documentation](https://prometheus.io/docs/operating/configuration/).

#### Configuration in the Example

The configuration in the file `prometheus.yml` looks like this:


{lang="yaml"}
~~~~~~~~
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'order'
    scrape_interval: 10s
    metrics_path: '/prometheus'
    static_configs:
    - targets: ['order:8380']
~~~~~~~~

The section `global` (line 1) contains settings which influence the
entire Prometheus system. This is in the example only the default
setting `scrape_interval` (line 2) which determines how often the
metrics are fetched from the monitored microservices
("scraping").

The section `scrape_configs` (line 4) defines the jobs. A job is a
configuration with which Prometheus fetches metrics from a server for
storage and processing.

The job `prometheus` (lines 5--8) serves to monitor Prometheus itself.
Prometheus fetches metrics from the server `localhost` at port 9090.
That is the port on which Prometheus listens and also provides its
metrics.
Prometheus scrapes the metrics every five seconds and therefore more
frequently than the
default value of fifteen seconds.

The other job is called `order` (lines 10--14) and serves for
monitoring the order microservice. Prometheus fetches the metrics
every ten seconds from the server `order` at port 8380. In the Docker
compose configuration, a Docker compose link is generated so that the
host name `order` is resolved to the order microservice.
`metrics_path` is also defined for the job. The default is
`/metrics`. However, for a Spring Boot application the JSON formatted
Spring Boot Actuator metrics are available at this path. Prometheus
requires a custom text-based format that is made available in a
Spring Boot application under the path `/prometheus`.

## 20.5 Recipe Variations {#section-monitoring-variationen}

The service mesh Istio uses Prometheus as a monitoring tool. Istio
uses proxies to
measure the traffic between the microservices. So it can provide 
information about the communication such as the number of requests,
the amount of transferred data and the number of failed requests
without any modification to the microservice (see
[section 23.4](#section-service-mesh-monitoring)).

#### Other Tools

In the area of monitoring there is a large number of tools available.

* [StatsD](https://github.com/etsy/statsd) can consolidate the
collected metrics in order to send less data over the network. The
[StatsD exporter](https://github.com/prometheus/statsd_exporter)
can export the metrics to Prometheus.

* [collectd](https://collectd.org/) measures system metrics. With the
  [collectd exporter](https://github.com/prometheus/collectd_exporter)
  Prometheus can use the data.
  
* There are also alternatives which replace Prometheus, for example the
[TICK stack](https://www.influxdata.com/time-series-platform/).
  * Telegraf collects data and passes it on.
  * InfluxDB is a time series database.
  * Chronograf offers visualization and analysis.
  * Kapacitor takes care of alerts and anomaly detection.

* [Grafana](https://grafana.com/) provides graphical analysis for
  metrics. It can display data stored in Prometheus. Istio uses this
  approach to provide dashboards for microservices (see [section
  23.4](#section-service-mesh-monitoring)).

## 20.6 Experiments {#section-monitoring-experimente}

#### Experiments: Choosing Metrics

Which metrics are relevant for a microservice is different for each
microservice and also differs from project to project. The monitoring
can be adapted to your own requirements as follows.

* Consider a project you know. Which *metrics* are currently being
measured? Note: The technical metrics from operations are often easy
to find out about because they are processed in the familiar monitoring
applications. From a business perspective, tools for web analysis or
reports can be used. Ultimately, these are also metrics, even if they
are processed in other tools.

* Which *stakeholders* are there? Obvious stakeholders are development
and operations. However, the business side is also relevant. On the
business side, there can even be many different stakeholders if, for
example, several business divisions are interested in the
application. Perhaps
other groups such as QA are also interested.

* Together with stakeholders, you can find out which *further metrics*
are relevant. Microservices allow fast release cycles. In order to
better understand what should be changed and brought into production,
there must be more and better metrics, in particular business metrics.

* What changes with *microservices*? There will then be more servers,
more communication between the microservices over the network and a
clearer division based on domains. What impact does this have on the
metrics?

* Do the additional metrics and the larger number of
deployable artifacts result in a *data volume* that cannot be
processed with the current monitoring technologies?

#### Experiments: Extending Prometheus

The Prometheus installation can be extended in different ways.

* Instead of the static configuration, where each microservice must be
listed in the Prometheus configuration, service discovery with Consul
can be used. New services are then automatically included in
monitoring. For this, the following steps are necessary:

  - Delete the link from Docker
container `prometheus` to Docker container `order` in the file
`docker-compose-prometheus.yml`. Now access will
take place via Consul so that a Docker compose link to the
microservice is not necessary anymore.

  - Insert a link from Docker
container `prometheus` to Docker container `consul` into the file
`docker-compose-prometheus.yml`  so that Prometheus
can read the information about the services from Consul.

  - Delete the job `order` in the Prometheus configuration
`prometheus.yml` .

  - Create a new job `consul` in the Prometheus configuration
`prometheus.yml`. The
[documentation](https://github.com/prometheus/docs/blob/master/content/docs/operating/configuration.md#consul_sd_config)
states that instead of `scrape_configs` an element named
`consul_sd_configs` has to be created in which the `server` is then
defined. Because of the Docker Compose link the hostname of the Consul
server is `consul`. The port also has to be indicated. Therefore
`'consul:8500'` is correct.

   - Build the Docker images with `docker-compose -f
docker-compose-prometheus.yml build` and start them anew with
`docker-compose -f docker-compose-prometheus.yml up -d`.

* Only the *order* microservice provides metrics for Prometheus at the
moment. Thus, as an exercise the microservices *catalog* and
*customer* can also be
provided with Prometheus support. To do this, you must:

  - Insert the dependencies with `groupId`
  `io.prometheus` in `pom.xml`  so that the Prometheus client code is
  available.
  `pom.xml` from the order microservice should serve as template.

  - Copy the package `com.ewolff.microservice.order.prometheus` into
the other project and rename it there, for example to
    `com.ewolff.microservice.catalog.prometheus`.

  - If the Consul service discovery from the previous experiment is
used, Prometheus will automatically find the new service. Without
Consul, a new job 
must be created in the `scrape_configs` section of
`prometheus.yml`. The
order job can be a good template for this.

   - As described above, recompile the applications with `./mvnw clean
package` (macOS, Linux) or `mvnw.cmd clean package` (Windows) in the
directory `microservice-consul-demo`. Then build the Docker
containers with `docker-compose -f docker-compose-prometheus.yml
build` newly and start them again with `docker-compose -f
docker-compose-prometheus.yml up -d`. Now the metrics of another
microservice should be shown in the dashboard.
You can check it at <http://localhost:9090/targets>.

* Use the [node exporter](https://github.com/prometheus/node_exporter)
for displaying the metrics of the host in Prometheus. These are
metrics of the system like disk I/O.

* [cAdvisor](https://github.com/google/cadvisor) can be used for
reading the metrics of a
  Docker container. cAdvisor also can provide the metrics to
  [Prometheus](https://github.com/google/cadvisor/blob/master/docs/storage/prometheus.md).

* Monitor the Docker runtime environment with Prometheus. The
  [Docker documentation](https://docs.docker.com/engine/admin/prometheus/)
  shows how the monitoring can be implemented.

* Install [Grafana](https://prometheus.io/docs/visualization/grafana/)
as an alternative graphical frontend for Prometheus.

* Install the alertmanager. To do so, analogous to the Prometheus
image from directory
  `microservies-consul/docker/prometheus` you can create your own alertmanager Docker image
  which is based on the official
  [alertmanager Docker image](https://hub.docker.com/r/prom/alertmanager/tags/)
  and just adds a configuration. The [documentation of the configuration](https://prometheus.io/docs/alerting/configuration/)
  and [examples for alerts](https://prometheus.io/docs/alerting/notification_examples/)
  can help with creating your own configuration.

## 20.7 Conclusion {#section-monitoring-fazit}

Monitoring is essential for every type of microservice and is
demanding due to the high number of services. Central monitoring for all
services often has considerable advantages. Central monitoring should
therefore be considered for each microservices system.

Prometheus as a monitoring tool in microservices environments has the
advantage that the multidimensional model allows raw metrics to be
combined into metrics that show the behavior of all instances of a
specific microservice later on and thus enable metrics from the
user's point of view. The user uses not only one instance of a
microservice, but the entire system so such metrics provide a better
impression about what is happening in the system and how it matters
for users. Other technologies
can further support the monitoring of a microservices system or
replace Prometheus.

#### Advantages

* Prometheus polls data and protects itself against overload.

* Prometheus' multidimensional data can be evaluated in different
ways. For
example, all instances of a microservice can be summed up.

* Prometheus can be integrated with other solutions.

#### Challenges

* Prometheus covers an area for which most organisations already have
solutions. So retraining is necessary.

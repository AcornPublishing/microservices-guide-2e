# 9 Recipe: Server-side Integration using Edge Side Includes (ESI) {#chapter-esi}

Servers can also integrate multiple frontends. This chapter focuses on
[ESI](https://www.w3.org/TR/esi-lang) (Edge Side Includes). 
The readers get to know:

* How the web cache Varnish implements ESI.

* How applications can implement an integration using ESI.

* Which benefits and disadvantages ESI has and which alternatives
  to ESI exist for implementing server-side frontend integration.

## 9.1 ESI: Concepts {#section-esi-konzepte}

ESI (Edge Side Includes) enables web applications to integrate HTML
fragments of another web application (see
[figure 9-1](#fig-esi-konzept)). To do so, the web application sends
HTML containing ESI tags. The ESI implementation analyzes the ESI tags
and integrates HTML fragments of other web application at the right
positions.

{id="fig-esi-konzept"}
![Fig. 9-1: Integration with ESI](images/esi-konzept.png)

#### Caches Implement ESI

In the example, the web cache [Varnish](https://varnish-cache.org/)
serves as ESI implementation. Also other caches such as
[Squid](http://www.squid-cache.org/) support ESI. Websites use these
caches to deliver web pages out of the cache upon incoming requests.
The servers handle requests only for cache-misses. This speeds up the
website and decreases the load of the web servers.

#### CDNs Implement ESI

Content Delivery Networks (CDNs) such as
[Akamai](http://www.akamai.com/html/support/esi.html) also implement
the ESI standard.  In principle, CDNs serve to deliver static HTML
pages and images. To do so, CDNs run servers at several Internet
nodes so that every user can load web pages and images from a nearby
server and the loading times are reduced. Via the support of ESI the
assembling of HTML fragments can be done on a server which is close to
the user.

CDNs and caches implement ESI to be able to assemble web pages from
different fragments. Static parts can be cached, even if other parts have
to be dynamically generated. This makes it possible to at least
partially cache
dynamic web pages which otherwise would have to be excluded from
caches completely. This improves performance. Therefore, ESI does not
only offer features for frontend integration, but also features which
are especially useful for caching.

## 9.2 Example {#section-esi-beispiel}

The [example](https://github.com/ewolff/SCS-ESI) shows how 
Edge Side Includes (ESI) can be used to assemble HTML fragments from different
sources and how the entire HTML can be sent to the browser. For this, the HTML contains
special ESI tags which are replaced by HTML fragments.

{id="fig-esi-beispiel"}
![Fig. 9-2: Overview of the ESI Example](images/esi-beispiel.png)

[Figure 9-2](#fig-esi-beispiel) shows an overview of the structure of
the example. The Varnish cache directs the HTTP request to the *order*
microservice or the *common* service. The *order* microservice contains
the logic for processing orders. The *common* service offers CSS assets
and HTML fragments which microservices have to integrate in their HTML
pages. The example therefore shows a typical scenario. The
applications
like *order*
deliver content which is displayed in a frame which all applications
can uniformly integrate and which is provided by *common*.

{id="fig-esi-screenshot", width=60%}
![Fig. 9-3: Screenshot of the ESI Example](images/esi-screenshot.png)

[Figure 9-3](#fig-esi-screenshot) shows one page of the ESI
example. The links to the home page and Thymeleaf and also the date
are provided by the *common* service. Also the CSS and therefore the
layout originate from the *common* service. The *order* service only
provides the list of orders. Thus, when additional microservices have
to be integrated into the system, they only have to return the
respective information in the middle. The frame and the layout is
added by the
common service.

During a reload the time is updated. However, this is only done every
30 s since the data is cached for this long. The cache only works if
no cookies were sent in the request.

#### Running the Example

[Section 0.4](#section-einleitung-quick-start) describes which
software has to be installed for the example.

To start the example, download the code with `git
clone https://github.com/ewolff/SCS-ESI.git`. Afterwards, you have to
compile the Java application in sub directory `scs-demo-esi` with
`./mvnw clean package` (macOS, Linux) or `mvnw.cmd clean
package` (Windows).
See [appendix B](#anhang-maven) for more details on Maven and how to
trouble shoot the build.
Finally, you can build the Docker images in the
directory `docker` with
`docker-compose build` and start the example with `docker-compose up
-d`.
See [appendix C](#anhang-docker) for more details on Docker,
docker-compose and how to troubleshoot them.

<https://github.com/ewolff/SCS-ESI/blob/master/HOW-TO-RUN.md> explains
the installation procedure and how to start the example in more
detail.

On the Docker host at port 8080 Varnish is available, which receives
the HTTP requests and processes the ESI tags.  If the Docker containers are
running on the local computer, you can reach Varnish at
<http://localhost:8080>. The web pages of the order microservice can
be accessed at <http://localhost:8090>.
These web pages contain the ESI tags and therefore appear broken if
displayed in a web browser.

## 9.3 Varnish {#section-esi-varnish}

[Varnish](https://varnish-cache.org/) is a web cache and is used as
ESI implementation in the example.

Varnish is mainly used for optimizing web servers. Varnish intercepts HTTP
requests to web servers. Varnish caches the
responses and forwards only those requests  to the web servers that are
not in the cache.  This improves the performance.

#### Licence and Support

Varnish is licensed under a
[BSD license](https://github.com/varnishcache/varnish-cache/blob/master/LICENSE).
The cache is mainly developed by
[Varnish Software](https://www.varnish-software.com/), which also
provides commercial support.

#### Caching with HTTP and HTTP Headers

Caching data correctly is not trivial. Above all, the question
is when content can be retrieved from the cache and when the data must
be retrieved from the web server, because the data in the cache no
longer represents the current state. Varnish uses its own HTTP headers
for this. The HTTP protocol has a very good support for caching
through its HTTP headers. The control over whether data is cached or
not rests with the web server, which informs the cache of the settings
via HTTP headers. Only the web server can decide whether a page can be
cached or not since that depends on the domain logic.

#### Varnish Docker Containers

In the example, Varnish runs in a Docker container which contains a
Ubuntu 14.04 LTS. On it, a Varnish from the official Varnish repository 
will be installed. 

#### Varnish Configuration

Varnish offers a powerful configuration language. For the
example, Varnish is installed in its own Docker container.
The configuration can be found in the file `default.vcl` in the directory
`docker/varnish/` in the example. Here is an extract with the essential settings.

{linenos=off}
~~~~~~~~
vcl 4.0;

backend default {
    .host = "order";
    .port = "8080";
}

backend common {
    .host = "common";
    .port = "8180";
}

sub vcl_recv vcl_recv {
    if (req.url ~ "^/common") {
       set req.backend_hint = common;
    }
}

sub vcl_backend_response{
    set beresp.do_esi = true;
    set beresp.ttl = 30s;
    set beresp.grace = 15m;
}
~~~~~~~~

* `vcl 4.0;`chooses the version 4 of the Varnish configuration language.

* The first backend has the name `default`. Each request arriving at
  the Varnish cache is passed on to the web server if there is no
  other configuration. The hostname `order` is resolved by Docker
  compose.  The `default` backend is the *order* microservice, which
  implements all functions to accept and display orders.

* The second backend has the name `common` and is provided by the host
  of the same name. Also in this case, Docker compose resolves the
  name to a Docker container. The `common` service provides headers and
  footers for the HTML pages of the microservices and
  [Bootstrap](http://getbootstrap.com/) as a shared library for the UI
  of the microservices.

* When an HTTP request arrives for a URL where the path starts with 
  `/common`, the HTTP request is redirected to the `common` backend. 
 The code of the subroutine `vcl_recv` is responsible for this. 
 Varnish automatically calls this routine to determine the routes for the requests.

* The subroutine `vcl_backend_response` configures Varnish with
 `beresp.do_esi` so that Varnish interprets ESI tags. `beresp.ttl`
 turns on the caching.  Each page is cached for 30 seconds. Finally,
 `beresp.grace` ensures that if a backend fails, the web pages are
 cached for 15 minutes. This can temporarily compensate for a backend
 failure.
 Of course this only works if the web page is already in cache because
 it has been accessed before the failure. So if the page is not in the
 cache or not cacheable at all, this feature does not help. 
 However, this caching is very simple. If a new order has
 been created, it is not displayed until the cache has been
 invalidated. This can take up to 30 seconds. It would of course be
 better if a new order leads to invalidation of the cache. This is
 still relatively easy to implement in the example, but in a complex
 application it can be difficult to invalidate the correct
 pages. For example, goods are displayed on product pages and
 order pages. So several pages need to be invalidated if the data for
 the goods is changed.  Therefore, simple time-based caching can be the better
 solution. It is easy to implement and might do a good enough job.
 There is a
 [chapter](https://book.varnish-software.com/4.0/chapters/Cache_Invalidation.html)
 about cache invalidation in the Varnish book.

The present configuration not only enables ESIs, but also implements
the functionality of a reverse proxy. The configuration redirects
requests to specific microservices.

#### Load Balancing

It is also possible to add [load
balancing](https://varnish-cache.org/trac/wiki/LoadBalancing) to the
Varnish configuration. In that case the requests to the microservices
can be split across multiple microservices by defining multiple
`backends` in the configuration. However, it is of course also
possible to rely on an external load balancer. In that case, Varnish
would only do ESI and caching.

#### Evaluation of VCL

As you can see, VCL is a very powerful language that has many
possibilities for manipulating HTTP requests. That is also
necessary. For example, you can only be sure that a request can be
cached if it does not contain cookies since cookies could change the
response. Therefore, VCL must be able to remove cookies, for example.

A comprehensive documentation of Varnish and VCL can be found in the
free
[Varnish Book](http://book.varnish-software.com/).

#### Order Microservice

The *order* microservice offers a normal web interface, 
which, however, has been supplemented with ESI tags in some places.

A typical HTML page of the *order* microservice looks like this.

{linenos=off, lang="html"}
~~~~~~~~
<html>
<head>
  ...
  <esi:include src="/common/header"></esi:include>
</head>

<body>
  <div class="container">
    <esi:include src="/common/navbar"></esi:include>
    ...
  </div>
  <esi:include src="/common/footer"></esi:include>
</body>
</html>
~~~~~~~~

#### HTML with ESI Tags in the Example

The *order* microservice is available at port 8090 of the Docker host. 
The output goes past the Varnish and still contains the ESI tags. 
At <http://localhost:8090/> the HTML with the ESI tags can be viewed.

The ESI tags look like normal HTML tags.  They only have an `esi`
prefix. Of course, a web browser cannot interpret them.

#### ESI Tags in the HTML Head

In the `head` the ESI tags are used to integrate common assets like 
bootstrap into all pages. Changing the header under `"/common/header"` 
causes all pages to get new versions of bootstrap or other libraries. 
If the pages with a new version are no longer displayed correctly, such 
a change will cause problems. Therefore, the pages themselves should be 
responsible for using new versions. For this, a version number can be 
encoded in the URL in the ESI tag for example.

#### ESI Tags in the Remaining HTML

The ESI Include for `"/common/navbar"` ensures that each web page
has the same navigation bar. Finally, `"/common/footer"`
can contain scripts or a footer for the web page.

#### Result: HTML at the Browser

Varnish collects these HTML snippets from the common service
so that the browser receives the following HTML:

{linenos=off, lang="html"}
~~~~~~~~
<html>
<head>
...
  <link rel="stylesheet"
   href="/common/css/bootstrap-3.3.7-dist/css/bootstrap.min.css" />
  <link rel="stylesheet"
   href="/common/css/bootstrap-3.3.7-dist/css/bootstrap-theme.min.css" />
</head>

<body>
  <div class="container">
    <a class="brand"
     href="https://github.com/ultraq/thymeleaf-layout-dialect">
     Thymeleaf - Layout </a>
    Mon Sep 18 2017 17:52:01 </div></div>
    ...
  </div>
  <script src="/common/css/bootstrap-3.3.7-dist/js/bootstrap.min.js" />
</body>
</html>
~~~~~~~~

The ESI tags have thus been replaced by suitable HTML snippets.

ESI offers many other features, for instance, for securing a system against
the failure of a web server or for integrating HTML fragments 
only under certain conditions.

#### No Tests without ESI Infrastructure

A problem with the ESI approach is that individual services cannot be tested 
without an ESI infrastructure. At the very least, they do not display any pages 
with meaningful content, as the ESI tags would have to be interpreted for that. 
This only works if the HTTP requests are routed through Varnish. Therefore, 
suitable environments containing a Varnish must be provided for the development.

####  Effects on the Application

The application is a normal Spring Boot web application without any
dependencies on Spring Cloud or ESI. This shows that pure frontend
integration leads to a very loose coupling and has little impact on
the applications.

#### Common Microservice

In the example, the common service is a very simple
[Go](https://golang.org/) application. It handles the three URLs
`"/common/header"`, `"/common/navbar"` and `"/common/footer"`. For
these URLs, the Go code generates suitable HTML fragments.

#### Asset Server

The Go code also contains a web server that provides static resources
under `"/common/css/"` - the Bootstrap framework. In this way, the
common microservice assumes the function of an asset server.  Such a
server offers CSS, images or JavaScript code to applications. The ESI
example shows an alternative for the integration of shared assets. In
[chapter 8](#chapter-links-javascript) a common asset project has
ensured that all applications can use the same assets. In the example
in this chapter, an asset server is used for this purpose.

The application displays the current time in the navigation bar.  This
shows that dynamic content can also be displayed with ESI includes.
[Section 5.4](#section-technisch-mikro-go) has already explained how
this part of the system functions and can be built.

## 9.4 Recipe Variations {#section-esi-variationen}

Of course, instead of Varnish, a different ESI implementation could be used, 
for example, by [Squid](http://www.squid-cache.org/) or by a 
CDN like [Akamai](https://www.akamai.com/us/en/support/esi.jsp).

#### SSI

Another option for server-side frontend integration is
[SSI](https://en.wikipedia.org/wiki/Server_Side_Includes)(Server-side
includes).  This is a feature that most web servers offer.
<https://scs-commerce.github.io/> is an example of a system that uses
SSI with the nginx web server to integrate the frontends.

SSI and ESI have different benefits and disadvantages.

* Web servers are often already available in the infrastructure for
 SSL/TLS termination or for other reasons. Since web servers can
 implement SSI, compared to ESI no additional infrastructure such as a
 Varnish is necessary.

* Caches cannot only speed up applications, but can also compensate for
 web server failures for some time, thus improving resilience. This
 speaks for ESI and a cache like Varnish. ESI also has more features
 for further optimizing caching.  However, correct caching can also be
 difficult to implement. For instance, changing the data of a single
 data record can trigger a cascade of invalidations.  Finally, every
 page and every HTML fragment containing information about the goods
 must be regenerated.

#### Tailor

[Tailor](https://github.com/zalando/tailor) is a system for
server-side frontend integration that Zalando implemented as part of
[Mosaic](https://www.mosaic9.org/). It is optimized for showing the user the
first parts of the HTML page as quickly as possible.  For
e-commerce the rapid display of a web page is very
important to keep users and can increase sales.  To achieve this, Tailor
implements a
[BigPipe](https://de-de.facebook.com/notes/facebook-engineering/bigpipe-pipelining-web-pages-for-highperformance/389414033919/).
First, a very simple HTML is transferred to the user in order to be able to
display a simple page very quickly. JavaScript is used to load more
details step-by-step.  Tailor implements this with asynchronous I/O
using Node.js streams.

#### Client-side Integration

Client-side integration does not use any additional infrastructure 
and can be the simpler option for frontend integration. Therefore, it 
makes sense to find out how far you can go with client-side integration 
before using server-side frontend integration.

For an integration of a header and footer as in the example for this chapter, 
server-side integration is the better choice because a page cannot be 
displayed without these elements. The pages should be delivered to the user
in such a way that they can be displayed to the user without loading any
additional content.

Therefore, client-side integration for optional elements makes sense. 
Dealing with failed services is then a task for the client code.
That might simplify the server implementation and the server setup.

#### Shared Library

The example from [chapter 8](#chapter-links-javascript) uses a library
to deliver assets. Theoretically, the HTML fragments that are
integrated with ESI in this example could also be delivered as a
library. But then all systems would have to be rebuilt and deployed
for an additional link in the navigation.  With the ESI solution, all
you have to do is change the HTML fragment on the server.

#### Additional Integration

It is unlikely that pure frontend integration is
enough. Therefore, a system will combine backend integration with
synchronous (see [chapter 13](#chapter-synchron)) or asynchronous (see
[chapter 10](#chapter-asynchron)) communication mechanisms with
frontend integration. An exception is a scenario like in
[chapter 8](#chapter-links-javascript), where a complex portal
application is implemented. The parts of the portal can communicate
through frontend integration.  Backend integration is not 
necessary because the services do not implement a lot of logic and
have no database, but only create a web interface.

## 9.5 Experiments {#section-esi-experimente}

* Supplement the system with an additional microservice.
  * A microservice which simply displays a static HTML page can serve
    as example.
  * Of course, you can simply copy and modify the existing Go or the
   Spring Boot microservice.
  * The new microservice should integrate header, navigation bar and
    footer of the common microservice via ESI tags.
  * Package the microservice as a Docker image and reference it in
     `docker-compose.yml`. There you can also determine the name of
     the Docker container.
  * The microservice has to be accessible via the Varnish.  To achieve
    that you have to integrate a new backend with the name of the
    Docker container in `default.vcl` and adapt the routing in
    `vcl_recv()`.
  * Now you should be able to access the new microservice at e.g.
    <http:localhost:8080/name> if the Docker containers run on the
    local computer and the new service is configured in Varnish’s routing
    with `name`. The ESI tags should have been replaced by
    HTML code.

* The Go application only returns a dynamic HTML fragment -- the
 navigation bar with the current time. Instead of the Go application,
 a web server can also deliver static pages. For example, replace the
 Go application with an Apache httpd server that delivers the HTML
 fragments and the bootstrap library. The time does not necessarily
 have to be displayed in the navigation bar, so a static HTML fragments
 is all that is needed.

* Change the caching so that the pages are immediately invalidated
 when new orders are received. You can for example use matching HTTP
 headers as explained in a
 [chapter](http://book.varnish-software.com/4.0/chapters/HTTP.html) of
 the Varnish book. An alternative is to remove objects directly from
 the cache.  The Varnish book also contains a
 [chapter](http://book.varnish-software.com/4.0/chapters/Cache_Invalidation.html)
 on this option.

* Replace the Varnish cache in the example with [Squid](http://www.squid-cache.org/), 
which also implements ESI.

* Replace ESI with SSI and replace the Varnish cache with an Apache httpd or nginx.

* What happens if the web servers fail? Simulate the failure with
 `docker-compose up --scale common=0` or `docker-compose up --scale
 order=0`. Which parts of the web page still work? Is it still
 possible to place orders, for example?

## 9.6 Conclusion {#section-esi-fazit}

ESIs are a possible implementation of frontend integration and 
lead to loose coupling. The applications are simple web applications that, 
apart from the ESI tag, have no dependencies on the infrastructure.

The integration with ESIs has the advantage that the web pages are completely 
assembled by the cache and can be displayed directly in the browser. 
It is therefore not possible that the page is delivered, but cannot be 
used by the user because some fragments still have to be reloaded.

Using a cache together with ESI has the advantage that fragments can be cached. 
This means that not only static pages but also static parts of dynamic pages 
can be cached, which improves performance. The pages can even be cached and 
assembled from a CDN that supports ESI, further improving performance.

The cache can also be used to achieve a certain degree of resilience. 
If a web server fails, the cache can return the old data. In this way, 
the web page remains available but might potentially return invalid information. 
However, for this the cache must hold the pages for a long time. In addition,
the cache must also check the availability of the services.

#### Benefits

* Web page is always delivered in entirety 
* Resilience via cache
* Higher performance via cache
* No code in the browser

#### Challenges

* Uniform look & feel
* Additional server infrastructure necessary

# 8 Recipe: Links and Client-side Integration {#chapter-links-javascript}

This chapter provides an example for a frontend integration with 
links and JavaScript.

The reader learns in this chapter:

* For which scenarios a simple approach based on links and JavaScript for frontend 
  integration is feasible.

* The example in this chapter is not a self-contained system (SCS). This
  shows that frontend
  integration does not only make sense in the context of
  SCS, but also in other scenarios.

* The example shows how an integration with links and JavaScript can
  be implemented.

* With which mechanisms a uniform look & feel can be implemented with
  such an integration.

## 8.1 Overview {#section-links-javascript-ueberblick}

The example in this chapter is a classic assurance application
designed to help office workers interact with customers. It was
created as a prototype for a real insurance company. The example shows
how such a rather classic application can be implemented as a web
application with frontend integration.

The example shows the division of a system into several web
applications and the integration of these applications. ROCA is used
as basis for this.  This is an approach for implementing web
applications (see [section 7.3](#section-frontend-roca)), which has a
number of fundamental advantages especially for frontend integration.

The example was created as a prototype showing how a web application
can be implemented with ROCA and which advantages it offers. The INNOQ
employees Lucas Dohmen and Marc Jansing implemented the example.

#### Search

The assurance application is available at
 <https://crimson-portal.herokuapp.com/>. It can also be run on a 
 local computer as a Docker container (see
 [section 8.2](#section-links-javascript-beispiel)).

The application is in German. However, nowadays browsers can translate
web pages into other languages. The screenshots were done with
the Firefox browser and Google Translator's translation from German to English.

On the main page there is an input line to search for customers (see
[figure 8-1](#fig-links-javascript-screenshot)).
While the user enters the customer's name, matching names are
suggested. For this, the frontend uses jQuery.

{id="fig-links-javascript-screenshot", width=60%}
![Fig. 8-1: Screenshot Insurance Application](images/links-javascript-screenshot.png)

#### Postbox

Another functionality is the *postbox*. With a click on the postbox
icon on the main page, the user gets an overview of the current
news. The overview is displayed in the current main page with
JavaScript (see [Figure 8-2](#fig-links-javascript-postbox)).

{id="fig-links-javascript-postbox"}
![Fig. 8-2: Overlayed Postbox](images/links-javascript-postbox.png)

The entire application uses a uniform frontend. However, if you look
at the address line, you will notice that several web applications are
used. There is one web application each for the main application
(<https://crimson-portal.herokuapp.com/>), for reporting damages
(*damage* application) (<https://crimson-damage.herokuapp.com/), for
writing letters (*letter* application)
(<https://crimson-letter.herokuapp.com/>) and for the postbox
(*postbox* application)(<https://crimson-postbox.herokuapp.com/>).
Nevertheless, the frontend has the same look & feel for all these
applications.

#### Structure of the Application

{id="fig-links-javascript-ueberblick"}
![Fig. 8-3: Overview of the Insurance Example](images/links-javascript-ueberblick.png)

[Figure 8-3](#fig-links-javascript-ueberblick) shows how the different
applications are integrated.

* The *main* application is used to search for customers and to
 display the basic data of a customer. The code is available at
 <https://github.com/ewolff/crimson-portal>.  There you will also find
 instructions on how to compile and start the application.  The
 application is written in Node.js.

* The *assets* at <https://github.com/ewolff/crimson-styleguide>
 contain artifacts that are used by all applications to achieve a
 consistent look & feel.  The other projects refer to this project in
 the `package.json`. This allows the npm build to use the artifacts
 from this project. npm is a build tool specialized in JavaScript. The
 asset project includes CSS, fonts, images, and JavaScript
 code. Before the assets are used in the other projects, the build
 optimizes them in the asset project. For example, the JavaScript code
 is minified.

* The *damage* application for reporting a damage is also written in
  Node.js.  The code can be found at
  <https://github.com/ewolff/crimson-damage>.

* The *letter* application is written in Node.js as well. The code is
  available at <https://github.com/ewolff/crimson-letter>.

* The code for the *postbox* can be accessed at
  <https://github.com/ewolff/crimson-postbox>. The postbox is
  implemented with Java and Spring Boot. To be able to use the shared
  asset project, the build is divided into two parts. The Maven build
  compiles the Java code, while npm is responsible for integrating the
  assets. npm then copies the assets into the Maven build.

* Finally, the *backend simulator* can be found at
  <https://github.com/ewolff/crimson-backend>. It receives REST calls
  and returns data regarding customers, contracts etc.  This simulator
  is also written in Node.js.

#### Why Monolithic Backend?

So this application uses a monolithic backend and frontend
microservices. Because the microservice lack logic, they are not
Self-contained Systems (see [chapter 6](#chapter-scs)). However, this
architecture can still make sense. At least the frontend consists of
microservices. So independent development of the microservices is
possible. Also it is possible to use different technologies in each
frontend microservice. With the large number of available UI
frameworks and the high speed of innovation, that is a clear
advantage.

Also it is probably not possible to migrate the backend into
microservices. Another team might be responsible for it. So if the
scope of the project is just to improve the frontend, there is just no
way the architecture of the backend can be changed.

While SCS are generally a great idea, this example shows one exception
from the rule.

#### Integration with Redirects

If a user in the *damage* application enters a claim for a car, the
user is sent back to the overview of the car displayed by the portal.
The transition from the *damage* application to the portal is
implemented with a redirect. The *damage* application sends an HTTP
redirect after reporting the claim, leading to the web page of the
*main* application.

A redirect is a very simple integration. The
*damage* application only needs to know the URL for the redirect. The
portal could even pass this URL to the *damage* application to further
decouple the two applications.

Such an integration is also used if a user registers with his / her
Google account on a web page. After the user agrees to register on the
Google web page, the Google page sends a redirect back to the initial
web page.

#### Integration with Links

The integration of the applications is mainly done via links such as
<https://crimson-letter.herokuapp.com/template?contractId=996315077&partnerId=4711>
to display a web page for writing a letter.

They contain all the essential information necessary for the web page
to write the letter. The contract number and the partner number. In
this way, the *letter* application can retrieve the data from the
backend simulator and render it in the web page. The coupling between the
main application and the letter application is very loose.
It's just a link with two parameters. The main application does not
need to know what is behind the link. In this way, the *letter*
application can change its UI at any time without any impact on the
*portal* application. However, all applications
use a common database from the backend simulator and are consequently
nevertheless tightly coupled since a change to the data would affect
the backend and the respective microservice.

#### Integration with JavaScript

In the example, the integration of the frontends is practically always
done via links. 
However, the *postbox* displays an 
overview of the current messages in the *main* application. For this,
a simple link is not enough.
Still, this integration is also a link. A look into the HTML code [^Ger]
shows.

{linenos=off, lang="html"}
~~~~~~~~~~~~~~~
<a href="https:&#x2F;&#x2F;crimson-postbox.herokuapp.com/m50000/messages"
  class="preview" data-preview="enabled"
  data-preview-title="Notifications"
  data-preview-selector="table.messages-overview"
  data-preview-error-msg="Postbox unreachable!"
  data-preview-count="tbody>tr" data-preview-window>
~~~~~~~~~~~~~~~

[^Ger]: Actually, when running the application the HTML code contains German expressions. They can be translated by Google Translate. However, the HTML code remains unchanged and still includes German expressions. For convenience, the English expressions are shown in the HTML code in the listing here.

The link contains additional attributes. They ensure that the
information of the *postbox* is displayed in the current web page and
define how exactly this happens. This information is interpreted by
less than 60 lines of JavaScript code from the asset project (see
<https://github.com/ewolff/crimson-styleguide/tree/master/components/preview>).
The code uses jQuery, so every application in the system now needs to
use a compatible version of jQuery when using this code from the asset
project. However, this is not mandatory. Alternatively, any microservice
that integrates the *postbox* with such a link can write its own code
to interpret the link. After all, every microservice writes its own code to
read data from JSON that is provided by other microservices, for example.
So code to integrate other projects' HTML should be fine, too.

This type of integration is called *transclusion* because it includes
HTML from more than one microservice in a webpage. In this example, the
transclusion is implemented with JavaScript code that integrates the
different backends (see
[figure 8-4](#fig-links-javascript-konzept)). The JavaScript code runs
in the browser, loads HTML fragments of the other web applications, and displays
them in the current web page.

{id="fig-links-javascript-konzept"}
![Fig. 8-4: Integration with JavaScript](images/links-javascript-konzept.png)

#### Presentation Logic in the Postbox

With transclusion, *the postbox* retains control over how messages are
displayed, even if the messages are shown as previews in another
service. This leads to a clean architecture. The code for displaying
the postbox is located in the postbox service, even if the
postbox is displayed in another service.  This shows how frontend
integration can contribute to an elegant solution and architecture.

The approach of dynamically including content from other URLs is not
only used for the *postbox*. For each customer there is also an
overview of the offers, applications, claims, and contracts. The links
for this are located below the postbox icon and also once again
further down in the inventory. Just as with the postbox, this
information is also referenced
with links whose contents are displayed on the web page by the
JavaScript code. In this case, the URLs are in the same microservice,
but still provide a better modularization. It also shows that the
code solves a general technical problem and is reusable beyond the
integration between microservices.

#### Assets with Integrated HTML

Transclusion embeds HTML fragments into other web pages. The HTML can
require assets such as CSS or JavaScript. There are different
approaches for ensuring that these assets are present.

* The HTML can be designed to not require any assets at all. Thus, no
  assets have to be shared between the microservices.

* The HTML uses only the assets from the shared asset library, in the
 example `crimson-styleguide`. Then no special measures are necessary
 either because the assets are present in all microservices anyway.

* The HTML can also bring along its own assets or link to them.
 However, in this case you have to be careful not to load the assets
 more than once if several contents are transcluded in one web page.

Till Schulte-Coerne has written a
[blog post](https://www.innoq.com/en/blog/transclusion/) about this
topic.

#### Resilience

The application achieves a high degree of resilience and reliability
through the consistent use of links. If one microservice fails, the
other microservices continue to work.  They can still display the
links. The JavaScript code contacts the *postbox* to transclude an
overview of the messages in the web page. If this doesn't work, the
code displays an exclamation mark -- but the application still works.

Thanks to JavaScript, loading the transcluded HTML is carried out in
the background so
that the failure of one system does not affect the transclusion of the
other contents and high performance is achieved.

#### With and Without JavaScript

The user can also use the applications if JavaScript is disabled.  In
this case, for example, the automatic completion of customer names
does not work and also displaying the overview of messages in the
*postbox* is no longer functional.  Nevertheless, the application
remains usable. The start page is rendered completely as HTML on the
server and does not require client-side templates.  The *postbox*
simply offers a link instead of the displayed overview.  If the user
clicks on this link, he or she gets to the *postbox*.

## 8.2 Example {#section-links-javascript-beispiel}

[Section 0.4](#section-einleitung-quick-start) describes which
software has to be installed to start the example.

This example is not only provided in the Heroku cloud, but also as
collection of
[Docker containers](https://github.com/ewolff/crimson-assurance-demo)
so that you can run it on a local computer.  To do so, you first have
to download the code with `git clone
https://github.com/ewolff/crimson-assurance-demo`. Then you need to
change into the newly created directory `crimson-assurance-demo` with
`cd crimson-assurance-demo`. `docker-compose up
-d` generates all necessary Docker
images with the help of Docker Compose (see
[section 4.6](#section-docker-docker-compose)) and starts them. 
This takes quite a while because all the Docker containers are built
and also all the dependencies are downloaded from the Internet.
If the Docker containers do not run on
`localhost`, the hostname of the server has to be assigned to
the environment variable `CRIMSON_SERVER`.
This is necessary to make the links work.
See [appendix C](#anhang-docker) for more details on Docker,
docker-compose and how to troubleshoot them.

A more detailed description of how the example can be started, can be found at
<https://github.com/ewolff/crimson-assurance-demo/blob/master/HOW-TO-RUN.md>.

#### Network Ports

The application is available at port 3000 on the Docker host, i.e. at
<http://localhost:3000>. *Postbox* has the port 3001, *letter* the
port 3002 and *damage* the port 3003 (see
[figure 8-5](#fig-links-javascript-beispiel)). The frontend services
communicate with the backend, which runs in a separate Docker
container.

{id="fig-links-javascript-beispiel"}
![Fig. 8-5: Docker Containers in the Example](images/links-javascript-beispiel.png)

Of course, the ports of the Docker containers could be redirected to
any other ports of the Docker host. Likewise, all applications in the
Docker containers can use the same ports in each Docker
container. However, if the port
numbers of the containers are identical to those of the hosts, as in
the example, this is the least confusing.

You can also try the system directly on the web at
[Heroku](https://crimson-portal.herokuapp.com/). The links then point
to the separate applications for *postbox*, *letter* and
*damage* that are also deployed at Heroku. Heroku is a PaaS (Platform
as a Service, see [chapter 18](#chapter-paas)) available
in the public cloud.

## 8.3 Variations {#section-links-javascript-variationen}

The example uses a Node project for the shared assets. An alternative
option is an asset server which stores the assets. Since the assets
are static files which are loaded via HTTP, an asset server can just
be a simple web server.

Over time, the assets will change. For the asset project from the
example, a new version of the asset project would have to be created.
The new version must be integrated in every microservice. This sounds
like an overhead, but this way each application can be tested with a
new version of the assets.

Therefore, even with an asset server, a new version of the assets
should not simply be put into production, but the
applications should be adjusted to the new version and also tested
with the assets. The version of the assets can be included in the URL
path.  Thus version 3.3.7 of bootstrap might be found under
`/css/bootstrap-3.3.7-dist/css/bootstrap.min.css`.  A new version
would be available under a different path
e.g. `/css/bootstrap-4.0.0-dist/css/bootstrap.min.css`.

#### Simpler JavaScript Code

The JavaScript code in the example is quite flexible and can also deal
with the failure of a service. A primitive alternative is shown in the
[SCS jQuery Project](https://github.com/ewolff/SCS-jQuery/). In
essence, it uses the following JavaScript code.

{linenos=off, lang="JavaScript"}
~~~~~~~~
$(document).ready(function() {
  $("a.embeddable").each(function(i, link) {
    $("<div />").load(link.href, function(data, status, xhr) {
      $(link).replaceWith(this);
    });
  });
});
~~~~~~~~

The code uses jQuery to search for hyperlinks (`<a ...>`) with the CSS
class `embeddable` and then replaces the link with the content the
link refers to.

This demonstrates how simple it is to implement a client integration
with JavaScript.
At the minimum it is just seven lines of jQuery code.

##### Integration using other Frontend Integrations

Of course, client-side integration and links with
server-side integration (see [chapter 9](#chapter-esi))
can be combined. Both approaches have different benefits:

* Web pages with server-side integration originate entirely from the
  server. Thus, the integration makes sense when the web page can only
  be correctly displayed if all the content is integrated.

* With client-side integration transclusion is not executed when the
  other server is not available. The web page is displayed
  nevertheless, just without transclusion.  This can be the better
  option.
  It improves resilience. However, the webpage would need to be usable
  without the transcluded content. 
  
However, a server-side integration requires additional infrastructure.
For client-side integration, this is not necessary. Therefore, it
makes sense to start with client-side integration and to supplement
server-side integration when necessary.

#### Other Integrations

Synchronous communication ([chapter 13](#chapter-synchron)) or
asynchronous communication ([chapter 10](#chapter-asynchron)) enable
the communication of the backend system and therefore can be combined,
of course, with client-side frontend integration.

## 8.4 Experiments {#section-links-javascript-experimente}

* Start the example and disable JavaScript in the browser. Is the
  example still usable? Specifically, does the *postbox* integration
  still work?

* Analyze the JavaScript code for transclusion at
  <https://github.com/ewolff/crimson-styleguide/tree/master/components/preview>. How
  difficult is it to replace this code by an implementation with a
  different JavaScript library?

* Supplement the system with an additional microservice.
  * A microservice which generates a note for a meeting with a client
    can serve as example.
  * Of course, to add the service you can copy and modify one of
    the existing
    Node.js or the Spring Boot microservice.
  * The microservice has to be accessible by the *portal* microservice.
    To achieve this, you have to integrate a link to the new
    microservice into the portal.
  * The link can provide the partner ID to the new
    microservice. This ID identifies the customer and might be useful
    to figure out which customer the note belongs to.
  * After entering the note the microservice can trigger a redirect
    back to the portal.
  * For a uniform look & feel, you have to use the assets from
    the styleguide project. The Spring Boot project for the *postbox*
    shows the integration for Spring/Java and the portal for
    Node.js. Of course, you can also use other technologies for the
    implementation of the new microservice.
  * The microservice can store the data concerning the meeting in a
    separate database.
  * Package the microservice in a Docker image.
  * Reference the Docker image in `docker-compose.yml`.

## 8.5 Conclusion {#section-links-javascript-fazit}

This example system is deliberately presented in the first chapter on
frontend integration.  It shows how much is already possible with a
simple integration via links.  Only for the *postbox* you need some
JavaScript. Before using the advanced technologies for frontend
integration, one should first understand what is already possible with such a simple approach.

The example integrates different technologies. In addition to the
Node.js systems, there is also a Java/Spring Boot application which
seamlessly integrates into the system. This demonstrates that a
frontend integration results only in few limitations regarding the
technology choice.

#### ROCA

ROCA helps especially with this type of integration.  The
microservices can be accessed via links. This makes integration very
easy.  At the same time the applications are largely decoupled in
terms of deployments and technology. If an application is to be
deployed in a new version, this is easily possible and does not affect
the other applications.  The applications can also be implemented in
different technologies.

At the same time, the ROCA UI is comfortable and easy to use. Compared
to a single page app (SPA), there are no compromises in user comfort.

#### Assets

Finally, the application shows how to handle assets. In this case by
using a common Node. js project. As a result, each application can
decide for itself when to adopt a new version of the assets.  This is
important because otherwise a change of assets is automatically
rolled out to all applications and might cause problems in the
applications.
However, several versions of the assets should only be used
temporarily on the web page. After all, the design and the look & feel
should be uniform. Dealing with the asset project in such a way
that not all services always use the current version is only meant to
minimize the risk of an update, but must not lead to long-term
inconsistencies.

However, the asset project also ensures that all web pages contain
jQuery in the version used by the asset project.  Thus, the asset
project limits the freedom of individual projects with regards to
JavaScript libraries.

#### Self-contained Systems

Unlike self-contained systems (see [section 3](#chapter-scs)) this
solution uses a common backend. With an SCS, the logic should also
be part of the respective SCS and not be implemented in another system.
However, it is still possible to use some SCS ideas even if all
systems share a common backend. The systems do not have to deal with
logic and storing data as much as an SCS because they are in
the backend. Thus, this system shows how a well modularized portal
for a monolithic backend can be implemented.

But this approach also presents challenges. Any changes to the system
will probably affect one of the frontend applications and also the
backend. Then the development and deployment of the two components
must be coordinated.

#### Benefits

* Loose coupling
* Resilience
* No additional server components
* Low technical complexity
* Links often enough

####  Challenges

* Uniform look & feel

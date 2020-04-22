+++
title = "Core Web Stack"
weight = 300
+++

Unlike applications that live only on the Web, WWT has an installed based of
Windows clients some of which are more than a decade old. Some of those
Windows clients are installed at planetariums or museums that are using WWT in
public-facing exhibits but, generally speaking, very reluctant to experiment
with their technology infrastructure. Therefore, it is extremely important to
preserve the detailed behavior of the WWT web API endpoints used by the
historical clients.

What makes this requirement a bit tricky is that the historical WWT web API
endpoints are served off of the domain [worldwidetelescope.org], the same
domain that is used to host the webclient (at
[worldwidetelescope.org/webclient/]), the Communities service, and user-facing
web pages. In principle, we might like to use different web server frameworks
for each of these components of the WWT web presence. However, web requests to
the same domain name must all be handled by the same server(s) as seen by the
public internet.

[worldwidetelescope.org]: //worldwidetelescope.org/
[worldwidetelescope.org/webclient/]: //worldwidetelescope.org/webclient/


# Azure Application Gateway

Until April of 2020, traffic to [worldwidetelescope.org] was all handled by a
pair of virtual machines running a customized ASP.NET setup on
[IIS 8.5][iis85] that dealt with all forms of traffic to the domain.
Deployment was manual, seriously bottlenecking development due to the
aforementioned importance of keeping the core WWT data services highly
reliable.

[iis85]: https://en.wikipedia.org/wiki/Internet_Information_Services

As of April 2020, traffic to the flagship domain is handled by an
[Azure Application Gateway][app-gateway] frontend, a "layer 7" load balancer
and reverse proxy that can redirect traffic to different web server backends
based on the URL path requested. This is the core capability that allows us to
begin sharding out requests to the [worldwidetelescope.org] domain to
different servers depending on the service being requested.

[app-gateway]: https://azure.microsoft.com/en-us/services/application-gateway/

Furthermore, the app gateway acts as a
[TLS termination proxy][tls-termination], such that it can handle HTTPS
traffic while the communication to the data backends can use plain HTTP, which
is valuable since the data requests take the form of many small HTTP requests.
Before April 2020, the flagship [worldwidetelescope.org] domain did not
support HTTPS due to the configuration challenge and worries about the load
that TLS processing would add to the core data VMs.

[tls-termination]: https://en.wikipedia.org/wiki/TLS_termination_proxy


# Web Backends on the Flagship Domain

Since the transition to the App Gateway deployment model, requests to
[worldwidetelescope.org] may be served by one of five backends.

Note that the classic website treated paths case-insensitively; that is,
`/wwtweb/` and `/WWTWeb/` were routed identically. The App Gateway routing
infrastructure is also case-insensitive, but the Linux-based backends and the
Azure Storage static files website are case *sensitive*.

## CORS/TLS proxy backend

Requests to `/webserviceproxy.aspx` and `/wwtweb/webserviceproxy.aspx` are
used by the web client to proxy requests that would otherwise be forbidden by
the user’s web browser because of either [CORS] or [mixed content]
restrictions.

[CORS]: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[mixed content]: https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content

These requests are handled by a small server running [Squid] inside a [Docker]
container. The assets to create the container are in the repository
[wwt-squid-proxy]. Merges to the `master` branch of that repository trigger a
continuous deployment pipeline that updates the Docker image published as
[aasworldwidetelescope/proxy]. Finally, the WWT Azure infrastructure includes
an App Service named `wwtproxy` that runs that Docker image, including
hot-reloads when the image is updated. The App Gateway routes proxy requests
to the `wwtproxy` App Service.

[Squid]: http://www.squid-cache.org/
[Docker]: https://www.docker.com/
[wwt-squid-proxy]: https://github.com/WorldWideTelescope/wwt-squid-proxy
[aasworldwidetelescope/proxy]: https://hub.docker.com/repository/docker/aasworldwidetelescope/proxy

## nginx utility server

Requests to the `/docs` prefix and the path `/webclient` — but not
`/webclient/` or any paths below it — are handled by a utility [nginx] server,
since it can easily be configured to issue both server-side redirects and
simple static content.

[nginx]: https://nginx.org/

The infrastructure for this utility server mirrors that of the CORS/TLS proxy.
The container assets are tracked in the repository [wwt-nginx-core]. Merges to
`master` trigger a CD pipeline that publishes a built Docker image as
[aasworldwidetelescope/nginx-core]. In the WWT Azure infrastructure, an App
Service named `wwtnginxcore-prod` runs a server based on that Docker image
with automatic reload when the image is updated.

[wwt-nginx-core]: https://github.com/WorldWideTelescope/wwt-nginx-core
[aasworldwidetelescope/nginx-core]: https://hub.docker.com/repository/docker/aasworldwidetelescope/nginx-core

The nginx utility server also services requests and redirects for
miscellaneous WWT domains of mainly historical interest. See the README and
configuration file in [wwt-nginx-core] for details.

## IIS/ASP.NET "user" website server

When we refer to the core or flagship “user” website, we mean the
mostly-static user-facing content accessible through the portal at
[worldwidetelescope.org/home].

[worldwidetelescope.org/home]: //worldwidetelescope.org/home

At the moment, this website is an ASP.NET application whose source is tracked
in, and continuously deployed from, the repository [wwt-user-website]. Work is
underway to refresh this content and switch to a [static site generator]
backend. As such, the documentation of this framework will stop here.

[wwt-user-website]: https://github.com/WorldWideTelescope/wwt-user-website
[static site generator]: https://gohugo.io/about/benefits/

## Static content backend

Much of the content served by [worldwidetelescope.org] consists of static file
content. Since the infrastructure rebuild of April 2020, that includes the web
client AngularJS application (URL paths `/webclient/*`) as well as
miscellaneous data files in paths such as `/data/*` and `/images/*`.

Requests for this content are routed by the App Gateway to an Azure storage
account with the [static website hosting][azure-storage-staticweb] feature
enabled. In the WWT Azure services, this storage account is named
`wwtwebstatic`. Updates to this content are deployed by uploading new files to
the Azure storage account, which can be (and is) automated in continuous
deployment pipelines for version-controlled assets.

[azure-storage-staticweb]: https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website

## Legacy virtual machines

All web requests not handled by any of the other backends are routed to the
legacy Windows Server 2012R2 virtual machines. This includes important
elements of the WWT web services such as:

- Data and other services at URL paths `/wwtweb/*`
- The entire Communities web service
- Requests to the root path `/`, required by the way that App Gateway works

The sources to these web backends are nominally tracked in the repository
[wwt-website]. However, due to the manual deployment process required by these
VMs, **the actual assets running in production and the source repository have
diverged somewhat**. Work is underway to understand and minimize this
divergence, as well as to migrate the infrastructure to a system whose
deployment can be fully automated.

Further note that the production deployment includes configuration files
containing various keys and connection strings that must be kept secret.

[wwt-website]: https://github.com/WorldWideTelescope/wwt-website


# Web Client application

The WWT “web client”, hosted at [worldwidetelescope.org/webclient/], is an
[AngularJS] web application whose sources live in the repository
[wwt-web-client].

[AngularJS]: https://angularjs.org/
[wwt-web-client]: https://github.com/WorldWideTelescope/wwt-web-client/

Since the April 2020 infrastructure upgrade, a more careful distinction has
been made between the web client application and the [WebGL engine] library
powering it. The two are certainly closely connected, but they are now tracked
in separate repositories and released independently.

[WebGL engine]: https://github.com/WorldWideTelescope/wwt-webgl-engine/

Merges to the `master` branch of the [wwt-web-client] repository are
continuously deployed to a testing prefix,
[worldwidetelescope.org/testing_webclient/], by uploading built artifacts to
the appropriate location in the `wwtwebstatic` Azure Storage account.
Version-tagged releases are deployed to production through an analogous
process. For more information, see the README in the source repository.

[worldwidetelescope.org/testing_webclient/]: //worldwidetelescope.org/testing_webclient/


# TLS Certificates

As of the April 2020 infrastructure rebuild, the [worldwidetelescope.org]
domain supports HTTPS via TLS termination at the App Gateway.

TLS certificates are supplied through [Let’s Encrypt][lets-encrypt].
Certificate requests and deployment are automated using an instance of the
[keyvault-acmebot] application that runs on the WWT Azure services.

[lets-encrypt]: https://letsencrypt.org/
[keyvault-acmebot]: https://github.com/shibayan/keyvault-acmebot


# CDN Frontend

The core content served by the [worldwidetelescope.org] domain is also made
available through a [Content Distribution Network][cdn] (CDN) associated with
the domains `cdn.worldwidetelescope.org` and `content.worldwidetelescope.org`.
A request for the URL `cdn.worldwidetelescope.org/SOMEPATH` will yield
identical results as a request for the URL `worldwidetelescope.org/SOMEPATH`,
except that the response may hopefully arrive faster if the user is near one
of the CDN point-of-presence nodes and it has already cached the relevant
content.

[cdn]: https://en.wikipedia.org/wiki/Content_delivery_network

Because this CDN frontend is expected to serve content for arbitrary URL paths
at exposed by the [worldwidetelescope.org] origin, CDN requests must be routed
through the Azure App Gateway load-balancer, imposing a minor efficiency cost
(as well as a financial cost in App Gateway bandwidth charges). The vision is
to start transitioning CDN-friendly WWT content to route through subdomains of
the `wwtassets.org` domain, which can be set up to avoid the overhead of
routing through the App Gateway. A secondary advantage is that requests to a
separate second-level-domain will not include the HTTP cookies set by the core
[worldwidetelescope.org] domain, which can potentially increase the efficiency
of the HTTP traffic. (We are told that this is why most major websites use a
separate domain for their CDN traffic.)

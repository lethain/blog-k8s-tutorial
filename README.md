*(As a meta-point, my secret handling is fairly poor here. If/when I rework these,
I'll actually use Kubernetes to store all of my secrets versus including them in various
files, although I've dedacted all of them here.)*

Most of the configs used for my k8s setup.
Note, I recommend having the `Dockerfile`, `deployment.yaml` and `service.yaml` files
in their own repositories (e.g. one repository per container), and specifically the
`daedalus` Dockerfile (e.g. name of my blog's codebase, a more usable refactor of the previous version,
[icarus](https://github.com/lethain/icarus)) should be at the base of your Go repository.

Manifest is:

* `daedalus` - A simple Go blog that downloads state from S3, so it has no backend.
    * `Dockerfile` - simple Dockerfile to build the Go application
    * `deployment.yaml` - configures the application itself, critically including the healthchecks so it is routable!
    * `service.yaml` - gives a clusterip which is a simple load balancer that respects liveness health checks,
        and is used by the frontend to reverse proxy to the service cleanly during deployments.
* `frontend` - Ingress container, which your Google Loadbalancer will communicate with over TCP. Does SSL termination if you want it to.
    * `Dockerfile` - installs Nginx and copies nginx configs in (also SSL certs, if you set that up).
    * `deployment.yaml` - configures the frontend, here is proxying to Daedalus' healthcheck for frontend's liveness as well.
    * `service.yaml` a LoadBalancer type service, which instantiates a Google Loadbalancer to represent it externally (TCP).


The above is all you'd need for an HTTP setup, if you want HTTPS, I'm also running a simple `cert-refresh` application,
which is... less automated than I'd like, but is a singleton reverse-proxied to by the `frontend` tier to serve let's encrypt
challenge files.

* `Dockerfile` - builds container with Nginx and `certbot`. Also shows commands I ran to gen certs.
    Overall, it's a mess!
* `deployment.yaml` - this is a pretty simple one, but with a different liveness probe.
* `service.yaml` - just another ClusterIP for routing.

I'm also running [gke_ci](https://github.com/lethain/gke_ci) for continuous integration.
(Note that it doesn't need a `service.yaml` because it isn't routable.):

  * [Dockerfile](https://github.com/lethain/gke_ci) - is same as in `lethain/gke_ci`.
  *  `deployment.yaml` - configuration for a singleton instance running CI

I also ported over [systemsandpapers.com](https://systemsandpapers.com/), which is a bit more complex:
a Ruby service using MySQL and Memcache. Google doesn't have a convenient hosted Memcache solution
(it has one, but it's part of Google App Engine and doesn't appear usable externalyl), so this is running
a single node because I didn't want to figure out discovery properly when I set it up (and it's just for
session tokens so... whatever).

* `memcache` - Docker and Kubernetes configs for memcache service. Has typical docker and k8s configs.

Also note that Google Cloud SQL is [moderately frustrating to access from GKE](https://cloud.google.com/sql/docs/mysql/connect-container-engine),
requiring you to setup a proxy container, so Papers' `deployment.yaml` is more complex:

* `Dockerfile` - build a Ruby app. Pretty basic. Bind to `0.0.0.0`!
* `deployment.yaml` - pretty complex because it runs two containers in the deployment's pods.
    that is because it needs the the CloudSQL proxy, which is actually a well designed
    Kubernetes app rather than a bunch of config shoved everywhere like the configs I wrote.
    Highly recommend reading the [Cloud SQL from GKE tutorial](https://cloud.google.com/sql/docs/mysql/connect-container-engine),
    at least two or three times. :)
* `service.yaml` - standard cluster ip again.




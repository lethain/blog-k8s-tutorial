
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

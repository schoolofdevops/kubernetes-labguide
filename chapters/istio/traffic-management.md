# Traffic Management


===> [ Gateway ] ==> Virtual ==> Destination ==> Pods
        L4-L6        Service     Rules




## Virtual Services

  * decouples client requests from actual workloads at destination
  * allows you to define traffic routing rules
  * without virtual service, envoy would just use round robin method
  * Enables A/B testing, Canary
  * Advanced rules e.g. user 'x' routes to version v1
  * Service subsets are defined with destination rules
  * One virtual service can route to multiple kubernetes services e.g. rating/ productpage/ rules are defined in same virtual service.
  * works with gateway for front facing apps which need ingress/egress rules.



## Destination Rules

  * Subsets
  * Load Balancing Algorithms
    * Round Robin (default)
    * Random
    * Weighted
    * Least Requests

## Gateways

  * Similar to ingress controller
  * Sits at the edge
  * Gateway rules are applied to envoy proxy sidecars  at the edge, not application sidecars
  * Similar to k8s ingress, but handles only L4-L6 (e.g. ports, tls) and sends to Virtual Service for test.
  * Default is ingress gateway
  * Egress gateway is opt in. Allow you to route outgoing traffic through specific hosts

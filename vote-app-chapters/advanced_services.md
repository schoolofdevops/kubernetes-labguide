# Services

Services are a way to expose your backend pods to outside world. Services can also be used for internal networks.

## Service Discovery

Whenever we create a service, the kubernetes api server will create a route for that service within the cluster. So you will be able to access the service using the dns entry for that service. Usually the service dns will look like <SVC-NAME.NAMESPACE>

Let us create a service for frontend

`File: frontend-svc.yml`

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
```

```
kubectl apply -f frontend-svc.yml
kubectl exec -it front-end-5cbc986f44-dqws2 sh

nslookup front-end.instavote

Name:      front-end.instavote
Address 1: 10.233.6.236 front-end.instavote.svc.cluster.local
```

This is achieved by the cluster **dns add-on**. This is how backend of service A consumes service B.

## Labels and Selectors

When we create a service, it automatically adds appropriate pods to its backend. How does this happen? This is achieved by a mechanism called **labels and selectors**. We label our pods with keys and values. Our service template has selectors with these keys and values. So when a service is created, it will check for pods running with the same label. If it finds a pod with that label, the pod will be added to endpoint of the service.

```
kubectl describe svc front-end

Name:              front-end
Namespace:         instavote
Labels:            app=front-end
                   env=dev
Annotations:       kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"front-end","env":"dev"},"name":"front-end","namespace":"instavote"},"...
Selector:          app=front-end
Type:              ClusterIP
IP:                10.233.6.236
Port:              <unset>  8079/TCP
TargetPort:        8079/TCP
Endpoints:         10.233.71.13:8079
Session Affinity:  None
Events:            <none>
```

Since we have only one pod running with the label **app=front-end**, it is added to the service endpoint list.

## Picking up the right type of a service

We have four type of services to choose from. Each has a unique use case for which it can be used. Let us see about the types of services below.

### Types:

  * ClusterIp
    * Internal network.
    * Can not accessed from outside world.
    * Best suited for backend services which should not be exposed (ex: DB).
  * NodePort
    * Can be accessed from outside world.
    * NodePort is best suited for development environment(debugging).
    * Not suited for production use.
  * LoadBalancer
    * Normal cloud load balancer.
    * Exposes service to outside world.
  * ExternalName
    * Used for entities which are outside the cluster.
    * CNAME record for an external service.
    * No ports are defined.

We have one more service configuration, which is not exactly a service type. That is .,
  * ExternalIP

## Cluster IP and DNS

ClusterIP is internal to the cluster. ClusterIP is provided by default. If you do not define the type of the service, then the service is configured with a clusterIP. A typical service type of clusterIP looks like,

`File: frontend-svc.yml`

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
  type: ClusterIP
```

## NodePort

Exposes a port on node within 30000 to 32767 range. If a service is defined as nodePort, we can access the service on <ANY-NODE-IP:NODEPORT>. Let us change the frontend service type to NodePort and see what happens.

`File: frontend-svc.yml`

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
  type: NodePort
```

Lets get the node port created for the front-end service.

```
kubectl get svc front-end


```

Visit this node port on any of node's IP. In my case,


## ExternalIP

ExternalIP is not exactly a service type. It can be coupled with any other service type. For example a service with NodePort can be used with ExternalIP as well. Only cache here is, the external IP has to one of the node/master's IP. It is used to route the traffic to one or more cluster nodes.

`File: frontend-svc.yml`

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
  type: NodePort
  externalIPs:
  - 10.40.1.26
  - 10.40.1.11
```

Lets check how externalIPs works,

```
kubectl describe svc front-end

curl 10.40.1.26
curl 10.40.1.11
```

## LoadBalancer

Service type LoadBalancer is only supported on AWS,GCE,Azure and Openstack. If you have you cluster running on any of these clouds, give it a try. It will create a load balancer for us with a ephemeral IP. We can also specify a loadBalancerIP. Mostly not recommended to use. Using service type LoadBalancer will rise your cloud provider spendings. Think about launching 10 load balancers for 10 different services. Thats where **ingress** come into picture (explained in the later part).

```
apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  selector:
    app: front-end
  ports:
    - protocol: TCP
      port: 8079
  type: LoadBalancer
```

## Headless services with Endpoints/External Names

Sometimes you may to decouple the services from backend pods. Sometimes you may want to use some external services outside the cluster and want to integrate those external services with the cluster. For these kind of use cases we can use **Headless services**. Let us see an example.

Let us consider a scenario where you have your redis cluster hosted outside the cluster. Your backend in cluster need to use this redis cluster. In this case, you will create a service with ExternalName, which is nothing but a CNAME record of your redis cluster. This type of services will neither have any ports defined nor have any selectors. But, how do you make sure the backend pods uses this service? For that, you will create a custom endpoint, in which map your service to specific endpoints.

`Ex: External Name service`

```
kind: Service
apiVersion: v1
metadata:
  name: redis-aws
  namespace: instavote
spec:
  type: ExternalName
  externalName: redis.aws.schoolofdevops.com
```

`Ex: Endpoints`

```
kind: Endpoints
apiVersion: v1
metadata:
  name: redis-aws
subsets:
  - addresses:
      - ip: 10.40.30.123
      - ip: 10.40.30.324
      - ip: 10.40.30.478
    ports:
      - port: 9376
```

These IPs have to be manually managed by the cluster administrator.

## Ingress

Ingress controller is an Kubernetes object that manages external access to the backend services. This is the preferred way of exposing your services.

### Ingress Controller

To use an ingress resource, an ingress controller (ex. nginx, traefik) must be running. This controller has to be deployed manually.

To create an ingress controller in our cluster, run the following commands,

```
kubectl apply -f traefik-rbac.yaml
kubectl apply -f traefik-deployment.yaml
```

Check whether the ingress controller is running or not.

```
kubectl get pods -n kube-system

[...]
kubernetes-dashboard-75c5ff6844-r9sk8         1/1       Running   6          3d
nginx-proxy-node3                             1/1       Running   8          27d
nginx-proxy-node4                             1/1       Running   7          27d
traefik-ingress-controller-568dd77566-vk72b   1/1       Running   0          5m
```

### Purpose of an Ingress

Ingress can be used for,
  * Load balancing,
  * SSL Termination,
  * Name based virtual hosting.

### Types of Ingress

Let us see about different implementation of ingress.

```
Keep or delete the following(Need to come up with use cases).
### Single Service Ingress

### Simple fanout Ingress

### Name based virtual hosting

### TLS

### Load Balancer
```

## Services Vs Ingress

1. Service type LoadBalancer
2. Service type Nodeport

## Creating service resources for applications and backing services


---

Need a demo set up for types: load balancer and external name
Need input on this: Creating service resources for applications and backing services, Service API specs, Traffic Policy
Can add some points about scaling of ingress controller (running it as a daemonset on all the nodes)

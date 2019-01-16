# Exposing Application with  a Service

Types of Services:   

  * ClusterIP
  * NodePort
  * LoadBalancer
  * ExternalName


![kubernetes service](https://github.com/schoolofdevops/ultimate-kubernetes-bootcamp/blob/master/images/k8s_service.jpg?raw=true)


```
kubectl get pods
kubectl get svc
```

Sample Output:
```
NAME                         READY     STATUS    RESTARTS   AGE
frontend-64597c55f8-dk6g7   1/1       Running   0          4m
frontend-64597c55f8-mkp2h   1/1       Running   0          4m
frontend-64597c55f8-r2c54   1/1       Running   0          4m
```

## Publishing a service with NodePort

Filename: frontend-svc.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: mogambo
spec:
  selector:
    app: frontend
    env: dev
  ports:
    - port: 8079
      protocol: TCP
      targetPort: 8079
  type: NodePort

```

Save the file.

Now to create a service:

```
kubectl apply -f frontend-svc.yaml
kubectl get svc
```

Now to check which port the pod is connected
```
kubectl describe service frontend
```
Check for the Nodeport here

Sample Output
```
Name:                     frontend
Namespace:                mogambo
Labels:                   role=svc
                          tier=front
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"role":"svc","tier":"front"},"name":"frontend","namespace":"mogambo"},"spec...
Selector:                 app=frontend
Type:                     NodePort
IP:                       10.11.248.77
Port:                     <unset>  8079/TCP
TargetPort:               8079/TCP
NodePort:                 <unset>  30197/TCP
Endpoints:                10.8.1.13:8079,10.8.1.14:8079,10.8.1.15:8079 + 3 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Go to browser and check hostip:NodePort

Here the node port is 30197.

Sample output will be:

![frontend](images/frontend.png)

## Exposing the app with ExternalIP

`file: /k8s-code/projects/mogambo/dev/frontend-svc.yml`

```
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: NodePort
  externalIPs:
    - 192.168.12.11
    - 192.168.12.12
```

Where 192.168.12.11 and 192.168.12.12 are the actual IPs of kubernetes hosts. Replace it as you see fit.

apply
```
kubectl apply -f frontend-svc.yml
kubectl  get svc
kubectl describe svc frontend
```

## Exposing the app with LoadBalancer (Only on Cloud)

Service type LoadBalancer is only supported on AWS,GCE,Azure and Openstack. If you have you cluster running on any of these clouds, give it a try. It will create a load balancer for us with a ephemeral IP. We can also specify a loadBalancerIP. Mostly not recommended to use. Using service type LoadBalancer will rise your cloud provider spendings. Think about launching 10 load balancers for 10 different services. Thats where **ingress** come into picture (explained in the later part).

```
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: mogambo
  labels:
    app: frontend
    env: dev
spec:
  selector:
    app: frontend
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
  namespace: mogambo
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


#### Exercises

  * Create service definitions for catalogue and catalogue-db

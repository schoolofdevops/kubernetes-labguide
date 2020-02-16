# Setting up Istio Service Mesh on Kubernetes

If you already have prometheus and grafana configured, clean it up as istio will come with its own monitoring setup.

Assuming you have installed helm version 3,

```
helm list

```
[sample output]
```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
grafana         instavote       1               2020-02-15 06:55:38.895788799 +0000 UTC deployed        grafana-4.6.3           6.6.0
prometheus      instavote       1               2020-02-15 06:54:00.177788896 +0000 UTC deployed        prometheus-10.4.0       2.15.2
```

Uninstall prometheus and grafana as,

```
helm uninstall prometheus
helm uninstall grafana
```


## Install Istio Control Plane

Download and install version 1.4.4. of istio using istioctl utility as,

```
cd ~
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.4.4/
export PATH=$PWD/bin:$PATH
echo "export PATH=$PWD/bin:$PATH" >> ~/.bashrc
```

Install istio with demo configuration profile. This is a quick start profile. You could customize the profile and apply it later.

```
istioctl manifest apply --set profile=demo
```

validate

```
kubectl get pods -n istio-system
```

[Sample Output]

```
NAME                                      READY   STATUS    RESTARTS   AGE
grafana-6b65874977-8mlrq                  1/1     Running   0          6h26m
istio-citadel-58c44d6964-9vsl2            1/1     Running   0          6h26m
istio-egressgateway-74b6995559-w6hx8      1/1     Running   0          6h26m
istio-galley-5bf54d9674-plq6j             1/1     Running   0          6h26m
istio-ingressgateway-7f4b56656f-cs6gw     1/1     Running   0          6h26m
istio-pilot-568855fb7-lbt2v               1/1     Running   0          6h26m
istio-policy-fdb9d6845-prmb5              1/1     Running   4          6h26m
istio-sidecar-injector-56b7656fd8-ws7hr   1/1     Running   0          6h26m
istio-telemetry-676db4d695-8kmgh          1/1     Running   3          6h26m
istio-tracing-c66d67cd9-db2bp             1/1     Running   0          6h26m
kiali-8559969566-sjrqm                    1/1     Running   0          6h26m
prometheus-66c5887c86-vpbnk               1/1     Running   0          6h26m
```

As you notice, Istio is been installed with all components including prometheus and grafana in its own **istio-system** namespace.

## Nano Project: Expose istio services with NodePort

```
kubectl get svc -n istio-system
```

[sample output]

```
grafana                  ClusterIP      10.108.218.221   <none>        3000/TCP                                                                                                                     6h27m
istio-citadel            ClusterIP      10.99.97.53      <none>        8060/TCP,15014/TCP                                                                                                           6h27m
istio-egressgateway      ClusterIP      10.101.104.101   <none>        80/TCP,443/TCP,15443/TCP                                                                                                     6h27m
istio-galley             ClusterIP      10.100.6.131     <none>        443/TCP,15014/TCP,9901/TCP,15019/TCP                                                                                         6h27m
istio-ingressgateway     LoadBalancer   10.100.139.2     <pending>     15020:31152/TCP,80:32201/TCP,443:32595/TCP,15029:30669/TCP,15030:32260/TCP,15031:31472/TCP,15032:30961/TCP,15443:31147/TCP   6h27m
istio-pilot              ClusterIP      10.102.149.206   <none>        15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                       6h27m
istio-policy             ClusterIP      10.105.1.93      <none>        9091/TCP,15004/TCP,15014/TCP                                                                                                 6h27m
istio-sidecar-injector   ClusterIP      10.102.207.177   <none>        443/TCP                                                                                                                      6h27m
istio-telemetry          ClusterIP      10.110.151.201   <none>        9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                       6h27m
jaeger-agent             ClusterIP      None             <none>        5775/UDP,6831/UDP,6832/UDP                                                                                                   6h27m
jaeger-collector         ClusterIP      10.107.193.83    <none>        14267/TCP,14268/TCP,14250/TCP                                                                                                6h27m
jaeger-query             ClusterIP      10.108.133.235   <none>        16686/TCP                                                                                                                    6h27m
kiali                    ClusterIP      10.108.29.253    <none>        20001/TCP                                                                                                                    6h27m
prometheus               ClusterIP      10.96.44.190     <none>        9090/TCP                                                                                                                     6h27m
tracing                  ClusterIP      10.104.140.214   <none>        80/TCP                                                                                                                       6h27m
zipkin                   ClusterIP      10.105.49.167    <none>        9411/TCP                                                                                                                     6h27m
```

You would observe that istio is been deployed with either CLusterIP or LoadBalancer as type of services. You may need to access some of these services from outside, and the quickest way to do so would be using NodePort as the service type.

Your task is to open the following services to outside traffic using NodePort as the service type

  * Grafana
  * Istio Ingress Gateway
  * zipkin
  * tracing
  * kiali



## Deploy bookinfo sample app

To deploy a sample microservices app [bookinfo](https://istio.io/docs/examples/bookinfo/), first create a bookinfo namespace and switch to it.


```
kubectl create namespace bookinfo
kubectl label namespace bookinfo istio-injection=enabled
kubectl get ns --show-labels
kubectl config set-context --current --namespace=bookinfo
```

Now deploy bookinfo sample app as,

`ensure you are in the istio-xxx directory before running these commands`
```
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

```
validate that all services have been deployed with the following command

```
kubectl get all

```

[sample output]
```
NAME                                  READY   STATUS    RESTARTS   AGE
pod/details-v1-78d78fbddf-csfd2       2/2     Running   0          71s
pod/productpage-v1-596598f447-t7bn4   2/2     Running   0          70s
pod/ratings-v1-6c9dbf6b45-b4h6h       2/2     Running   0          70s
pod/reviews-v1-7bb8ffd9b6-4tq7s       2/2     Running   0          70s
pod/reviews-v2-d7d75fff8-zkq7l        2/2     Running   0          70s
pod/reviews-v3-68964bc4c8-mq8xq       2/2     Running   0          70s

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/details       ClusterIP   10.103.247.170   <none>        9080/TCP   71s
service/productpage   ClusterIP   10.103.250.98    <none>        9080/TCP   70s
service/ratings       ClusterIP   10.105.113.217   <none>        9080/TCP   70s
service/reviews       ClusterIP   10.97.164.129    <none>        9080/TCP   70s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/details-v1       1/1     1            1           71s
deployment.apps/productpage-v1   1/1     1            1           70s
deployment.apps/ratings-v1       1/1     1            1           70s
deployment.apps/reviews-v1       1/1     1            1           70s
deployment.apps/reviews-v2       1/1     1            1           70s
deployment.apps/reviews-v3       1/1     1            1           70s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/details-v1-78d78fbddf       1         1         1       71s
replicaset.apps/productpage-v1-596598f447   1         1         1       70s
replicaset.apps/ratings-v1-6c9dbf6b45       1         1         1       70s
replicaset.apps/reviews-v1-7bb8ffd9b6       1         1         1       70s
replicaset.apps/reviews-v2-d7d75fff8        1         1         1       70s
replicaset.apps/reviews-v3-68964bc4c8       1         1         1       70s
```

You should see the following microservices deployed,

  * productpage-v1
  * details-v1
  * ratings-v1
  * reviews-v1
  * reviews-v2
  * reviews-v3

You would also see each of the pod is running atleast 2 pods, that validates that envoy proxy is been injected along with each of your application.  If you see these services running, you have a working istio cluster with a sample app running on top of your kubernetes environment.

## Exposing bookinfo app from outside

Istio has its own way of handling traffic and exposing it outside. Istio creates two difference resources which are equivalent of what kubernetes service offers.  

  * Gateway
  * VirtualService

To expose the bookinfo application externally, create and examine gateway and VirtualService as,

```
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

```

validate

```
kubectl get virtualservice,gateway

or

istioctl get all
```

Observe the output of  following

```
kubectl describe gateway bookinfo
kubectl describe virtualservice bookinfo
```

To access this application, you need to know the following two things,

  * IP Address/Hostname of host which is running Ingress Gateway
  * NodePort mapping to port 80 for Ingress Gateway

Use the following commands and note down the IP Address/Hostname and NodePort for ingress
```
kubectl get pod -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}'

kubectl get svc -l istio=ingressgateway -n istio-system
```

[sample output]

```
kubectl get pod -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}'
178.128.53.126


kubectl get svc -l istio=ingressgateway -n istio-system

NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                                                                                                                      AGE
istio-ingressgateway   LoadBalancer   10.100.139.2   <pending>     15020:31152/TCP,80:32201/TCP,443:32595/TCP,15029:30669/TCP,15030:32260/TCP,15031:31472/TCP,15032:30961/TCP,15443:31147/TCP   8h
```

In the example above, IP address of my ingress host is 178.128.53.126 and NodePort mapping to 80 is 32201.

Now to access bookinfo frontend app, I would construct a url as follows

http://178.128.53.126:32201/productpage

Alternately you could use the hostname/fqdn of the host if you know it.

If you load this url in the browser, you should see the product page for bookinfo app as,

![bookinfo product page](../../images/bookinfo.png)


## Apply Destination Rules

```
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml

kubectl get dr

kubectl describe dr rating

```

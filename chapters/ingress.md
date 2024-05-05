# Lab K201 - Application Routing with Ingress Controllers


## Pre Requisites

  * Ingress controller such as Nginx, Trafeik needs to be deployed before creating ingress resources.
  * On GCE, ingress controller runs on the master. On all other installations, it needs to be deployed, either as a deployment, or a daemonset. In addition, a service needs to be created for ingress.
  * Daemonset will run ingress on each node. Deployment will just create a highly available setup, which can then be exposed on specific nodes using ExternalIPs configuration in the service.

## Launch Ingress Controller

An ingress controller needs to be created in order to serve the ingress requests.  As part of this lab you are going to use [Traefik](https://traefik.io/) as the ingress controller. Its a fast and lightweight ingress controller and also comes with great documentation and support.  

```bash


+----+----+--+            
| ingress    |            
| controller |            
+----+-------+            


```

You are going to setup ingress controller using helm. Assuming helm is already installed, begin adding the repository to install traefik as a ingress controller from:


```
helm repo add traefik https://helm.traefik.io/traefik
helm repo update

```

Pull Traffic Chart

```
helm pull  --untar traefik/traefik
cd traefik
```


Create a  new file  `my.values.yaml` with the following contents

```
ports:
  traefik:
    port: 9000
    expose: true
    exposedPort: 9000
    nodePort: 30300
    protocol: TCP
  web:
    port: 8000
    expose: true
    exposedPort: 80
    nodePort: 30400
    protocol: TCP
service:
  enabled: true
  single: true
  type: NodePort
```
Source :  [traefik deployment customization spec · GitHub](https://gist.github.com/initcron/f8099b32b456cb04e2e64263eaf9abfb)


Apply with the custom values file created as above,

```
kubectl create ns traefik
helm install traefik --namespace=traefik --values=my.values.yaml .

```


Validate

```
helm list -A
kubectl get all -n traefik
```

Use the following URIs to access Traefik and Traefik Dashboard ,

* Traefik Web :  http://xx.xx.xx.xx:30400 (you may see 404 page not found error, which is okay and expected here).
* Traefik Dashboard :  http://xx.xx.xx.xx:30300/dashboard/  (training slash “/“ is a must)  (you are expected to a nice looking see Traefik dashboard here).

`replace xx.xx.xx.xx with the external IP used earlier`


![Traefik](../images/traefik01.png)

## Set up Named Based Routing for Vote App

We will direct all our request to the ingress controller now, but with differnt hostname e.g. **vote.example.com** or **results.example.com**. And it should direct to the correct service based on the host name.

In order to achieve this you, as a user would create a **ingress** object with a set of rules,


```bash


+----+----+--+            
| ingress    |            
| controller |            
+----+-------+            
     |              +-----+----+
     +---watch----> | ingress  | <------- user
                    +----------+

```


`file: vote-ing.yaml`

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vote
  namespace: instavote
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.ingress.kubernetes.io/rule-type: PathPrefixStrip
spec:
  rules:
  - host: vote.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vote
            port:
              number: 80  
```

And apply

```
kubectl get ing
kubectl apply -f vote-ing.yaml --dry-run
kubectl apply -f vote-ing.yaml
```

Since the ingress controller  is constantly monitoring for the ingress objects, the moment it detects, it connects with traefik and creates a rule as follows.


```bash

                    +----------+
     +--create----> | traefik  |
     |              |  rules   |
     |              +----------+
+----+----+--+            ^
| ingress    |            :
| controller |            :
+----+-------+            :
     |              +-----+----+
     +---watch----> | ingress  | <------- user
                    +----------+

```

where,

  * A user creates a ingress object with the rules. This could be a named based or a path based routing.
  * An ingress controller, in this example traefik constantly monitors for ingress objects. The moment it detects one, it creates a rule and adds it to the traefik load balancer. This rule maps to the ingress specs.


You could now see the rule added to ingress controller,

![Traefik with Ingress Rules](../images/traefik02.png)

Where,

  * **vote.example.com** is  added as frontend. This frontends point to  service **vote**.

  * respective backend also appear on the right hand side of the screen, mapping to each of the service.

## Add Local DNS

You have created the ingress rules based on hostnames e.g.  **vote.example.com** and **results.example.com**. In order for you to be able to access those, there has to be a dns entry pointing to your nodes, which are running traefik.

```bash

  vote.example.com     -------+                        +----- vote:81
                              |     +-------------+    |
                              |     |   ingress   |    |
                              +===> |   node:80   | ===+
                              |     +-------------+    |
                              |                        |
  results.example.com  -------+                        +----- results:82

```

To achieve this you need to either,

  * Create a DNS entry, provided you own the domain and have access to the dns management console.
  * Create a local **hosts** file entry. On unix systems its in `/etc/hosts` file. On windows its at `C:\Windows\System32\drivers\etc\hosts`. You need admin access to edit this file.


For example, on a linux or osx, you could edit it as,

```
sudo vim /etc/hosts
```

And add an entry such as ,

```
xxx.xxx.xxx.xxx vote.example.com results.example.com  kube-ops-view.example.org
```

where,

  * xxx.xxx.xxx.xxx is the actual IP address of one of the nodes running traefik.

And then access the app urls using http://vote.example.com or http://results.example.com

![Name Based Routing](../images/domain-name.png)


## Add another  Web Application with Ingress

```
kubectl create deployment nginx --image=nginx
kubectl create service clusterip nginx --tcp=80
```

Add ingress rule

`file: web-ing.yaml`

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
  namespace: instavote
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.ingress.kubernetes.io/rule-type: PathPrefixStrip
spec:
  rules:
  - host: web.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```
Source: [Ingress for Web App · GitHub](https://gist.github.com/initcron/469b8d975c1c41a8423eb19ee2fd9c0f)

And update hosts file with new route as  ,

```
xxx.xxx.xxx.xxx vote.example.com web.example.com results.example.com kube-ops-view.example.org
```

Visit  http://web.example.com:30400 to see your request being routed to nginx web page.  


## Mini Project : Configure  Ingress  for Result App

Now that you have added the ingress rule for the vote app, follow the same method and add one for the result app as well which is running in the same namespace.

The final validation would be, when you access or http://result.example.com you should see it pointing to the result app.





### Reading Resources 

  * [Trafeik's Guide to Kubernetes Ingress Controller](https://docs.traefik.io/user-guide/kubernetes/)
  * [Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)
  * [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

**References**

  * [Online htpasswd generator](http://www.htaccesstools.com/htpasswd-generator/)

**Keywords**

  * trafeik on kubernetes
  * kubernetes ingress
  * kubernetes annotations
  * daemonsets

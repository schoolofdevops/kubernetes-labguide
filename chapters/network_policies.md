# Enforcing Network Policies

If you are using KIND based environment, you would have to recreate the cluster to support network policies as the `kind-net` does not support this feature. Use the following instructions to recreate the cluster with `calico` and then proceed to create the network policies.


## Recreate KIND Cluster with Calico

For Network Policies to work, you need to have a CNI Plugin which supports it. `kind-net` which is installed as  a defulay CNI with KIND, does not support it.  You could recreate the cluster with `calico` as a CNI plugin by following the instructions here.


Delete existing cluster created with KIND as,

```
kind get clusters
kind delete cluster --name k8slab
```

assuming `k8slab` is the name of the cluster. Change it with the actual name.

File : `k8s-code/helper/kind/kind-three-node-cluster.yaml`

disable the default network as

```
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.31.0/16
```

launch the cluster again as

```
cd k8s-code/helper/kind
kind create cluster  --config kind-three-node-cluster.yaml
```

validate
```
kubectl get nodes
```

the nodes would be in NotReady stat at this time because of no CNI (Network) Plugin.

Set up calico as

```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/tigera-operator.yaml

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/custom-resources.yaml
```

wait for calico pods to be ready  

```
watch kubectl get pods -l k8s-app=calico-node -A

```

once the pods for calico are setup, exit from this watch command (use ^c) and  validate the node status again as:

```
kubectl get nodes
kubectl get pods -A

```

At this time, nodes should be up and running.  That

You may proceed to create any deployments, services needed at this time.



## Recreating the Application Deployment

```
kubectl create namespace instavote
kubectl config set-context --current --namespace=instavote
```

validate you are switched to `instavote` namespace as

```
kubectl config get-contexts
```

assuming you have access to all the code to create `instavote` stack, apply it using command similar to follows

```
kubectl apply -f vote-svc.yaml -f vote-deploy.yaml  \
  -f redis-svc.yaml -f redis-deploy.yaml \
  -f db-svc.yaml -f db-deploy.yaml  \
  -f worker-deploy.yaml -f results-svc.yaml \
  -f results-deploy.yaml
```

validate you have 5 deployments and 4 services as

```
kubectl get deploy,svc
```

[sample output]
```
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db       1/1     1            1           45m
deployment.apps/redis    1/1     1            1           45m
deployment.apps/result   1/1     1            1           45m
deployment.apps/vote     1/1     1            1           45m
deployment.apps/worker   1/1     1            1           45m

NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/db       ClusterIP   10.96.180.62    <none>        5432/TCP       45m
service/redis    ClusterIP   10.96.197.185   <none>        6379/TCP       45m
service/result   NodePort    10.96.61.34     <none>        80:30100/TCP   45m
service/vote     NodePort    10.96.242.67    <none>        80:30000/TCP   45m
```

## Locking down access with a NetworkPolicy

Now, define a restrictive network policy which would,

  * Block all incoming connections
  * Block all outgoing connections

```bash

  +-----------------------------------------------------------+
  |                                                           |
  |    +----------+          +-----------+                    |
x |    | results  |          | db        |                    |
  |    |          |          |           |                    |
  |    +----------+          +-----------+                    |
  |                                                           |
  |                                                           |
  |                                        +----+----+--+     |           
  |                                        |   worker   |     |            
  |                                        |            |     |           
  |                                        +----+-------+     |           
  |                                                           |
  |                                                           |
  |    +----------+          +-----------+                    |
  |    | vote     |          | redis     |                    |
x |    |          |          |           |                    |
  |    +----------+          +-----------+                    |
  |                                                           |
  +-----------------------------------------------------------+

```




file: `instavote-netpol.yaml`

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default
  namespace: instavote
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

apply

```
kubectl get netpol

kubectl apply -f instavote-netpol.yaml

kubectl get netpol

kubectl describe netpol/default

```

Try accessing the vote and results ui. Can you access it ?

`Troubleshooting Tip`: If you do not see the above policy being in effect (i.e. if you can still access the applications), go back and check if you have applied the label to the namespace as mentioned in the beginning of this section.


## Enabling external traffic to outward facing applications

```bash

  +-----------------------------------------------------------+
  |                                                           |
  |    +----------+          +-----------+                    |
=====> | results  |          | db        |                    |
  |    |          |          |           |                    |
  |    +----------+          +-----------+                    |
  |                                                           |
  |                                                           |
  |                                        +----+----+--+     |           
  |                                        |   worker   |     |            
  |                                        |            |     |           
  |                                        +----+-------+     |           
  |                                                           |
  |                                                           |
  |    +----------+          +-----------+                    |
  |    | vote     |          | redis     |                    |
=====> |          |          |           |                    |
  |    +----------+          +-----------+                    |
  |                                                           |
  +-----------------------------------------------------------+

```

To the same file, add a new network policy object.


`file: instavote-netpol.yaml`

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default
  namespace: instavote
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: public-ingress
  namespace: instavote
spec:
  podSelector:
    matchExpressions:
      - {key: role, operator: In, values: [vote, result]}
  policyTypes:
  - Ingress
  ingress:
    - {}
```

where,

*instavote-ingress* is a new network policy  which,

  * defines policy for pods with **vote** and **results** role
  * and allows them incoming access from anywhere


apply
```
kubectl apply -f instavote-netpol.yaml
```
**Exercise**

  * Try accessing the ui now and check if you are able to.
  * Try to vote, see if that works? Why ?


## Enabling communication between pods in the same namespace

When you tried to vote, you might have observed that it does not work. Thats because the default network policy we created earlier blocks all outgoing traffic. Which is good for securing the  environment, however you still need to provide inter connection between services from the same project.  Specifically **vote**, **worker** and **results** apps need outgoing connection to **redis** and **db**. Lets allow that with a egress policy.


```bash

  +-----------------------------------------------------------+
  |                                                           |
  |    +------------+        +-----------+                    |
=====> | results    | ------>| db        |                    |
  |    |            |        |           | <-------+          |
  |    +------------+        +-----------+         |          |
  |                                                |          |
  |                                                |          |
  |                                        +----+----+---+    |           
  |                                        |   worker    |    |            
  |                                        |             |    |           
  |                                        +----+--------+    |           
  |                                                |          |
  |                                                |          |
  |    +----------+          +-----------+         |          |
  |    | vote     |          | redis     | <-------+          |
=====> |          |  ------> |           |                    |
  |    +----------+          +-----------+                    |
  |                                                           |
  +-----------------------------------------------------------+

```


Edit the same policy file  and add the following snippet,

`file: instavote-netpol.yaml`

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default
  namespace: instavote
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector: {}  # Allows all pods within the same namespace
  egress:
  - to:
    - podSelector: {}  # Allows all pods within the same namespace
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: public-ingress
  namespace: instavote
spec:
  podSelector:
    matchExpressions:
      - {key: role, operator: In, values: [vote, result]}
  policyTypes:
  - Ingress
  ingress:
    - {}
```

where,

*instavote-egress* is a new network policy  which,

  * defines policy for pods with **vote**, **worker** and **results** role
  * and allows them outgoing  access to  any pods in the same namespace, and that includes **redis** and **db**

### Troubleshooting Exercise

Applying the above policy has no effect on the communication between **vote** and **redis**  applications. You could validate this by loading the vote app and submit a vote. It should not work. There is a problem in the network policy file above. Analyse the policies, compare them against the kubernetes api reference document, understand how its being applied and see if you could fix this problem.  Your task is to ensure that **vote** and **redis** apps are communicating with one another.  

#### Solution  

The reason why communication between `vote` and `redis` is broken is because of the name resoulution (DNS Based Service Discovery) is broken. This is because the network policies that you have set up do not allow the services in `instavote` namespace to communicate to even the DNS server in the cluster running in `kube-system` namespace.

You could allow this by adding one more policiy. You could add it to the same file `instavote-netpol.yaml`  

e.g.

```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: dns-access
  namespace: instavote
spec:
  podSelector: {}  # Applies to all pods in the namespace
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
      ports:
        - protocol: UDP
          port: 53
```

now apply and validate

```
kubectl apply -f instavote-netpol.yaml
```

Now you should see `vote` app connecting with `redis`.

## Nano Project

The above network policies are a good start. However you could even further restrict access by creating a granular network policy for each application.  

Create network policies with following specs,

**vote**

  * allow incoming connections from anywhere, only on port 80
  * allow outgoing connections to **redis**
  * block everything else, incoming and outgoing  

**redis**

  * allow incoming connections from **vote** and **worker**, only on port 6379
  * block everything else, incoming and outgoing  

**worker**

  * allow outgoing connections to **redis** and **db**
  * block everything else, incoming and outgoing  

**db**

  * allow incoming connections from **worker** and **results**, only on port 5342
  * block everything else, incoming and outgoing  


**result**

  * allow incoming connections from anywhere, only on port 80
  * allow outgoing connections to **db**
  * block everything else, incoming and outgoing  

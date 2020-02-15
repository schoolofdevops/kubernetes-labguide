# Setting up a firewall with Network Policies


While setting up the network policy, you may need to refer to the namespace created earlier. In order to being abel to referred to, namespace should have a label. Lets  update the namespace with a label.

`file: instavote-ns.yaml`

```
kind: Namespace
apiVersion: v1
metadata:
  name: instavote
  labels:
    project: instavote

```

apply

```
kubectl get namespace --show-labels
kubectl apply -f instavote-ns.yaml
kubectl get namespace --show-labels

```

## Locking down access to a namespace

Now, define a restrictive network policy which would,

  * Block all incoming connections from any source except for pods from the same namespace  
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
    - namespaceSelector:
        matchLabels:
          project: instavote
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          project: instavote
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

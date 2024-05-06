# Lab K203 - Advanced Pod Scheduling

In the Kubernetes bootcamp training, we have seen how to create a pod and and some basic pod configurations to go with it. But this chapter explains some advanced topics related to pod scheduling.


From the [api document for version 1.11](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/#pod-v1-core) following are the pod specs which are relevant from scheduling perspective.

  * nodeSelector
  * nodeName
  * affinity
  * schedulerName
  * tolerations


## nodeName

You could bind a pod to a specific node with a name using nodeName spec. Lets take an example where you want to run the deployment for `result` service on a specific node. Lets look at how you would do it,

Begin by listing the nodes

```
kubectl get nodes
```

[sample output]

```
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   35h   v1.29.2
kind-worker          Ready    <none>          35h   v1.29.2
kind-worker2         Ready    <none>          35h   v1.29.2
```

Now, bind your pod to one node e.g. `kind-worker` by modyfying the deployment spec as:

File : `result-deploy.yaml`

```
apiVersion: apps/v1
kind: Deployment
....
.....
spec:
  containers:
  - image: schoolofdevops/vote-result
    name: vote-result
  nodeName: kind-worker
```

apply and validate

```
kubectl apply -f result-deploy.yaml
kubectl get pods -o wide
```

## nodeSelector

Using nodeSelector instead of directly specifying nodeName in Kubernetes offers greater flexibility and resilience in scheduling pods. While nodeName forces a pod to schedule on a specific node, effectively bypassing Kubernetes' scheduler, nodeSelector allows for more dynamic placement by specifying a set of criteria that nodes must meet for the pod to be scheduled there. This approach utilizes Kubernetes' intelligent scheduling capabilities, enabling the system to consider multiple suitable nodes that meet the specified labels. This not only improves fault tolerance by avoiding dependencies on a single node but also facilitates more efficient resource utilization across the cluster. Additionally, nodeSelector supports scenarios where the environment might change, such as when nodes are added or removed, or their labels are updated, ensuring that the pods can still be scheduled according to the current state of the cluster.


To use nodeSelector, begin by labeling your nodes as:

```
kubectl get nodes --show-labels

kubectl label nodes <node-name> zone=aaa

kubectl get nodes --show-labels

```

e.g.
```
kubectl label nodes kind-worker zone=aaa
kubectl label nodes kind-worker2 zone=bbb
kubectl get nodes --show-labels
```

where, replace **kind-worker** and **kind-worker2** can be the  the actual nodes in your cluster.


Now update one of the deployments and add the nodeSelector spec to the pod  e.g.

File : `result-deploy.yaml`

```
spec:
  containers:
  - image: schoolofdevops/vote-result
    name: vote-result
  nodeSelector:
    zone: bbb
```

`Note: ensure you have removed nodeName if present.`

apply and validate

```
kubectl apply -f result-deploy.yaml
kubectl get pods -o wide
```

You shall see the pod being recreated now on the node matching the label selected using nodeSelector.

## Affinity and Anti-Affinity

We have discussed about scheduling a pod on a particular node using **NodeSelector**. Using affinity and anti-affinity in Kubernetes offers a more sophisticated and granular level of control compared to nodeSelector, enabling not just simple label matching but also complex rules that govern pod placement. Affinity rules allow you to specify preferences that attract pods to certain nodes, either based on the node's properties or other pods that are already running on those nodes. Conversely, anti-affinity rules are used to ensure pods are spread across different nodes or node groups, enhancing high availability and reducing the risk of simultaneous failures. This is particularly useful in large-scale deployments where maintaining workload balance and resilience is crucial. For example, you can ensure that multiple instances of a service run in different racks or availability zones, minimizing potential impact from physical infrastructure failures. These features allow Kubernetes to more effectively manage the distribution and redundancy of workloads, which is vital for maintaining robust, scalable applications.

More over,  using nodeSelector wolud mean defining a strict condition which must be met. If the condition is not met, the pod cannot be scheduled. Node/Pod affinity and anti-affinity solves this issue by introducing soft and hard conditions which are flexible based on when they are applied. This is controlled using the following properties

  * required
  * preferred

  * DuringScheduling
  * DuringExecution


and using theese operators

  * In
  * NotIn
  * Exists
  * DoesNotExist
  * Gt
  * Lt

Lets take up some examples and understand this.


### nodeAffinity


Examine  the current pod distribution  


```
kubectl get pods -o wide --selector="role=vote"

```

and node labels
```
kubectl get nodes --show-labels

```

Lets create node affinity criteria as

  * Pods for vote app **must** not run on the master nodes
  * Pods for vote app **preferably** run on a node in zone **bbb**

First is a **hard** affinity versus second being **soft** affinity.

`file: vote-deploy-nodeaffinity.yaml`

```
....
  template:
....
    spec:
      containers:
        - name: app
          image: schoolofdevops/vote:v1
          ports:
            - containerPort: 80
              protocol: TCP

              affinity:
                nodeAffinity:
                  requiredDuringSchedulingIgnoredDuringExecution:
                    nodeSelectorTerms:
                    - matchExpressions:
                      - key: node-role.kubernetes.io/control-plane
                        operator: DoesNotExist
                  preferredDuringSchedulingIgnoredDuringExecution:
                    - weight: 1
                      preference:
                        matchExpressions:
                        - key: zone
                          operator: In
                          values:
                            - bbb
```


clearn up previous deployment and apply this code as

```
kubectl delete deploy vote

kubectl apply -f vote-deploy-nodeaffinity.yaml

kubectl get pods -o wide
```

### podAffinity and podAntiAffinity


Lets define pod affinity criteria as,

  * Pods for **vote** and **redis** should be co located as much as possible (preferred)
  * No two pods with **redis** app should be running on the same node (required)


```
kubectl get pods -o wide --selector="role in (vote,redis)"

```


`file: vote-deploy-podaffinity.yaml`

```
...
    template:
...
    spec:
      containers:
        - name: app
          image: schoolofdevops/vote:v1
          ports:
            - containerPort: 80
              protocol: TCP

      affinity:
...

        podAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                  - key: role
                    operator: In
                    values:
                    - redis
                topologyKey: kubernetes.io/hostname
```


`file: redis-deploy-podaffinity.yaml`

```
....
  template:
...
    spec:
      containers:
      - image: schoolofdevops/redis:latest
        imagePullPolicy: Always
        name: redis
        ports:
        - containerPort: 6379
          protocol: TCP
      restartPolicy: Always

      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: role
                operator: In
                values:
                - redis
            topologyKey: "kubernetes.io/hostname"
```


clean up the previous deployments and apply as

```
kubectl delete deploy vote
kubectl delete deploy,sts redis

kubectl apply -f redis-deploy-podaffinity.yaml
kubectl apply -f vote-deploy-podaffinity.yaml


```

check the pods distribution

```
kubectl get pods -o wide --selector="role in (vote,redis)"
```

Observations from the above output,

  * Since redis has a hard constraint not to be on the same node, you would observe redis pods being on differnt nodes  (node2 and node4)
  * since vote app has a soft constraint, you see some of the pods running on node4 (same node running redis), others continue to run on node 3

If you kill the pods on node3, at the time of  scheduling new ones, scheduler meets all affinity rules

Now try scaling up redis instances

```
kubectl scale deploy/redis --replicas=4
kubectl get pods -o wide
```

  * Are all redis pods runnning ? Why?


When you are done experimenting, revert to original configurations

```
kubectl delete deploy vote
kubectl delete deploy redis
kubectl apply -f vote-deploy.yaml -f redis-deploy.yaml
```

## Taints and Tolerations

  * Affinity is defined for pods
  * Taints are defined for nodes


You could add the taints with criteria and effects. Effetcs can be

**Taint Specs**:   

  * effect  
    * NoSchedule  
    * PreferNoSchedule  
    * NoExecute  
  * key  
  * value  
  * timeAdded (only written for NoExecute taints)  


Observe the pods distribution

```
kubectl get pods -o wide

```

Lets taint a node.

```
kubectl taint node kind-worker2 dedicated=worker:NoExecute

kubectl describe node kind-worker2
```


after tainting the node

```
kubectl get pods -o wide
```

All pods running on node2 just got evicted.

Add toleration in the Deployment for worker.

`File: worker-deploy.yml`

```
apiVersion: apps/v1
.....
  template:
....
    spec:
      containers:
        - name: app
          image: schoolofdevops/vote-worker:latest

      tolerations:
        - key: "dedicated"
          operator: "Equal"
          value: "worker"
          effect: "NoExecute"
```

apply

```
kubectl apply -f worker-deploy.yml

```

Observe the pod distribution now.


```
$ kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP             NODE
db-66496667c9-qggzd       1/1       Running   0          4h        10.233.74.74   node4
redis-5bf748dbcf-ckn65    1/1       Running   0          3m        10.233.71.26   node3
redis-5bf748dbcf-vxppx    1/1       Running   0          31m       10.233.74.79   node4
result-5c7569bcb7-4fptr   1/1       Running   0          4h        10.233.71.18   node3
result-5c7569bcb7-s4rdx   1/1       Running   0          4h        10.233.74.75   node4
vote-56bf599b9c-22lpw     1/1       Running   0          30m       10.233.74.80   node4
vote-56bf599b9c-4l6bc     1/1       Running   0          12m       10.233.74.83   node4
vote-56bf599b9c-bqsrq     1/1       Running   0          12m       10.233.74.82   node4
vote-56bf599b9c-xw7zc     1/1       Running   0          12m       10.233.74.81   node4
worker-6cc8dbd4f8-6bkfg   1/1       Running   0          1m        10.233.75.15   node2
```

You should see worker being scheduled on kind-worker2


To remove the taint created above

```
kubectl taint node kind-worker2 dedicated=worker:NoExecute-
```

## Exercise

  * Master node is unschedulable because of a taint. Find the taint on the master node and remove it. See if new pods get scheduled on it after that.

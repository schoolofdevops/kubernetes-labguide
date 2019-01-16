# Advanced Pod Scheduling

In the Kubernetes bootcamp training, we have seen how to create a pod and and some basic pod configurations to go with it. But this chapter explains some advanced topics related to pod scheduling.


From the [api document for version 1.11](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/#pod-v1-core) following are the pod specs which are relevant from scheduling perspective.

  * nodeSelector
  * nodeName
  * affinity
  * schedulerName
  * tolerations


## Using Node Selectors

```
kubectl get nodes --show-labels

kubectl label nodes <node-name> zone=aaa

kubectl get nodes --show-labels

```

e.g.
```
kubectl label nodes node1 zone=bbb
kubectl label nodes node2 zone=bbb
kubectl label nodes node3 zone=aaa
kubectl label nodes node4 zone=aaa
kubectl get nodes --show-labels
```


[sample output]

```
NAME      STATUS    ROLES         AGE       VERSION   LABELS
node1     Ready     master,node   22h       v1.10.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node1,node-role.kubernetes.io/master=true,node-role.kubernetes.io/node=true,zone=bbb
node2     Ready     master,node   22h       v1.10.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node2,node-role.kubernetes.io/master=true,node-role.kubernetes.io/node=true,zone=bbb
node3     Ready     node          22h       v1.10.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node3,node-role.kubernetes.io/node=true,zone=aaa
node4     Ready     node          21h       v1.10.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node4,node-role.kubernetes.io/node=true,zone=aaa
```

Check how the pods are distributed on the nodes using the following command,

```
kubectl get pods -o wide --selector="role=vote"
NAME                    READY     STATUS    RESTARTS   AGE       IP               NODE
vote-5d88d47fc8-6rflg   1/1       Running   0          1m        10.233.75.9      node2
vote-5d88d47fc8-gbzbq   1/1       Running   0          1h        10.233.74.76     node4
vote-5d88d47fc8-q4vj6   1/1       Running   0          1h        10.233.102.133   node1
vote-5d88d47fc8-znd2z   1/1       Running   0          1m        10.233.71.20     node3
```

From the above output, you could see that the pods running **vote** app are currently equally distributed.
Now, update pod definition to make it schedule only on nodes in zone **bbb**

`file: k8s-code/pods/vote-pod.yml`

```

....

template:
...
  spec:
    containers:
      - name: app
        image: schoolofdevops/vote:v1
        ports:
          - containerPort: 80
            protocol: TCP
    nodeSelector:
      zone: 'bbb'
```

For this change, pod needs to be re created.

```
kubectl apply -f vote-pod.yml
```

You would notice that, the moment you make that change, a new rollout kicks off, which will start redistributing the pods, now following the **nodeSelector** constraint that you added.

Watch the output of the following command

```

watch kubectl get pods -o wide --selector="role=vote"

```

You will see the following while it transitions

```

NAME                        READY     STATUS              RESTARTS   AGE       IP               NODE
pod/vote-5d88d47fc8-6rflg   0/1       Terminating         0          5m        10.233.75.9      node2
pod/vote-5d88d47fc8-gbzbq   0/1       Terminating         0          1h        10.233.74.76     node4
pod/vote-5d88d47fc8-q4vj6   0/1       Terminating         0          1h        10.233.102.133   node1
pod/vote-67d7dd8f89-2w5wl   1/1       Running             0          44s       10.233.75.10     node2
pod/vote-67d7dd8f89-gm6bq   0/1       ContainerCreating   0          2s        <none>           node2
pod/vote-67d7dd8f89-w87n9   1/1       Running             0          44s       10.233.102.134   node1
pod/vote-67d7dd8f89-xccl8   1/1       Running             0          44s       10.233.102.135   node1

```

and after the rollout completes,

```

NAME                    READY     STATUS    RESTARTS   AGE       IP               NODE
vote-67d7dd8f89-2w5wl   1/1       Running   0          2m        10.233.75.10     node2
vote-67d7dd8f89-gm6bq   1/1       Running   0          1m        10.233.75.11     node2
vote-67d7dd8f89-w87n9   1/1       Running   0          2m        10.233.102.134   node1
vote-67d7dd8f89-xccl8   1/1       Running   0          2m        10.233.102.135   node1

```

#### Exercise

Just like **nodeSelector** above, you could enforce a pod to run on a specific node using **nodeName**. Try using that property to run all pods for results application on **node3**



## Defining  affinity and anti-affinity

We have discussed about scheduling a pod on a particular node using **NodeSelector**, but using node selector is a hard condition. If the condition is not met, the pod cannot be scheduled. Node/Pod affinity and anti-affinity solves this issue by introducing soft and hard conditions.

  * required
  * preferred

  * DuringScheduling
  * DuringExecution


Operators

  * In
  * NotIn
  * Exists
  * DoesNotExist
  * Gt
  * Lt

### Node Affinity


Examine  the current pod distribution  


```
kubectl get pods -o wide --selector="role=vote"

```


```
NAME                    READY     STATUS    RESTARTS   AGE       IP               NODE
vote-8546bbd84d-22d6x   1/1       Running   0          35s       10.233.102.137   node1
vote-8546bbd84d-8f9bc   1/1       Running   0          1m        10.233.102.136   node1
vote-8546bbd84d-bpg8f   1/1       Running   0          1m        10.233.75.12     node2
vote-8546bbd84d-d8j9g   1/1       Running   0          1m        10.233.75.13     node2
```


and node labels
```
kubectl get nodes --show-labels

```

```
NAME      STATUS    ROLES         AGE       VERSION   LABELS
node1     Ready     master,node   1d        v1.10.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node1,node-role.kubernetes.io/master=true,node-role.kubernetes.io/node=true,zone=bbb
node2     Ready     master,node   1d        v1.10.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node2,node-role.kubernetes.io/master=true,node-role.kubernetes.io/node=true,zone=bbb
node3     Ready     node          1d        v1.10.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node3,node-role.kubernetes.io/node=true,zone=aaa
node4     Ready     node          1d        v1.10.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node4,node-role.kubernetes.io/node=true,zone=aaa
```


Lets create node affinity criteria as

  * Pods for vote app **must** not run on the master nodes
  * Pods for vote app **preferably** run on a node in zone **bbb**

First is a **hard** affinity versus second being **soft** affinity.

`file: vote-deploy.yaml`

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
              - key: node-role.kubernetes.io/master
                operator: DoesNotExist
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
              preference:
                matchExpressions:
                - key: zone
```


### Pod Affinity


Lets define pod affinity criteria as,

  * Pods for **vote** and **redis** should be co located as much as possible (preferred)
  * No two pods with **redis** app should be running on the same node (required)


```
kubectl get pods -o wide --selector="role in (vote,redis)"

```
[sample output]
```
NAME                     READY     STATUS    RESTARTS   AGE       IP               NODE
redis-6555998885-4k5cr   1/1       Running   0          4h        10.233.71.19     node3
redis-6555998885-fb8rk   1/1       Running   0          4h        10.233.102.132   node1
vote-74c894d6f5-bql8z    1/1       Running   0          22m       10.233.74.78     node4
vote-74c894d6f5-nnzmc    1/1       Running   0          21m       10.233.71.22     node3
vote-74c894d6f5-ss929    1/1       Running   0          22m       10.233.74.77     node4
vote-74c894d6f5-tpzgm    1/1       Running   0          22m       10.233.71.21     node3
```


`file: vote-deploy.yaml`

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


`file: redis-deploy.yaml`

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


apply

```
kubectl apply -f redis-deploy.yaml
kubectl apply -f vote-deploy.yaml

```

check the pods distribution

```
kubectl get pods -o wide --selector="role in (vote,redis)"
```

[sample output ]

```
NAME                     READY     STATUS    RESTARTS   AGE       IP             NODE
redis-5bf748dbcf-gr8zg   1/1       Running   0          13m       10.233.75.14   node2
redis-5bf748dbcf-vxppx   1/1       Running   0          13m       10.233.74.79   node4
vote-56bf599b9c-22lpw    1/1       Running   0          12m       10.233.74.80   node4
vote-56bf599b9c-nvvfd    1/1       Running   0          13m       10.233.71.25   node3
vote-56bf599b9c-w6jc9    1/1       Running   0          13m       10.233.71.23   node3
vote-56bf599b9c-ztdgm    1/1       Running   0          13m       10.233.71.24   node3
```

Observations from the above output,

  * Since redis has a hard constraint not to be on the same node, you would observe redis pods being on differnt nodes  (node2 and node4)
  * since vote app has a soft constraint, you see some of the pods running on node4 (same node running redis), others continue to run on node 3

If you kill the pods on node3, at the time of  scheduling new ones, scheduler meets all affinity rules

```
$ kubectl delete pods vote-56bf599b9c-nvvfd vote-56bf599b9c-w6jc9 vote-56bf599b9c-ztdgm
pod "vote-56bf599b9c-nvvfd" deleted
pod "vote-56bf599b9c-w6jc9" deleted
pod "vote-56bf599b9c-ztdgm" deleted


$ kubectl get pods -o wide --selector="role in (vote,redis)"
NAME                     READY     STATUS    RESTARTS   AGE       IP             NODE
redis-5bf748dbcf-gr8zg   1/1       Running   0          19m       10.233.75.14   node2
redis-5bf748dbcf-vxppx   1/1       Running   0          19m       10.233.74.79   node4
vote-56bf599b9c-22lpw    1/1       Running   0          19m       10.233.74.80   node4
vote-56bf599b9c-4l6bc    1/1       Running   0          20s       10.233.74.83   node4
vote-56bf599b9c-bqsrq    1/1       Running   0          20s       10.233.74.82   node4
vote-56bf599b9c-xw7zc    1/1       Running   0          19s       10.233.74.81   node4
```



## Taints and tolerations

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
$ kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP             NODE
db-66496667c9-qggzd       1/1       Running   0          4h        10.233.74.74   node4
redis-5bf748dbcf-gr8zg    1/1       Running   0          27m       10.233.75.14   node2
redis-5bf748dbcf-vxppx    1/1       Running   0          27m       10.233.74.79   node4
result-5c7569bcb7-4fptr   1/1       Running   0          4h        10.233.71.18   node3
result-5c7569bcb7-s4rdx   1/1       Running   0          4h        10.233.74.75   node4
vote-56bf599b9c-22lpw     1/1       Running   0          26m       10.233.74.80   node4
vote-56bf599b9c-4l6bc     1/1       Running   0          8m        10.233.74.83   node4
vote-56bf599b9c-bqsrq     1/1       Running   0          8m        10.233.74.82   node4
vote-56bf599b9c-xw7zc     1/1       Running   0          8m        10.233.74.81   node4
worker-7c98c96fb4-7tzzw   1/1       Running   1          4h        10.233.75.8    node2
```

Lets taint a node.

```
kubectl taint node node2 dedicated=worker:NoExecute
```


after taining the node

```
$ kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP               NODE
db-66496667c9-qggzd       1/1       Running   0          4h        10.233.74.74     node4
redis-5bf748dbcf-ckn65    1/1       Running   0          2m        10.233.71.26     node3
redis-5bf748dbcf-vxppx    1/1       Running   0          30m       10.233.74.79     node4
result-5c7569bcb7-4fptr   1/1       Running   0          4h        10.233.71.18     node3
result-5c7569bcb7-s4rdx   1/1       Running   0          4h        10.233.74.75     node4
vote-56bf599b9c-22lpw     1/1       Running   0          29m       10.233.74.80     node4
vote-56bf599b9c-4l6bc     1/1       Running   0          11m       10.233.74.83     node4
vote-56bf599b9c-bqsrq     1/1       Running   0          11m       10.233.74.82     node4
vote-56bf599b9c-xw7zc     1/1       Running   0          11m       10.233.74.81     node4
worker-7c98c96fb4-46ltl   1/1       Running   0          2m        10.233.102.140   node1
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

You should see worker being scheduled on node2


To remove the taint created above

```
kubectl taint node node2 dedicate:NoExecute-
```

# LAB K105 - Load Balancing and Service Discovery with Services

In this lab, you would not only publish the application deployed with replicaset earlier, but also learn about the load balancing and service discovery features offered by kubernetes.


Concepts related to Kubernetes Services are depicted in the following diagram,


![kubernetes service.\label{fig:captioned_image}](images/k8s_service.jpg)


### Publishing external facing app with NodePort

Kubernetes comes with four types of services viz.

  * ClusterIP
  * NodePort
  * LoadBalancer
  * ExternalName

Lets create a  service of type **NodePort** to understand how it works.

To check the status of kubernetes objects,

```
kubectl get pods,rs,svc
```

You could also start watching the above output for changes. To do so, open a separate terminal window and run,

```
watch -n 1 kubectl get  pod,deploy,rs,svc
```


Refer to [Service Specs](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/#service-v1-core) to understand the properties that you could write.

`filename: vote-svc.yaml`

```
---
apiVersion: v1
kind: Service
metadata:
  name: vote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30000
  type: NodePort

```


Apply this file to to create a service

```
kubectl apply -f vote-svc.yaml --dry-run
kubectl apply -f vote-svc.yaml
kubectl get svc
kubectl describe service vote
```

[Sample Output of describe command]

```
Name:                     vote
Namespace:                instavote
Labels:                   role=svc
                          tier=front
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"role":"svc","tier":"front"},"name":"vote","namespace":"instavote"},"spec":{...
Selector:                 app=vote
Type:                     NodePort
IP:                       10.108.108.157
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31429/TCP
Endpoints:                10.38.0.4:80,10.38.0.5:80,10.38.0.6:80 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Observe the following

  * Selector
  * TargetPort
  * NodePort
  * Endpoints


Go to browser and check http://HOSTIP:NODEPORT

Here the node port is 30000 (as defined by nodePort in service spec).

Sample output will be:

![Vote](images/vote-rc.png)


If you refresh the page, you should also notice its sending traffic to diffent pod each time, in round robin fashion.


#### Exercises  

  * Change the selector criteria to use a non existant label. Use `kubectl edit svc./vote` to update and apply the configuration. Observe the output of describe command and check the endpoints. Do you see any ?  How are  selectors and pod labels related ?
  * Observe the number of endpoints. Then change the scale of replicas created by the replicasets. Does it have any effect on the number of endpoints ?    



#### Services Under the Hood

Lets traverse the route of the **network packet** that comes in on port 30000 on any node in your cluster.

```
iptables -nvL -t nat  
iptables -nvL -t nat  | grep 30000
```

Anything that comes on dpt:3000, gets forwarded to the chain created for that service.

```
iptables -nvL -t nat  | grep KUBE-SVC-VIQHAVHDK4QE7NA4  -A 10

```


```
Chain KUBE-SVC-VIQHAVHDK4QE7NA4 (2 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 KUBE-SEP-RFJGHFMXUDJXIEW6  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* instavote/vote: */ statistic mode random probability 0.20000000019
    0     0 KUBE-SEP-GBR5YQCVRYY3BA6U  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* instavote/vote: */ statistic mode random probability 0.25000000000
    0     0 KUBE-SEP-BAI3HQ7SV7RZ2CI6  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* instavote/vote: */ statistic mode random probability 0.33332999982
    0     0 KUBE-SEP-2EQSLPEP3WDOTI5J  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* instavote/vote: */ statistic mode random probability 0.50000000000
    0     0 KUBE-SEP-2CJQISP4W7F2HCRW  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* instavote/vote: */

```

Where,
  * counrt of KUBE-SEP-xxx matches number of pods
  * KUBE-SEP-BAI3HQ7SV7RZ2CI6  is a chain created for one of host. examine that next

```


iptables -nvL -t nat  | grep KUBE-SEP-BAI3HQ7SV7RZ2CI6  -A 3
```

[output]
```
pkts bytes target     prot opt in     out     source               destination
    0     0 KUBE-MARK-MASQ  all  --  *      *       10.32.0.6            0.0.0.0/0            /* instavote/vote: */
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* instavote/vote: */ tcp to:10.32.0.6:80
--
```

where the packet is being forwarded to 10.32.0.6, which should corraborate with the ip of the pod

e.g.

```
kubectl get pods -o wide

NAME         READY     STATUS    RESTARTS   AGE       IP          NODE
vote-58bpv   1/1       Running   0          1h        10.32.0.6   k-02
vote-986cl   1/1       Running   0          1h        10.38.0.5   k-03
vote-9rrfz   1/1       Running   0          1h        10.38.0.4   k-03
vote-dx8f4   1/1       Running   0          1h        10.32.0.4   k-02
vote-qxmfl   1/1       Running   0          1h        10.32.0.5   k-02
```

 10.32.0.6 matches ip of vote-58bpv  


to check how the packet is routed next use,

```
route -n
```

[output]
```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         206.189.144.1   0.0.0.0         UG    0      0        0 eth0
10.15.0.0       0.0.0.0         255.255.0.0     U     0      0        0 eth0
10.32.0.0       0.0.0.0         255.240.0.0     U     0      0        0 weave
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
206.189.144.0   0.0.0.0         255.255.240.0   U     0      0        0 eth0

```

where, 10.32.0.0 is going over **weave** interface.




### Exposing app with ExternalIP

Observe the output of service list, specifically note the **EXTERNAL-IP** colum in the output.

```
kubectl  get svc
```

Now, update the service spec and add external IP configs. Pick IP addresses of any two nodes  (You could add one or more) and it to the spec as,

```
kubectl edit svc vote
```

[sample file edit]

```
---
apiVersion: v1
kind: Service
metadata:
  name: vote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30000
  type: NodePort
  externalIPs:
    - xx.xx.xx.xx
    - yy.yy.yy.yy
```

Where

replace xx.xx.xx.xx and yy.yy.yy.yy with IP addresses of the nodes on two of the kubernetes hosts.


apply
```
kubectl  get svc
kubectl apply -f vote-svc.yaml
kubectl  get svc
kubectl describe svc vote
```

[sample output]

```
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP                    PORT(S)        AGE
vote      NodePort   10.107.71.204   206.189.150.190,159.65.8.227   80:30000/TCP   11m
```

where,

EXTERNAL-IP column shows which IPs the application is been exposed on. You could go to http://<IPADDRESS>:<SERVICE_PORT> to access this application.  e.g. http://206.189.150.190:80 where you should replace 206.189.150.190 with the actual IP address of the node that you exposed this on.


## Internal Service Discovery

Kubernetes not only allows you to publish external facing apps with the services, but also allows you to discover other components of your application stack with the clusterIP and DNS attached to it.

Before you begin adding service discovery,

  * Visit the vote app from browser
  * Attempt to vote by clicking on one of the options

observe what happens. Does it go through?  


Debugging,


```
kubectl get pod
kubectl exec vote-xxxx nslookup redis

```
[replace xxxx with the actual pod id of one of the vote pods ]

keep the above command on a watch. You should create a new terminal to run the watch command.

e.g.

```
kubectl exec -it vote-xxxx sh
watch  kubectl exec vote-xxxx ping redis
```
where, vote-xxxx is one of the vote pods that I am running. Replace this with the actual pod id.


Now create **redis** service

```
kubectl apply -f redis-svc.yaml

kubectl get svc

kubectl describe svc redis
```

Watch the nslookup screen  and observe if its able to resolve **redis** by hostname and its pointing to an IP address.

e.g.

```
Name:      redis
Address 1: 10.104.111.173 redis.instavote.svc.cluster.local
```

where

  * 10.104.111.173 is the ClusterIP assigned to redis service
  *  redis.instavote.svc.cluster.local is the dns attached to the ClusterIP above

What happened here?

  * Service **redis** was created with a ClusterIP e.g. 10.102.77.6
  * A DNS entry was created for this service. The fqdn of the service is **redis.instavote.svc.cluster.local** and it takes the form of
  my-svc.my-namespace.svc.cluster.local
  * Each pod points to  internal  DNS server running in the cluster. You could see the details of this by running the following commands


```
kubectl exec vote-xxxx cat /etc/resolv.conf
```
[replace vote-xxxx with actual pod id]

[sample output]
```
nameserver 10.96.0.10
search instavote.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

where **10.96.0.10** is the ClusterIP assigned to the DNS service. You could co relate that with,

```
kubectl get svc -n kube-system


NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
kube-dns               ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP   1h
kubernetes-dashboard   NodePort    10.104.42.73   <none>        80:31000/TCP    23m

```

where, **10.96.0.10** is the ClusterIP assigned to **kube-dns** and matches the configuration in **/etc/resolv.conf** above.

#### Creating Endpoints for Redis

Service is been created, but you still need to launch the actual pods running **redis** application.

Create the endpoints now,

```
kubectl apply -f redis-deploy.yaml
kubectl describe svc redis

```

[sample output]

```
Name:              redis
Namespace:         instavote
Labels:            role=redis
                   tier=back
Annotations:       kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"role":"redis","tier":"back"},"name":"redis","namespace":"instavote"},"spec"...
Selector:          app=redis
Type:              ClusterIP
IP:                10.102.77.6
Port:              <unset>  6379/TCP
TargetPort:        6379/TCP
Endpoints:         10.32.0.6:6379,10.46.0.6:6379
Session Affinity:  None
Events:            <none>
```

Again, visit the vote app from browser, attempt to register your vote.  observe what happens. This time the vote should be registered successfully.


#### Summary

In this lab, you have published a front facing application, learnt how services are implemented under the hood as well as added service discovery to provide  connection strings automatically.


##### Reading


  * [Debugging Services](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)
  * [Kubernetes Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
  * [Service API Specs for Kubernetes Version 1.10](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#service-v1-core)

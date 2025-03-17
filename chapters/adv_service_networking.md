# Service Networking, Load Balancing and Service Discovery

In this lab, you would not only publish the application deployed with replicaset earlier, but also learn about the load balancing and service discovery features offered by kubernetes.


Concepts related to Kubernetes Services are depicted in the following diagram,


![kubernetes service.\label{fig:captioned_image}](images/k8s_service.jpg)


## Publishing external facing app with NodePort

Kubernetes comes with four types of services viz.

  * ClusterIP
  * NodePort
  * LoadBalancer
  * ExternalName

Lets deploy ad micro-service and expose it with the  **NodePort** service type in kubernetes to understand how it works.

To check the status of kubernetes objects,

```
kubectl get all
```


Create deployment and service as 

```
kubectl  create deployment vote --image=schoolofdevops/vote:v1 --replicas=5
kubectl create service nodeport vote --tcp=80 --node-port=30000
```

validate the deployment and service

```
kubectl get all
kubectl get svc,endpoints

kubectl describe svc vote
```

[Sample Output of describe command]

```
root@advk8s-01:~# kubectl describe svc vote
Name:                     vote
Namespace:                default
Labels:                   app=vote
Annotations:              <none>
Selector:                 app=vote
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.130.182
IPs:                      10.96.130.182
Port:                     80  80/TCP
TargetPort:               80/TCP
NodePort:                 80  30000/TCP
Endpoints:                10.244.1.5:80,10.244.1.3:80,10.244.2.4:80 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
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


### Exercises  

  * Change the selector criteria to use a non existant label. Use `kubectl edit svc./vote` to update and apply the configuration. Observe the output of describe command and check the endpoints. Do you see any ?  How are  selectors and pod labels related ?
  * Observe the number of endpoints. Then change the scale of replicas created by the replicasets. Does it have any effect on the number of endpoints ?    



## Services Under the Hood



Lets traverse the route of the **network packet** that comes in on port 30000 on any node in your cluster.

Connect to a node (If creatd using KIND) with, 


```
docker exec -it --privileged kind-worker2 sh
```
and then check the IPTables config as, 

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

  * count of KUBE-SEP-xxx matches number of pods.
  * KUBE-SEP-BAI3HQ7SV7RZ2CI6 is an example of a chain created for one of the hosts. Examine that next.

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
ip route show
```

[output]
```
default via 128.199.128.1 dev eth0 onlink
10.15.0.0/16 dev eth0  proto kernel  scope link  src 10.15.0.10
10.32.0.0/12 dev weave  proto kernel  scope link  src 10.32.0.1
128.199.128.0/18 dev eth0  proto kernel  scope link  src 128.199.185.90
172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1 linkdown

```

where, 10.32.0.0/12 is going over **weave** interface.




## Summary

In this lab, you have published a front facing application and  learnt how services are implemented under the hood with the use of kernel level network routing with iptables. 


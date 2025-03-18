# Horizontally Autoscaling Apps with HPA

With Horizontal Pod Autoscaling, Kubernetes automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization (or, with alpha support, on some other, application-provided metrics).

The Horizontal Pod Autoscaler is implemented as a Kubernetes API resource and a controller. The resource determines the behavior of the controller. The controller periodically adjusts the number of replicas in a replication controller or deployment to match the observed average CPU utilization to the target specified by user

### Prerequisites

Check if essential  monitoring is setup

```
kubectl top pod

kubectl top node
```

where expected output should be similar to,

```
kubectl top node

NAME                 CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
kind-control-plane   123m         6%     688Mi           17%
kind-worker          39m          1%     498Mi           12%
kind-worker2         31m          1%     422Mi           10%
```
If you see a similar output, monitoring is now been setup.

If not, you may have to deploy the metric server using intructions provided during the class. 



### Create a HPA

To demonstrate Horizontal Pod Autoscaler we will use a custom docker image based on the php-apache image

`file: vote-hpa.yaml`


```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vote
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: ContainerResource
      containerResource:
        name: cpu
        container: vote  # change this as per actual container name
        target:
          type: Utilization
          averageUtilization: 50
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment # change it to Deployment if have created a deployment already
    name: vote
  behavior:
    scaleDown:
      policies:
      - type: Pods
        value: 2
        periodSeconds: 120
      - type: Percent
        value: 25
        periodSeconds: 120
      stabilizationWindowSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 45
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 2
        periodSeconds: 15
      selectPolicy: Max
```

apply

```
kubectl apply -f vote-hpa.yaml
```

Validate

```
kubectl get hpa
```

you should see

[sample output]
```
NAME   REFERENCE         TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
vote   Deployment/vote   2%/50%    2         10        2          12m
```

If you see `unknown/50%` under `TARGETS`, there is a underlying issue that you may need to resolve.  Check if you have defined the resources (requests, limits) for the containers in the pod.


```
kubectl describe hpa vote

kubectl get pod,deploy


```

###  Launch a Load Test

If you do not have the service for `vote` app to receive the traffic, create one as

```
cd k8s-code/projects/instavote/dev
kubectl apply -f vote-svc.yaml
```

Prepare to monitor the autoscaling by opening a new window connected to your cluster and by launching,

```
watch 'kubectl top pods;kubectl get pods;  kubectl get hpa ; kubectl get deploy'
```


Create a Load Test Job as,

`file: loadtest-job.yaml`

```
apiVersion: batch/v1
kind: Job
metadata:
  generateName: loadtest
spec:
  template:
    spec:
      containers:
      - name: siege
        image: schoolofdevops/loadtest:v1
        command: ["siege",  "--concurrent=1", "--benchmark", "--time=4m", "http://vote"]
      restartPolicy: Never
  backoffLimit: 4
```

and launch it as

```
kubectl create -f loadtest-job.yaml
```

This will launch a one off Job which would run for 4 minutes.  

To get information about the job

```
kubectl get jobs
kubectl describe  job loadtest-xxx

```
replace loadtest-xxx with actual job name.

To check the load test output

```
kubectl logs  -f loadtest-xxxx
```
[replace **loadtest-xxxx** with the actual pod id.]



[Sample Output]

```
** SIEGE 3.0.8
** Preparing 15 concurrent users for battle.
root@kube-01:~# kubectl logs vote-loadtest-tv6r2 -f
** SIEGE 3.0.8
** Preparing 15 concurrent users for battle.

.....


Lifting the server siege...      done.

Transactions:		       41618 hits
Availability:		       99.98 %
Elapsed time:		      299.13 secs
Data transferred:	      127.05 MB
Response time:		        0.11 secs
Transaction rate:	      139.13 trans/sec
Throughput:		        0.42 MB/sec
Concurrency:		       14.98
Successful transactions:       41618
Failed transactions:	           8
Longest transaction:	        3.70
Shortest transaction:	        0.00

FILE: /var/log/siege.log
You can disable this annoying message by editing
the .siegerc file in your home directory; change
the directive 'show-logfile' to false.
```
Now check the job status again,

```
kubectl get jobs
NAME            DESIRED   SUCCESSFUL   AGE
vote-loadtest   1         1            10m

```

While it is running,



  * Keep monitoring for the load on the pod as the job progresses.
  * You should see hpa in action as it scales out/in the  vote deployment with the increasing/decreasing load.


## Scaling Apps Vertically with VPA

Before you begin, ensure OpenSSL is installed and is up to date on your system. 


Clone the [kubernetes/autoscaler](https://github.com/kubernetes/autoscaler) GitHub repository and switch to path where VPA manifests are, 

```
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler/ 

```


deploy VPA as 

```
./hack/vpa-up.sh
```



Create a VPA Policy 

File: `vote-vpa.yaml`

```
---
apiVersion: "autoscaling.k8s.io/v1"
kind: VerticalPodAutoscaler
metadata:
  name: vote
  labels:
    role: vote
spec:
  # recommenders field can be unset when using the default recommender.
  # When using an alternative recommender, the alternative recommender's name
  # can be specified as the following in a list.
  # recommenders:
  #   - name: 'alternative'
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: vote
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: '*'
        minAllowed:
          cpu: 50m
          memory: 64Mi
        maxAllowed:
          cpu: 500
          memory: 512Mi
        controlledResources: ["cpu", "memory"]
```


apply 

```
kubectl apply -f vote-vpa.yaml
```

watch 

```
kubectl get vpa vote --watch

```

[sample output]

```
NAME   MODE   CPU   MEM   PROVIDED   AGE
vote   Auto                          5s
vote   Auto   50m   262144k   True       30s
```



Create a Load Test Job if not already present as,

`file: loadtest-job.yaml`

```
apiVersion: batch/v1
kind: Job
metadata:
  generateName: loadtest
spec:
  template:
    spec:
      containers:
      - name: siege
        image: schoolofdevops/loadtest:v1
        command: ["siege",  "--concurrent=1", "--benchmark", "--time=4m", "http://vote"]
      restartPolicy: Never
  backoffLimit: 4
```

and launch it as

```
kubectl create -f loadtest-job.yaml
```

This will launch a one off Job which would run for 4 minutes.


keep watching the following in differnt windows 

terminal  1 
```
kubectl get vpa vote --watch
```

terminal 2
```
kubectl get hpa vote --watch
```

you should see, both hpa and vpa doing complimentary jobs where, 
* HPA is launching new pods to ensure more requests are being served 
* VPA is updating the `requests` spec based on actual metrics 

You could check the modified request spec by VPA by describing the one of the pods runnnig vote app 

```
kubectl describe pod vote-xxxx-yyyy
```

```
   Restart Count:  0
    Limits:
      cpu:     1235m
      memory:  500Mi
    Requests:
      cpu:        247m
      memory:     262144k
    Environment:  <none>
    Mounts:
```





##### Summary

In this lab, you have successful configured and demonstrated dynamic scaling ability of kubernetes using HorizontalPodAutoscaler and VerticalPodAutoscaler together. You have also learnt about a new **jobs** controller type for running one off or batch jobs.


##### Reading List

  * [Kubernetes Monitoring Architecture](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/monitoring_architecture.md)
  * [Core Metrics Pipeline]( https://kubernetes.io/docs/tasks/debug-application-cluster/core-metrics-pipeline/)
  * [Metrics Server](https://github.com/kubernetes-incubator/metrics-server)
  * [Assigning Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/#specify-a-cpu-request-and-a-cpu-limit)
  * [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

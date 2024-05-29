# Lab 07 - Verticle Pod Autoscaler


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



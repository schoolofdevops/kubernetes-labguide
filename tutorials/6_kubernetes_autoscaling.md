# Kubernetes Horizonntal Pod Autoscaling

With Horizontal Pod Autoscaling, Kubernetes automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization (or, with alpha support, on some other, application-provided metrics).

The Horizontal Pod Autoscaler is implemented as a Kubernetes API resource and a controller. The resource determines the behavior of the controller. The controller periodically adjusts the number of replicas in a replication controller or deployment to match the observed average CPU utilization to the target specified by user

### Prerequisites

Heapster monitoring needs to be deployed in the cluster as Horizontal Pod Autoscaler uses it to collect metrics.

#### Deploying Heapster

Go to the below directory and create the deployment and services.

```
git clone https://github.com/kubernetes/heapster.git
cd heapster
kubectl apply -f deploy/kube-config/influxdb/
kubectl apply -f deploy/kube-config/rbac/heapster-rbac.yaml
```

Validate that heapster, influxdb and grafana are started
```
kubectl get pods -n kube-system
kubectl get svc -n kube-system

```

Now this will deploy the heapster monitoring.

### Run & expose php-apache server

To demonstrate Horizontal Pod Autoscaler we will use a custom docker image based on the php-apache image

```
kubectl run php-apache --image=gcr.io/google_containers/hpa-example --requests=cpu=200m --expose --port=80  
```

Sample Output
```
kubectl run php-apache --image=gcr.io/google_containers/hpa-example --requests=cpu=200m --expose --port=80
service "php-apache" created
deployment "php-apache" created
```

To verify the created pod:
```
kubectl get pods
```

Wait untill the pod changes to running state.

### Create Horizontal Pod Autoscaler
Now that the server is running, we will create the autoscaler using kubectl autoscale. The following command will create a Horizontal Pod Autoscaler that maintains between 1 and 10 replicas of the Pods controlled by the php-apache deployment we created in the first step of these instructions.

```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

Sample Output
```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
deployment "php-apache" autoscaled
```

We may check the current status of autoscaler by running:

```
kubectl get hpa
```
Sample Output:
```
kubectl get hpa
NAME         REFERENCE                     TARGET    CURRENT   MINPODS   MAXPODS   AGE
php-apache   Deployment/php-apache   50%       0%        1         10        18s
```
### Increase load

Now we can increase the load and trying testing what will happen.
We will start a container, and send an infinite loop of queries to the php-apache service

```
kubectl run -i --tty -n dev load-generator --image=busybox /bin/sh

Hit enter for command prompt

while true; do wget -q -O- http://php-apache; done

```
Now open a new window of the same machine.

And check the status of the hpa
```
kubectl get hpa
```
Sample Output:

```
kubectl get hpa
NAME         REFERENCE                     TARGET    CURRENT   MINPODS   MAXPODS   AGE
php-apache   Deployment/php-apache/scale   50%       305%      1         10        3m
```

Now if you check the pods it will be automatically scaled to the desired value.

```
kubectl get pods
```

Sample Output
```
kubectl get pods
NAME                              READY     STATUS    RESTARTS   AGE
load-generator-1930141919-1pqn0   1/1       Running   0          1h
php-apache-3815965786-2jmm9       1/1       Running   0          1h
php-apache-3815965786-4f0ck       1/1       Running   0          1h
php-apache-3815965786-73w24       1/1       Running   0          1h
php-apache-3815965786-80n2x       1/1       Running   0          1h
php-apache-3815965786-c6w0k       1/1       Running   0          1h
php-apache-3815965786-f06dg       1/1       Running   0          1h
php-apache-3815965786-nfs8d       1/1       Running   0          1h
php-apache-3815965786-phrhs       1/1       Running   0          1h
php-apache-3815965786-z6rnm       1/1       Running   0          1h
```

### Stop load

In the terminal where we created the container with busybox image, terminate the load generation by typing <Ctrl> + C

Then we will verify the result state (after a minute or so)
```
kubectl get hpa
NAME         REFERENCE                     TARGET    CURRENT   MINPODS   MAXPODS   AGE
php-apache   Deployment/php-apache/scale   50%       0%        1         10        11m

$ kubectl get deployment php-apache
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
php-apache   1         1         1            1           27m
```

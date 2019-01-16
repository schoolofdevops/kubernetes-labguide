# Troubleshooting the Kubernetes cluster

In this chapter we will learn about how to trouble shoot our Kubernetes cluster at *control plane* level and at *application level*.

### Troubleshooting the control plane

### Listing the nodes in a cluster
First thing to check if your cluster is working fine or not is to list the nodes associated with your cluster.

```
kubectl get nodes
```
Make sure that all nodes are in `Ready` state.

### List the control plane pods
If your nodes are up and running, next thing to check is the status of Kubernetes components.
Run,

```
kubectl get pods -n kube-system
```
If any of the pod is *restarting or crashing*, look in to the issue.
This can be done by getting the pod's description.
For example, in my cluster *kube-dns* is crashing. In order to fix this first check the deployment for errors.

```
kubectl describe deployment -n kube-system kube-dns
```

### Log files

#### Master
If your deployment is good, the next thing to look for is log files.
The locations of log files are given below...

```
/var/log/kube-apiserver.log - For API Server logs
/var/log/kube-scheduler.log - For Scheduler logs
/var/log/kube-controller-manager.log - For Replication Controller logs
```

If your Kubernetes components are running as pods, then you can get their logs by following the steps given below,
Keep in mind that the *actual pod's name may differ* from cluster to cluster...

```
kubectl logs -n kube-system -f kube-apiserver-node1
kubectl logs -n kube-system -f kube-scheduler-node1
kubectl logs -n kube-system -f kube-controller-manager-node1
```

#### Worker Nodes
In your worker, you will need to check for errors in kubelet's log...

```
sudo journalctl -u  kubelet
```

### Troubleshooting the application
Sometimes your application(pod) may fail to start because of various reasons. Let's see how to troubleshoot.

### Getting detailed status of an object (pods, deployments)

**object.status** shows a detailed information about whats the status of an object ( e.g. pod) and why its in that condition. This can be very useful to identify the issues.

Example
```
kubectl get pod vote -o yaml

```

example output snippet when a wrong image was used to create a pod. 

```
status:
...
containerStatuses:
....
state:
  waiting:
    message: 'rpc error: code = Unknown desc = Error response from daemon: manifest
      for schoolofdevops/vote:latst not found'
    reason: ErrImagePull
hostIP: 139.59.232.248
```



### Checking the status of Deployment
For this example I have a sample deployment called nginx.

`FILE: nginx-deployment.yml`

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: ngnix:latest
          ports:
            - containerPort: 80
```

List the deployment to check for the *availability of pods*

```
kubectl get deployment nginx

NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx         1         1         1            0           20h
```
It is clear that my pod is unavailable. Lets dig further.

Check the *events* of your deployment.

```
kubectl describe deployment nginx
```

List the pods to check for any *registry related error*

```
kubectl get pods

NAME                           READY     STATUS             RESTARTS   AGE
nginx-57c88d7bb8-c6kpc         0/1       ImagePullBackOff   0          7m
```

As we can see, we are not able to pull the image(*ImagePullBackOff*). Let's investigate further.

```
kubectl describe pod nginx-57c88d7bb8-c6kpc
```
Check the events of the pod's description.

```
Events:
  Type     Reason                 Age               From                                               Message
  ----     ------                 ----              ----                                               -------
  Normal   Scheduled              9m                default-scheduler                                  Successfully assigned nginx-57c88d7bb8-c6kpc to ip-11-0-1-111.us-west-2.compute.internal
  Normal   SuccessfulMountVolume  9m                kubelet, ip-11-0-1-111.us-west-2.compute.internal  MountVolume.SetUp succeeded for volume "default-token-8cwn4"
  Normal   Pulling                8m (x4 over 9m)   kubelet, ip-11-0-1-111.us-west-2.compute.internal  pulling image "ngnix"
  Warning  Failed                 8m (x4 over 9m)   kubelet, ip-11-0-1-111.us-west-2.compute.internal  Failed to pull image "ngnix": rpc error: code = Unknown desc = Error response from daemon: repository ngnix not found: does not exist or no pull access
  Normal   BackOff                7m (x6 over 9m)   kubelet, ip-11-0-1-111.us-west-2.compute.internal  Back-off pulling image "ngnix"
  Warning  FailedSync             4m (x24 over 9m)  kubelet, ip-11-0-1-111.us-west-2.compute.internal  Error syncing pod
```

Bingo! The name of the image is `ngnix` instead of `nginx`. So *fix the typo in your deployment file and redo the deployment*.

Sometimes, your application(pod) may fail to start because of some configuration issues. For those errors, we can follow the logs of the pod.

```
kubectl logs -f nginx-57c88d7bb8-c6kpc
```

If you have any errors it will get populated in your logs.

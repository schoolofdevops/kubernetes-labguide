# Lab K204 - Adding health checks with Probes


## Adding health checks

Health checks in Kubernetes work the same way as traditional health checks of applications. They make sure that our application is ready to receive and process user requests. In Kubernetes we have two types of health checks,
  * Liveness Probe
  * Readiness Probe
Probes are simply a *diagnostic action* performed by the kubelet. There are three types actions a kubelet perfomes on a pod, which are namely,

  * `ExecAction`: Executes a command inside the pod. Assumed successful when the command **returns 0** as exit code.
  * `TCPSocketAction`: Checks for a state of a particular port on the pod. Considered successful when the state of **the port is open**.
  * `HTTPGetAction`: Performs a GET request on pod's IP. Assumed successful when the status code is **greater than 200 and less than 400**

In cases of any failure during the diagnostic action, kubelet will report back to the API server. Let us study about how these health checks work in practice.

### Adding Liveness/Readineess  Probes
Liveness probe checks the status of the pod(whether it is running or not). If livenessProbe fails, then the pod is subjected to its restart policy. The default state of livenessProbe is *Success*.

Readiness probe checks whether your application is ready to serve the requests. When the readiness probe fails, the pod's IP is removed from the end point list of the service. The default state of readinessProbe is *Success*.

Let us add liveness/readiness  probes to our *vote* deployment.

`file: vote-deploy-probes.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  revisionHistoryLimit: 4
  replicas: 12
  minReadySeconds: 20
  selector:
    matchLabels:
      role: vote
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3, v4, v5]}
  template:
    metadata:
      name: vote
      labels:
        app: python
        role: vote
        version: v1
    spec:
      containers:
        - name: app
          image: schoolofdevops/vote:v1
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
          livenessProbe:
            tcpSocket:
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 3
```

where,

  * livenessProbe used a simple tcp check to test whether application is listening on port 80
  * readinessProbe does httpGet to actually fetch a page using get method and tests for the http response code.


Apply this code using,

```
kubectl apply -f vote-deploy-probes.yaml
kubectl get pods
kubectl describe svc vote

```

### Testing livenessProbe

```
kubectl edit deploy vote
```

```
livenessProbe:
  failureThreshold: 3
  initialDelaySeconds: 5
  periodSeconds: 5
  successThreshold: 1
  tcpSocket:
    port: 8888
  timeoutSeconds: 1
```

Since you are using *edit* command, as soon as you save the file, deployment is modified.

```
kubectl get pods
kubectl describe pod vote-xxxx
```

where, vote-xxxx is one of the new pods created.

```
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  38s                default-scheduler  Successfully assigned instavote/vote-668579766d-p65xb to k-02
  Normal   Pulled     18s (x2 over 36s)  kubelet, k-02      Container image "schoolofdevops/vote:v1" already present on machine
  Normal   Created    18s (x2 over 36s)  kubelet, k-02      Created container
  Normal   Started    18s (x2 over 36s)  kubelet, k-02      Started container
  Normal   Killing    18s                kubelet, k-02      Killing container with id docker://app:Container failed liveness probe.. Container will be killed and recreated.
  Warning  Unhealthy  4s (x5 over 29s)   kubelet, k-02      Liveness probe failed: dial tcp 10.32.0.12:8888: connect: connection refused
```

What just happened ?

  * Since livenessProbe is failing it will keep killing and recreating containers. Thats what you see in the description above.
  * When you list pods, you should see it in crashloopbackoff state with number of restarts incrementing with time.

e.g.

```
vote-668579766d-p65xb    0/1     CrashLoopBackOff   7          7m38s   10.32.0.12   k-02        <none>           <none>
vote-668579766d-sclbr    0/1     CrashLoopBackOff   7          7m38s   10.32.0.10   k-02        <none>           <none>
vote-668579766d-vrcmj    0/1     CrashLoopBackOff   7          7m38s   10.38.0.8    kube03-01   <none>           <none>
```

To fix it, revert the livenessProbe configs by editing the deplyment again.



### Readiness Probe


Readiness probe is configured just like liveness probe. But this time we will use *httpGet request*.

```
kubectl edit deploy vote

```

```
readinessProbe:
  failureThreshold: 3
  httpGet:
    path: /test.html
    port: 80
    scheme: HTTP
  initialDelaySeconds: 5
  periodSeconds: 3
  successThreshold: 1
```

where, readinessProbe.httpGet.path is been changed from **/** to **/test.html** which is a non existant path.  


check

```
kubectl get deploy,rs,pods
```


[output snippet]

```
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/vote    11/12   3            11          2m12s


vote-8cbb7ff89-6xvbc     0/1     Running   0          73s     10.38.0.10   kube03-01   <none>           <none>
vote-8cbb7ff89-6z5zv     0/1     Running   0          73s     10.38.0.5    kube03-01   <none>           <none>
vote-8cbb7ff89-hdmxb     0/1     Running   0          73s     10.32.0.12   k-02        <none>           <none>
```


```
kubectl describe pod vote-8cbb7ff89-hdmxb
```

where, vote-8cbb7ff89-hdmxb is one of the pods launched after changing readiness probe.

[output snippet]

```
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  109s                 default-scheduler  Successfully assigned instavote/vote-8cbb7ff89-hdmxb to k-02
  Normal   Pulled     108s                 kubelet, k-02      Container image "schoolofdevops/vote:v1" already present on machine
  Normal   Created    108s                 kubelet, k-02      Created container
  Normal   Started    108s                 kubelet, k-02      Started container
  Warning  Unhealthy  39s (x22 over 102s)  kubelet, k-02      Readiness probe failed: HTTP probe failed with statuscode: 404
```


```
kubectl describe svc vote
```

what happened ?

  * Since readinessProbe failed, the new launched batch does not show containers running (0/1)
  * Description of the pod shows it being *Unhealthy* due to failed HTTP probe
  * Deployment shows surged pods, with number of ready pods being less than number of desired replicas (e.g. 11/12).
  * Service does not send traffic to the pod which are marked as unhealthy/not ready.  


Reverting the changes to readiness probe should bring it back to working state.

# Checking health with Probes


## Adding health checks

Health checks in Kubernetes work the same way as traditional health checks of applications. They make sure that our application is ready to receive and process user requests. In Kubernetes we have two types of health checks,
  * Liveness Probe
  * Readiness Probe
Probes are simply a *diagnostic action* performed by the kubelet. There are three types actions a kubelet perfomes on a pod, which are namely,
  * `ExecAction`: Executes a command inside the pod. Assumed successful when the command **returns 0** as exit code.
  * `TCPSocketAction`: Checks for a state of a particular port on the pod. Considered successful when the state of **the port is open**.
  * `HTTPGetAction`: Performs a GET request on pod's IP. Assumed successful when the status code is **greater than 200 and less than 400**

In cases of any failure during the diagnostic action, kubelet will report back to the API server. Let us study about how these health checks work in practice.

### Liveness Probe
Liveness probe checks the status of the pod(whether it is running or not). If livenessProbe fails, then the pod is subjected to its restart policy. The default state of livenessProbe is *Success*.

Let us add liveness probe to our *frontend* deployment. The following probe will check whether it is able to *access the port or not*.

`File: code/frontend-deploy.yml`

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: front-end
        env: dev
    spec:
      containers:
        - name: front-end
          image: schoolofdevops/frontend
          imagePullPolicy: Always
          ports:
            - containerPort: 8079
          livenessProbe:
            tcpSocket:
              port: 8079
            initialDelaySeconds: 5
            periodSeconds: 5
```

`Expected output:`

```
kubectl apply -f front-end/frontend-deploy.yml
kubectl get pods
kubectl describe pod front-end-757db58546-fkgdw

[...]
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              22s   default-scheduler  Successfully assigned front-end-757db58546-fkgdw to node4
  Normal  SuccessfulMountVolume  22s   kubelet, node4     MountVolume.SetUp succeeded for volume "default-token-w4279"
  Normal  Pulling                20s   kubelet, node4     pulling image "schoolofdevops/frontend"
  Normal  Pulled                 17s   kubelet, node4     Successfully pulled image "schoolofdevops/frontend"
  Normal  Created                17s   kubelet, node4     Created container
  Normal  Started                17s   kubelet, node4     Started container
```

Let us change the livenessProbe check port to 8080.

```
          livenessProbe:
            tcpSocket:
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```

Apply this deployment file and check the description of the pod

`Expected output:`
```
kubectl apply -f frontend-deploy.yml
kubectl get pods
kubectl describe pod front-end-bf86ffd8b-bjb7p

[...]
Events:
  Type     Reason                 Age               From               Message
  ----     ------                 ----              ----               -------
  Normal   Scheduled              1m                default-scheduler  Successfully assigned front-end-bf86ffd8b-bjb7p to node3
  Normal   SuccessfulMountVolume  1m                kubelet, node3     MountVolume.SetUp succeeded for volume "default-token-w4279"
  Normal   Pulling                38s (x2 over 1m)  kubelet, node3     pulling image "schoolofdevops/frontend"
  Normal   Killing                38s               kubelet, node3     Killing container with id docker://front-end:Container failed liveness probe.. Container will be killed and recreated.
  Normal   Pulled                 35s (x2 over 1m)  kubelet, node3     Successfully pulled image "schoolofdevops/frontend"
  Normal   Created                35s (x2 over 1m)  kubelet, node3     Created container
  Normal   Started                35s (x2 over 1m)  kubelet, node3     Started container
  Warning  Unhealthy              27s (x5 over 1m)  kubelet, node3     Liveness probe failed: Get http://10.233.71.50:8080/: dial tcp 10.233.71.50:8080: getsockopt: connection refused
```

### Readiness Probe
Readiness probe checks whether your application is ready to serve the requests. When the readiness probe fails, the pod's IP is removed from the end point list of the service. The default state of readinessProbe is *Success*.

Readiness probe is configured just like liveness probe. But this time we will use *httpGet request*.

`File: code/frontend-deploy.yml`

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: front-end
  namespace: instavote
  labels:
    app: front-end
    env: dev
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: front-end
        env: dev
    spec:
      containers:
        - name: front-end
          image: schoolofdevops/frontend
          imagePullPolicy: Always
          ports:
            - containerPort: 8079
          livenessProbe:
            tcpSocket:
              port: 8079
            initialDelaySeconds: 5
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /
              port: 8079
            initialDelaySeconds: 5
            periodSeconds: 3
```

`Expected output:`

```
kubectl apply -f front-end/frontend-deploy.yml
kubectl get pods
kubectl describe pod front-end-c5bc89b57-g42nc

[...]
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              11s   default-scheduler  Successfully assigned front-end-c5bc89b57-g42nc to node4
  Normal  SuccessfulMountVolume  10s   kubelet, node4     MountVolume.SetUp succeeded for volume "default-token-w4279"
  Normal  Pulling                8s    kubelet, node4     pulling image "schoolofdevops/frontend"
  Normal  Pulled                 6s    kubelet, node4     Successfully pulled image "schoolofdevops/frontend"
  Normal  Created                5s    kubelet, node4     Created container
  Normal  Started                5s    kubelet, node4     Started container
```

**Task**: Change the readinessProbe port to 8080 and check what happens to the pod.

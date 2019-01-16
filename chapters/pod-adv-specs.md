# Additional  Pod Specs  - Resources, Security Specs


## Resource requests and limits

We can control the amount of resource requested and used by all the pods. This can be done by adding following data the deployment template.

### Resource Request
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
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
```
This ensures that pod always get the minimum cpu and memory specified. But this does not restrict the pod from accessing additional resources if needed. Thats why we have to use **resource limit** to limit the resource usage by a pod.

`Expected output:`

```
kubectl describe pod front-end-5c64b7c5cc-cwgr5

[...]
Containers:
  front-end:
    Container ID:
    Image:          schoolofdevops/frontend
    Image ID:
    Port:           8079/TCP
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Requests:
      cpu:        250m
      memory:     128Mi
```

### Resource limit

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
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
```

`Expected output:`

```
kubectl describe pod front-end-5b877b4dff-5twdd

[...]
Containers:
  front-end:
    Container ID:   docker://d49a08c18fd9651af2f3dd28772da977b238a4010f14372e72e0ca24dcec8554
    Image:          schoolofdevops/frontend
    Image ID:       docker-pullable://schoolofdevops/frontend@sha256:94b7a0843f99223a8a1d284fdeeb3fd5a731c03aea57a52751c6ebde40be1f50
    Port:           8079/TCP
    State:          Running
      Started:      Thu, 08 Feb 2018 17:14:54 +0530
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
```

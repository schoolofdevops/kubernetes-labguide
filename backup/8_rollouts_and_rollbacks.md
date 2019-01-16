## Rolling updates with deployments

Update the version of the image in frontend-deploy.yml

`File: k8s-code/projects/mogambo/dev/frontend-deploy.yml`

```
...
    app: front-end
    spec:
      containers:
      - image: schoolofdevops/frontend:dogs

```

Apply Changes and monitor the rollout

```
kubectl apply -f frontend-deploy.yaml
kubectl rollout status deployment/front-end
```

## Rolling Back a Failed Update

Lets update the image to a tag which is non existent. We intentionally introduce this intentional error to fail fail the deployment.

`File: k8s-code/projects/mogambo/dev/frontend-deploy.yml`

```
...
    app: front-end
    spec:
      containers:
      - image: schoolofdevops/frontend:movi

```

Do a new rollout and monitor

```
kubectl apply -f frontend-deploy.yml
kubectl rollout status deployment/front-end
```

Also watch the pod status which might look like

```
front-end-645bb6fcd5-xsndp   1/1       Running            0          3m
front-end-b9bc5495f-8nbz5    0/1       ImagePullBackOff   0          54s
front-end-b9bc5495f-9x5gr    0/1       ImagePullBackOff   0          54s
front-end-b9bc5495f-pdx2l    0/1       ImagePullBackOff   0          55s
```

To get the revision history and details  
```
kubectl rollout history deployment/front-end
kubectl rollout history deployment/front-end --revision=x
[replace x with the latest revision]
```

[Sample Output]

```
root@kube-01:~# kubectl rollout history deployment/front-end
deployments "front-end"
REVISION	CHANGE-CAUSE
1		kubectl scale deployment/front-end --replicas=5
3		<none>
6		<none>
7		<none>

root@kube-01:~# kubectl rollout history deployment/front-end --revision=7
deployments "front-end" with revision #7
Pod Template:
  Labels:	app=front-end
	pod-template-hash=656710519
	role=ui
	tier=front
  Containers:
   front-end:
    Image:	schoolofdevops/frontend:movi
    Port:	8079/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

To undo rollout,

```
kubectl rollout undo deployment/front-end
```

or

```
kubectl rollout undo deployment/front-end --to-revision=1
kubectl get rs
kubectl describe deployment front-end
```

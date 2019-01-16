# LAB K106 - Defining Release Strategy with  Deployment

A Deployment is a higher level abstraction which sits on top of replica sets and allows you to manage the way applications are deployed, rolled back at a controlled rate.

Deployment has mainly two responsibilities,

  * Provide Fault Tolerance: Maintain the number of replicas for a type of service/app. Schedule/delete pods to meet the desired count.
  * Update Strategy: Define a release strategy and update the pods accordingly.

```
/k8s-code/projects/instavote/dev/
cp vote-rs.yaml vote-deploy.yaml
```


Deployment spec (deployment.spec) contains everything that replica set has + strategy. Lets add it as follows,

`file: vote-deploy.yaml`

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
```


This time, start monitoring with --show-labels options added.

```
watch -n 1 kubectl get  pod,deploy,rs,svc --show-labels
```


Lets  create the Deployment. Do monitor the labels of the pod while applying this.

```
kubectl apply -f vote-deploy.yaml
```

Observe the chances to pod labels, specifically the **pod-template-hash**.


Now that the deployment is created. To validate,

```
kubectl get deployment
kubectl get rs --show-labels
kubectl get deploy,pods,rs
kubectl rollout status deployment/vote
kubectl get pods --show-labels
```
Sample Output
```
kubectl get deployments
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
vote   3         3         3            1           3m
```


## Scaling a deployment  

To scale a deployment in Kubernetes:

```
kubectl scale deployment/vote --replicas=15

kubectl rollout status deployment/vote

```

Sample output:
```

Waiting for rollout to finish: 5 of 15 updated replicas are available...
Waiting for rollout to finish: 6 of 15 updated replicas are available...
deployment "vote" successfully rolled out
```

You could also update the deployment by editing it.

```
kubectl edit deploy/vote
```

[change replicas to 8 from the editor, save and observe]



## Rolling Updates in Action

Now, update the deployment spec to apply

file: vote-deploy.yaml
```

...
template:
  metadata:
    labels:
      version: v2   
  spec:
    containers:
      - name: app
        image: schoolofdevops/vote:v2

```

apply

```
kubectl apply -f vote-deploy.yaml

kubectl rollout status deployment/vote
```

Observe rollout status and monitoring screen.



```

kubectl rollout history deploy/vote

kubectl rollout history deploy/vote --revision=1

```

Try updating the version of the image from v2 to v3,v4,v5. Repeat a few times to observe how it rolls out a new version.  

## Undo and Rollback

file: vote-deploy.yaml
```
spec:
  containers:
    - name: app
      image: schoolofdevops/vote:rgjerdf

```

apply

```
kubectl apply -f vote-deploy.yaml

kubectl rollout status

kubectl rollout history deploy/vote

kubectl rollout history deploy/vote --revision=xx
```

where replace xxx with revisions

Find out the previous revision with sane configs.

To undo to a sane version (for example revision 3)

```
kubectl rollout undo deploy/vote --to-revision=2
```

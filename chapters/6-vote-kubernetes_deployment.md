# LAB K106 - Defining Release Strategy with  Deployment

A Deployment is a higher level abstraction which sits on top of replica sets and allows you to manage the way applications are deployed, rolled back at a controlled rate.

Deployment provides three features,

  * Availability: Maintain the number of replicas for a type of service/app. Schedule/delete pods to meet the desired count.
  * Scalability: Updating the replica count, allows you to scale in and out, and its the responsibility of the deployment to provide with that scalability. You could scale manually or use horizontalPodAutoscaler to do it automatically.  
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
  labels:
    role: vote 
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


In a additional terminal where you have kubectl client setup, launch the monitoring console by running the following command. Ensure you have --show-labels options set.
```
watch -n 1 kubectl get all --show-labels
```


Lets  create the Deployment. Do monitor the labels of the pod while applying this. Also clean up the previous replicaset to start with a clean state.

```
kubectl apply -f vote-deploy.yaml

kubectl delete rs vote
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


## Rolling out a new version

Now, update the deployment spec to use a new version of the image.

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

and trigger a rollout by applying it

```
kubectl apply -f vote-deploy.yaml
```

Open a couple of additional terminals on the host where kubectl is configured and launch the following monitoring commands.

Terminal 1
```
kubectl rollout status deployment/vote
```


Terminal 2
```
kubectl get all -l "role=vote"
```

**What to watch for ?**

  * rollout status using the command above
  * following fields for vote deployment on monitoring screen
    * READY count (will reflect surged replicas e.g. 14/12)
    * AVAILABLE count ( will never fall below REPLICAS - maxUnavilable)
    * UP-TO-DATE field will reflect replicas with latest version defined in deployment spec
  * observe that a new replicaset is created for every rollout.  
  * Continuously refresh the vote application in the browser to see new version if being rolled out without a downtime.  

Try updating the version of the image from v2 to v3,v4,v5. Repeat a few times to observe how it rolls out a new version.  

## Breaking a Rollout  

Introduce an error by using an image which does not exist.

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
```

Observe how  deployment handles the failure in the rolls out.

## Undo and Rollback

To observe  the rollout history use the following commands.

```

kubectl rollout history deploy/vote

kubectl rollout history deploy/vote --revision=xx
```

where replace xx with revisions

Find out the previous revision with sane configs.

To undo to a sane version (for example revision 2)

```
kubectl rollout undo deploy/vote --to-revision=2
```

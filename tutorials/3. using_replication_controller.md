## Maintaining the Fleet with Replication Controller

 - running pods independently will not ensure uptime
 - replication controller ensures specified number of replicas are always available
 - maintains the fleet
 - works similar to auto scaling groups concept that AWS EC2 has
 - shortened to **rc** or **rcs**

### Specs
  * spec.template
  * spec.selector
  * spec.replicas

### Problems with single pod


```
kubectl get pods
kubectl delete pod vote
kubectl get pods

```

#### Creating a replication controller

Lets create a replication controller for the vote app

filename vote_rc.yaml

Paste the below code in it:
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: vote
  namespace: dev
  labels:
    role: rc
    tier: front
spec:
  replicas: 2
  selector:
    app: vote
    role: ui
  template:
    metadata:
      labels:
        app: vote
        role: ui
        tier: front
    spec:
      containers:
      - image: schoolofdevops/vote
        imagePullPolicy: Always
        name: vote
        ports:
        - containerPort: 80
          protocol: TCP
```


Now to create a Replication controller:

```
kubectl apply -f vote_rc.yaml
```

To list replication controllers on a cluster

```
kubectl get rc
```
Sample Output:
```
NAME      DESIRED   CURRENT   READY     AGE
vote   2         2         2         17s
```
To view detailed information about a specific replication controller

```
kubectl describe rc vote
```

Sample Output:

```
Name:           vote
Namespace:      default
Image(s):       schoolofdevops/vote
Selector:       app=vote,version=v1.0
Labels:         app=vote
                version=v1.0
Replicas:       2 current / 2 desired
Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed
No volumes.
Events:
  FirstSeen     LastSeen        Count   From                            SubObjectPath   Type            Reason                  Message
  ---------     --------        -----   ----                            -------------   --------        ------                  -------
  1m            1m              1       {replication-controller }                       Normal          SuccessfulCreate        Created pod: vote-0b3h0
  1m            1m              1       {replication-controller }                       Normal          SuccessfulCreate        Created pod: vote-dd9s1
```

Scaling Pods

Update vote_rc.yaml to set replicas to 3

```
spec:
  replicas: 4
```

Apply changes

```
kubectl apply -f vote_rc.yaml

```

```
$ kubectl get pods

NAME            READY     STATUS    RESTARTS   AGE
vote-0b3h0   1/1       Running   0          2m
vote-4frx0   1/1       Running   0          9s
vote-b44j4   1/1       Running   0          9s
vote-dd9s1   1/1       Running   0          2m

```

Testing RC

```
kubectl get pods

NAME            READY     STATUS    RESTARTS   AGE
vote-0b3h0   1/1       Running   0          2m
vote-4frx0   1/1       Running   0          46s
vote-b44j4   1/1       Running   0          46s
vote-dd9s1   1/1       Running   0          2m
```

Knock off one of the pods,

e.g.

```
kubectl delete pod vote-dd9s1
```

Check if creates a new pod

```
kubectl get pods

NAME            READY     STATUS              RESTARTS   AGE
vote-0b3h0   1/1       Running             0          3m
vote-4frx0   1/1       Running             0          1m
vote-b44j4   1/1       Running             0          1m
vote-r4t2w   0/1       ContainerCreating   0          3s
```

# Redis Operator

Examine the code at [schoolofdevops/redis-operator: Kubernetes Operator to Setup Redis Cluster](https://github.com/schoolofdevops/redis-operator)

Clone the repo

```
git clone https://github.com/schoolofdevops/redis-operator.git
cd redis-operator
```


Apply the opertor code

```
kubectl apply -f redis-crd.yaml -f redis_operator-rbac.yaml -f redis_operator-deploy.yaml


kubectl get all
```

now create an instance of redis cluster

File : `my-redis.yaml`

```
apiVersion: database.example.com/v1
kind: Redis
metadata:
  name: my-redis
  namespace: default
spec:
  slaveSize: 2  # Specifies the number of slave nodes
  image: "redis:6.2.6"  # Redis Docker image version
  storageClassName: "standard"  # Specify the Kubernetes storage class for PVCs
  volumeSize: "200Mi"  # Each Redis node will use a PVC of this size
  backupEnabled: true  # Enables the backup functionality
  backupSchedule: "0 */6 * * *"  # Backup schedule, every 6 hours
```

apply

```
kubectl apply -f my-redis.yaml
```

examine the objects created

```
kubectl get all

kubectl delete -f redis-crd.yaml -f redis_operator-rbac.yaml -f redis_operator-deploy.yaml
```


#### Cleaning Up
Once done, to clean up,


```
kubectl delete -f my-redis.yaml


```

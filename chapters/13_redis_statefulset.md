# Deploying Redis Cluster with StatefulSets

What will you learn  

  * Statefulsets  
  * initContainers


## Creating a headless service

We will use Redis as Statefulsets for our instavote application stack.
It is similar to Deployment, but Statefulsets requires a `Service Name`.

Lets being by cleaning up the existing redis installation

```
kubectl delete svc redis
kubectl delete deploy redis
```

So we will create a **headless** service for redis first. A headless service is the one which will be created with no ClusterIP of its own, but will return the actual IP address of its endpoints (e.g. pods), which we would create later in this case.

`file: dev/redis-sts/redis-svc.yml`

```
kind: Service
metadata:
  name: redis
  labels:
    app: redis
spec:
  type: ClusterIP
  ports:
  - name: redis
    port: 6379
    targetPort: redis
  clusterIP: None
  selector:
    app: redis
    role: master
```

Observe
  * clusterIP value is set to None
  * selector has been updated to send traffic only to the master

now apply this and validate

```
kubectl apply -f redis-svc.yml

kubectl get svc
kubectl describe  svc redis
```



## Adding Redis configurations with ConfigMap

Lets now add the redis configuration with configmap.

Redis ConfigMap has two keys
  * master.conf - to provide  Redis master configs
  * slave.conf - to provide  Redis slave configs

`file: redis-cm.yml`

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis
data:
  master.conf: |
    bind 0.0.0.0
    protected-mode yes
    port 6379
    tcp-backlog 511
    timeout 0
    tcp-keepalive 300
    daemonize no
    supervised no
    pidfile /var/run/redis_6379.pid
    loglevel notice
    logfile ""
  slave.conf: |
    slaveof redis-0.redis 6379
```

apply and validate

```
kubectl apply -f redis-svc.yml

kubectl get cm
kubectl describe cm redis
```

## Using initContainers to configure redis replication

We have to deploy redis master/slave set up from one statefulset cluster. This requires two different redis cofigurations , which needs to be described in one Pod template. This complexity can be resolved by using init containers. These init containers copy the appropriate redis configuration by analysing the hostname of the pod. If the Pod's (host)name has `0` as **Ordinal number**, then it is choosen as the master and master.conf is copied to /etc/ directory. Every other  pod will be configured as a replica with  slave.conf as configuration.


`file: redis-sts.yml`

```
[...]
      initContainers:
      - name: init-redis
        image: redis:4.0.9
        command:
        - bash
        - "-c"
        - |
          set -ex
          # Generate mysql server-id from pod ordinal index.
          [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          # Copy appropriate conf.d files from config-map to emptyDir.
          if [[ $ordinal -eq 0 ]]; then
            cp /mnt/config-map/master.conf /etc/redis.conf
          else
            cp /mnt/config-map/slave.conf /etc/redis.conf
          fi
        volumeMounts:
        - name: conf
          mountPath: /etc
          subPath: redis.conf
        - name: config-map
          mountPath: /mnt/config-map
```

### Deploying Redis Master Slaves with Statefulsets

These redis containers are started after initContainers are succefully run and exit. One thing to note here, these containers mount the same volume, `conf`, from the initContainers which has the proper Redis configuration.

`file: redis-sts.yaml`

```
[...]
      containers:
      - name: redis
        image: redis:4.0.9
        command: ["redis-server"]
        args: ["/etc/redis.conf"]
        env:
        - name: ALLOW_EMPTY_PASSWORD
          value: "yes"
        ports:
        - name: redis
          containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
        - name: conf
          mountPath: /etc/
          subPath: redis.conf
```

To apply

```
kubectl apply -f redis-sts.yml
```

Validate the MASTER-SLAVE configuration

```
kubectl exec redis-0 redis-cli ROLE

kubectl exec redis-1 redis-cli ROLE
```

redis-0 should have been  configured as master, redis-1 as slave.  You should also see that redis-1 is been configured as the slave of redis-0.redis as follows,

```
kubectl exec redis-1 redis-cli ROLE
slave
redis-0.redis
6379
connected
28
```
This validated the redis master slave configuration.

## Nano Project: Configuring persistent volume claim per instance of redis

Similar to databases, each redis instance needs its own data store, which should also be persistent. Current code that you have applied uses emptyDir as the volume type. This is a problem as emptyDir gets deleted when the pod is deleted, and is not persistent.    

Your task is to update the YAML file for the statefulset  with the following changes,

  * use volumeClaimTemplate instead of volumes for volume **redis-data**. This will ensure a persistentVolumeClaim per replica/pod. All other volumes remain unchanged.
  * Provide the storageClass as NFS, a provisioner for which is already been configured
  * Size of the volume could be 200Mi as its just a key value store used by the instavote app to store the votes.
  * accessModes should be ReadWriteOnce as the volumes for redis should not be shared between multiple instances.

Update the statefulSet, apply and validate that persistentVolumeClaim and persistentVolume are created for each instance/pod of redis application.

##### Reading List

* [Redis Replication](https://redis.io/topics/replication)
* [Run Replicated Statefulsets Applications](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/)
* [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)



**Search Keywords**

  * init containers
  * kubernetes statefulsets
  * redis replication

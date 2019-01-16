# Deploying Redis Cluster with StatefulSets

What will you learn  

  * Statefulsets  
  * initContainers


### Redis Service

We will use Redis as Statefulsets for our Vote application.
It is similar to Deployment, but Statefulsets requires a `Service Name`.
So we will create a `headless service` (service without endpoints) first.

`file: redis-svc.yml`

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

apply

```
kubectl apply -f redis-svc.yml
```

*Note: clusterIP's value is set to None*.

### Redis ConfigMap

Redis ConfigMap has two sections.
  * master.conf - for Redis master
  * slave.conf - for Redis slave

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

apply

```
kubectl apply -f redis-svc.yml
```

### Redis initContainers

We have to deploy redis master/slave set up from one statefulset cluster. This requires two different redis cofigurations , which needs to be described in one Pod template. This complexity can be resolved by using init containers. These init containers copy the appropriate redis configuration by analysing the hostname of the pod. If the Pod's (host)name has `0` as **Ordinal number**, then it is choosen as the master and master.conf is copied to /etc/ directory. Other Pods will get slave.conf as configuration.


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

### Redis Statefulsets

These redis containers are started after initContainers are succefully ran. One thing to note here, these containers mount the same volume, `conf`, from the initContainers which has the proper Redis configuration.

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


##### Reading List

* [Redis Replication](https://redis.io/topics/replication)
* [Run Replicated Statefulsets Applications](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/)
* [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)



**Search Keywords**

  * init containers
  * kubernetes statefulsets
  * redis replication

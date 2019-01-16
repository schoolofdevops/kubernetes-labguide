# Configmap

Configmap is one of the ways to provide configurations to your application.

### Injecting env variables with configmaps
Create our configmap for vote app

file:  projects/instavote/dev/vote-cm.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: vote
  namespace: instavote
data:
  OPTION_A: Visa
  OPTION_B: Mastercard
```

In the above given configmap, we define two environment variables,

  1. OPTION_A=EMACS
  2. OPTION_B=VI


Lets create the configmap object

```
kubectl get cm
kubectl apply -f vote-cm.yaml
kubectl get cm
kubectl describe cm vote


```
In order to use this configmap in the deployment, we need to reference it from the deployment file.

Check the deployment file for vote add for the following block.

file: `vote-deploy.yaml`

```
...
    spec:
      containers:
      - image: schoolofdevops/vote
        imagePullPolicy: Always
        name: vote
        envFrom:
          - configMapRef:
              name: vote
        ports:
        - containerPort: 80
          protocol: TCP
        restartPolicy: Always
```

So when you create your deployment, these configurations will be made available to your application. In this example, the values defined in the configmap (Visa and Mastercard) will override the default values(CATS and DOGS) present in your source code.

```
kubectl apply -f vote-deploy.yaml
```

Watch the monitoring screen for deployment in progress.

```
kubectl get deploy --show-labels
kubectl get rs --show-labels
kubectl  rollout status deploy/vote

```

![ConfigMap Dashboard.\label{fig:captioned_image}](images/Configmap.png)


## Providing environment Specific Configs

Copy the dev env to staging


```
cd k8s-code/projects/instavote
cp -r dev staging
```

Edit the configurations   for staging

```
cd staging

```
Edit *vote-cm.yaml*

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: vote
data:
  OPTION_A: Apple
  OPTION_B: Samsung
```

Edit  *vote-svc.yaml*

  * remove namespace if set
  * remove extIP
  * remove hard coded nodePort config if any

file: vote-svc.yaml
```
---
apiVersion: v1
kind: Service
metadata:
  name: vote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
  type: NodePort
```

Edit *vote-deploy.yaml*

  * remove namespace if set
  * change replicas to 2   

Lets launch it all in another namespace. We could use **default** namespace for this purpose.

```
kubectl config set-context $(kubectl config  current-context) --namespace=default
```


Verify the namespace has been switched by observing the monitoring screen.

Deploy new objects in this nmespace

```
kubectl apply -f vote-svc.yaml
kubectl apply -f vote-cm.yaml
kubectl apply -f vote-deploy.yaml

```

Now connect to the vote service by finding out the nodePort configs

```
kubectl get svc vote

```

**Troubleshooting**

  * Do you see the application when you browse to http://host:nodeport
  * If not, why? Find the root cause and fix it.

#### Clean up and Switch back the  namespace

Verify the environment specific options are in effect. Once verified, you could switch the namespace back to instavote.


```
kubectl delete deploy/vote svc/vote

kubectl config set-context $(kubectl config  current-context) --namespace=instavote

cd ../dev/

```

## Configuration file as ConfigMap

In the  topic above we have seen how to use configmap as environment variables. Now let us see how to use configmap as redis configuration file.

Syntax for consuming file as a configmap is as follows

```
  kubectl create configmap --from-file <CONF-FILE-PATH> <NAME-OF-CONFIGMAP>
```

We have redis configuration as a file named 
`projects/instavote/config/redis.conf`. We are going to convert this file into a configmap

```
cd k8s-code/projects/instavote/config/

kubectl create configmap --from-file  redis.conf redis
```

Now validate the configs  

```

kubectl get cm

kubectl describe cm redis
```


To use this confif file, update your redis-deploy.yaml file to use it by mounting it as a volume.


File: `dev/redis-deploy.yaml`

```
    spec:
      containers:
      - image: schoolofdevops/redis:latest
        imagePullPolicy: Always
        name: redis
        ports:
        - containerPort: 6379
          protocol: TCP
        volumeMounts:
          - name: redis
            subPath: redis.conf
            mountPath: /etc/redis.conf
      volumes:
      - name: redis
        configMap:
          name: redis
      restartPolicy: Always
```

And apply it

```
kubectl apply -f redis-deploy.yaml

kubectl apply -f redis-svc.yaml

kubectl get rs,deploy --show-labels
```

**Exercise:** Connect to redis pod and verify configs.

## Secrets

Secrets are for storing sensitive data like *passwords and keychains*. We will see how db deployment uses username and password in form of a secret.

You would define two fields for db,

  * username
  * password

To create secrets for db you need to generate  *base64* format as follows,

```
echo "admin" | base64
echo "password" | base64
```

where **admin** and **password** are the actual values that you would want to inject into the pod environment.

If you do not have a unix host, you can make use of online base64 utility to generate these strings.

```
http://www.utilities-online.info/base64
```

Lets now add it to the secrets file,

File: projects/instavote/dev/db-secrets.yaml

```
apiVersion: v1
kind: Secret
metadata:
  name: db
  namespace: instavote
type: Opaque
data:
  POSTGRES_USER: YWRtaW4=
  POSTGRES_PASSWD: cGFzc3dvcmQ=
```

```
kubectl apply -f db-secrets.yaml

kubectl get secrets

kubectl describe secret db
```

Secrets can be referred to inside a container spec with following syntax

```
env:
  - name: VAR
    valueFrom:
      secretKeyRef:
        name: db
        key: SECRET_VAR
```


To consume these secrets, update the deployment for db

 file: db-deploy.yaml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: db
  namespace: instavote
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: back
      app: postgres
  minReadySeconds: 10
  template:
    metadata:
      labels:
        app: postgres
        role: db
        tier: back
    spec:
      containers:
      - image: postgres:9.4
        imagePullPolicy: Always
        name: db
        ports:
        - containerPort: 5432
          protocol: TCP
        # Secret definition
        env:
          - name: POSTGRES_USER
            valueFrom:
              secretKeyRef:
                name: db
                key: POSTGRES_USER
          - name: POSTGRES_PASSWD
            valueFrom:
              secretKeyRef:
                name: db
                key: POSTGRES_PASSWD
      restartPolicy: Always
```

To apply this,

```
kubectl apply -f db-deploy.yaml

kubectl apply -f db-svc.yaml

kubectl get pods,rs,deploy

kubectl exec -it   db-xxxx sh

[replace db-xxxx with pod name for db ]

env | grep -i postgres 

[here you should see the env vars defined for POSTGRES_USER and POSTGRES_PASS]
```


### Note: Automatic Updation of deployments on ConfigMap  Updates

Currently, updating configMap does not ensure a new rollout of a deployment. What this means is even after updading configMaps, pods will not immediately reflect the changes.  

There is a feature request for this https://github.com/kubernetes/kubernetes/issues/22368

Currently, this can be done by using immutable configMaps.  

  * Create a configMaps and apply it with deployment.
  * To update, create a new configMaps and do not update the previous one. Treat it as immutable.
  * Update deployment spec to use the new version of the configMaps. This will ensure immediate update.

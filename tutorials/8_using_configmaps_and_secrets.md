Configmap is one of the ways to provide configurations to your application.

### Injecting env variables with configmaps
Create our configmap for vote app

file:  apps/voting/dev/vote-cm.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: vote
  namespace: dev
data:
  OPTION_A: EMACS
  OPTION_B: VI
```

In the above given configmap, we define two environment variables,
  1. OPTION_A=EMACS
  2. OPTION_B=VI

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

So when you create your deployment, these configurations will be made available to your application. In this example, the values defined in the configmap (EMACS and VI) will override the default values(CATS and DOGS) present in your source code.

![configmap](images/Configmap.png "ConfigMap")

### Configmap as a configuration file

In the  topic above we have seen how to use configmap as environment variables. Now let us see how to use configmap as redis configuration file.

Syntax for consuming file as a configmap is as follows

```
  kubectl create configmap --from-file <CONF-FILE-PATH> <NAME-OF-CONFIGMAP>
```

We have redis configuration as a file named `apps/voting/config/redis.conf`. We are going to convert this file into a configmap

```
kubectl create configmap --from-file apps/voting/config/redis.conf redis
```


Update your redis-deploy.yaml file to use this confimap.
File: `redis-deploy.yaml`

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

File: apps/voting/dev/db-secrets.yaml

```
apiVersion: v1
kind: Secret
metadata:
  name: db
  namespace: dev
type: Opaque
data:
  POSTGRES_USER: YWRtaW4=
  # base64 of admin
  POSTGRES_PASSWD: cGFzc3dvcmQ=
  # base64 of password
```


To consume these secrets, update the deployment as

file: db-deploy.yaml.

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: db
  namespace: dev
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

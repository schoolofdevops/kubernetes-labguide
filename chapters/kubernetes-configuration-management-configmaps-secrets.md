# Configurations Management with ConfigMaps and Secrets

Configmap is one of the ways to provide configurations to your application.

## Injecting env variables with configmaps
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



### Note: Automatic Updation of deployments on ConfigMap  Updates

Currently, updating configMap does not ensure a new rollout of a deployment. What this means is even after updading configMaps, pods will not immediately reflect the changes.  

There is a feature request for this https://github.com/kubernetes/kubernetes/issues/22368

Currently, this can be done by using immutable configMaps.  

  * Create a configMaps and apply it with deployment.
  * To update, create a new configMaps and do not update the previous one. Treat it as immutable.
  * Update deployment spec to use the new version of the configMaps. This will ensure immediate update.

## Mounting Files with ConfigMap Volume Type

If you want to make  files e.g. configuration files available inside the container, you could add it to the ConfigMap, reference it as a volume in a pod and then finally mount it inside a container.  Lets add a couple of config files for the vote app this way.

Begin modifying the ConfigMap for vote app with the content of the config files as:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: vote
  namespace: instavote
data:
  OPTION_A: Visa
  OPTION_B: Mastercard

  development_config.py: |
    class DevelopmentConfig:
      DEBUG = True
      TESTING = False
      SECRET_KEY = 'development-key'
      SQLALCHEMY_DATABASE_URI = 'sqlite:///development.db'
      SQLALCHEMY_TRACK_MODIFICATIONS = True
      SQLALCHEMY_ECHO = True
  production_config.py: |
    class ProductionConfig:
      DEBUG = False
      TESTING = False
      SECRET_KEY = 'production-secret-key'
      SQLALCHEMY_DATABASE_URI = 'postgresql://user:password@localhost/production'
      SQLALCHEMY_TRACK_MODIFICATIONS = False
      SQLALCHEMY_ECHO = False
```

where, `development_config.py` and `production_config.py` keys contain the entire configuration file content within this config map.

Apply and validate with,
```
kubectl apply -f vote-cm.yaml
kubectl describe cm vote  
```

Now, to mount it as a volume, modify `vote-deploy.yaml` and add the  `spec.volumes` and `spec.container.volumeMount` using the code reference below:


```
spec:
  containers:
  - image: schoolofdevops/vote:v1
    name: vote
...
...
    volumeMounts:
      - name: config
        mountPath: "/app/config"
        readOnly: true
  volumes:
    - name: config
      configMap:
        name: vote
        items:
          - key: "development_config.py"
            path: "development_config.py"
          - key: "production_config.py"
            path: "production_config.py"
        optional: true
```

now apply the changes as

```
kubectl apply -f vote-deploy.yaml
```

To validate exec into one of the pods and list the files

```

kubectl exec -it vote-xxxx-yyyy sh
cd /app/config
ls
cat development_config.py
```

[sample output]
```
/app/config # ls
development_config.py  production_config.py
/app/config # cat development_config.py
class DevelopmentConfig:
  DEBUG = True
  TESTING = False
  SECRET_KEY = 'development-key'
  SQLALCHEMY_DATABASE_URI = 'sqlite:///development.db'
  SQLALCHEMY_TRACK_MODIFICATIONS = True
  SQLALCHEMY_ECHO = True
```

This validates that the configuration files were created using ConfigMap and made available as a mount path inside the container.

## Enabling HTTP Authentication for Nginx Ingress with Secrets

In this part of the lab, you will

  * Create a secret with an existing file
  * Modify a ingress rule to use that secret via annotations
  * Validate if traefik, the ingress controller, automatically picks up the configurations and enables http auth s

Lets create  htpasswd spec as Secret   


```
  apt install -yq apache2-utils
  htpasswd -c auth devops
```


  Or use [Online htpasswd generator](http://www.htaccesstools.com/htpasswd-generator/) to generate a htpasswd spec. if you use the online generator, copy the contents to a file by name `auth` in the current directory.

  Then generate the secret as,

```
  kubectl create secret generic basic-auth --from-file auth

  kubectl get secret

  kubectl describe secret basic-auth
```

  And then add annotations to the ingress object so that it is read by the ingress controller to update configurations. Reference [Nginx Ingress Page](https://kubernetes.github.io/ingress-nginx/examples/auth/basic/) to learn more.

`file: vote-ing.yaml`

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
  namespace: instavote
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - Vote App'
spec:
  ingressClassName: nginx
  rules:
  - host: vote.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vote
            port:
              number: 80
  - host: result.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: result
            port:
              number: 80
```

where,

  *  *nginx.ingress.kubernetes.io/auth-type: "basic"* defines authentication type that needs to be added.
  *  *nginx.ingress.kubernetes.io/auth-secre: "basic-auth"* refers to the secret created earlier.

apply

```
kubectl apply -f vote-ing.yaml
kubectl get ing/vote -o yaml
```

Observe the annotations field. No sooner than you apply this spec, ingress controller reads the event and a basic http authentication is set with the secret you added.


```bash

                      +----------+
       +--update----> |   nginx  |
       |              |  configs |
       |              +----------+
  +----+----+--+            ^
  | ingress    |            :
  | controller |            :
  +----+-------+            :
       |              +-----+-------+
       +---watch----> | ingress     | <------- user
                      | annotations |
                      +-------------+
```



![Name Based Routing](../images/domain-name.png)

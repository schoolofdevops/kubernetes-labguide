# Building your own Helm Charts

If you have not installed helm yet, do so by using

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 |  bash 

helm

```
 
## Part I - Generating a Chart with One Service

Create a namespace and switch to it

```
kubectl create namespace dev
kubectl config set-context --current --namespace=dev
kubectl config get-contexts
```

Generate a helm chart scaffold and change into the directory created to create a copy of values.yaml

```
cd ~
helm create instavote
cd instavote/
cp values.yaml values.dev.yaml 
```


Edit `values.dev.yaml`  to  update replicaCount, image and tag as

```

replicaCount: 4 
image: 
  repository: schoolofdevops/vote 
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion. 
  tag: "v5" 
```

Update service type to Node Port
```
service:
  type: NodePort
  port: 80
  nodePort: 30300
```

Install this chart with helm to deploy the `vote` service as:

```
helm install instavote -n dev --values=values.dev.yaml  . --dry-run
helm install instavote -n dev --values=values.dev.yaml  .
```

Validate with

```
helm list -A
kubectl get all
```

## Part II - Adding support for More Microservices


Create copy of the templates so that you could add support for one more service e.g. redis

```

cd templates/
mkdir vote redis
mv deployment.yaml  hpa.yaml  ingress.yaml  service.yaml vote/
cp vote/*.yaml redis/
```


You will start editing the templates so that template variables  such as  `Values.replicaCount` will be replaced with  `Values.vote.replicaCount`. Thats the theme you will have to follow from here on.

e.g. this is the exisinng code

```
{{- if not .Values.autoscaling.enabled }}
replicas: {{ .Values.replicaCount }}
{{- end }}
```

which should be changes to
```

spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.vote.replicaCount }}
  {{- end }}
```

To make it easier, you could use text replacement feature of sed



```
cd templates/vote/
sed -i 's/Values./Values.vote./g' deployment.yaml
sed -i 's/Values./Values.vote./g' service.yaml
```

and then

```
cd templates/redis/
sed -i 's/Values./Values.redis./g' deployment.yaml
sed -i 's/Values./Values.redis./g' service.yaml
```

Also change the name in the metadata field for the following files  

```
templates/vote/deployment.yaml
templates/vote/service.yaml
```

from

```
metadata:
  name: {{ include "instavote.fullname" . }}
```

to

```
metadata:
  name: vote
```

And similarly in the following files

```
templates/redis/deployment.yaml
templates/redis/service.yaml
```

from

```
metadata:
  name: {{ include "instavote.fullname" . }}
```

to

```
metadata:
  name: redis
```


Now update `instavote/values.yaml` to support multiple services 

  * add `vote` as top level key 
  * indent all properties so that those become sub properties of key added above i.e.  vote
  * copy over and create blocks for redis

Use this file as a reference [values.yaml](https://gist.github.com/initcron/51b28ca62b568955516b6ddddcd40f77)

Now go ahead and deploy by  

  * Creating your own copy of values.yaml
  * Providing values specific to those services

You could download this file with sample values [values.dev.yaml](https://gist.github.com/initcron/67e104f3a2949f20c2da06e19c854faa) using the following command :

```
wget -c https://gist.githubusercontent.com/initcron/67e104f3a2949f20c2da06e19c854faa/raw/90c7b0c330f133efae43ea71efb1beba314ea451/values.dev.yaml
```


And apply
```
helm uninstall instavote
helm install instavote -n dev --values=values.dev.yaml  .
helm list -A
kubectl get all
```



Validate that vote and redis are running with the values that you provided.

###  Exercise
Now that you have deployed vote and redis, go ahead and add the code to deploy  worker, db and result as well.

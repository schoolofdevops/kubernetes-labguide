# Kubernetes Quick Dive

In this lab you are going to deploy the **instavote** application stack [as described here](https://github.com/schoolofdevops/example-voting-app) in a kubernetes environment using **kubectl** commands. Later, you would learn how to do the same by writing declarive *yaml* syntax.  

Purpose of this lab is to quickly get your app up and running and demonstrate kubernetes key features such as scheduling, high availability, scalability, load balancing, service discovery etc.



### Deploying app with kubernetes

Before launching the app, create a new namespace and switch to it

```
kubectl get ns
kubectl create namespace instavote
kubectl get ns
kubectl config get-contexts
kubectl config set-context --current --namespace=instavote
kubectl config get-contexts
```

Launch vote application with kubernetes in the newly created namespace. 

```
kubectl create deployment vote --image=schoolofdevops/vote:v4
```

You could now validate that the instance of vote app is running by using the following commands,

```
kubectl get pods

kubectl get deploy

kubectl get all
```


### Scalability


Scale the vote app to run 4 instances.

```
kubectl scale deployment vote --replicas=4
kubectl get deployments,pods
```


### High Availability

```
kubectl get pods
```

The above command will list pods. Try to delete a few pods and observe how it affects the availability of your application.

```
kubectl delete pods vote-xxxx vote-yyyy
kubectl get deploy,rs,pods

```


### Load Balancing with Services

Publish the application (similar to using -P for port mapping)

```


kubectl create service nodeport vote --tcp=80:80 --node-port=30300

kubectl get service

```


Connect to the app,  refresh the page to see it load balancing.  Also try to vote and observe what happens.  


### Roll Out a New Version


```
kubectl scale deployment vote --replicas=12

kubectl set image deployment vote vote=schoolofdevops/vote:v5

```


watch the rolling update  in action

```
kubectl rollout status deploy/vote
```

### Exercise - Deploy Complete Instavote App

Deploy the services with the following spec to complete this application stack.

| Service Name  | Image     | Service Type     | Service Port   | Node Port   |
| :------------- | :------------- | :------------- | :------------- | :------------- |
|  redis      |   redis:alpine     | ClusterIP       | 6379     | N/A     |
|  worker      |   schoolofdevops/worker:latest     | No Service Needed       | N/A     | N/A     |
|  db      |   postgres:9.4     | ClusterIP       | 5432     | N/A |
|  result      |   schoolofdevops/vote-result     | NodePort       | 80     | 30400 |

If you see **db** deployment failing, fix it by adding the environment var as,

```
kubectl set env deployment db POSTGRES_HOST_AUTH_METHOD=trust
```

After deploying all services to validate,

  * Browse to vote and result services exposed outside to see the UI
  * When you submit a vote, it should be reflected on result
  * To submit multiple votes, use either a different browser, or use incognito window.  

#### Cleaning up

Once you are done observing, you could delete it with the following commands,

```

kubectl delete deploy vote redis worker db result

kubectl delete service vote redis db result
```

### Summary

When you deploy an application in kubernetes, you submit it to the api server/ cluster manager. Kubernetes automatically schedules it on a cluster, networks the pods, provides service discovery. In addition as you observed, your application is scalable, high available and is already running behind a  load balancer.

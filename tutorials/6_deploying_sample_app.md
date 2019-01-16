# Mini Project: Deploying Multi Tier Application Stack

In this project , you would write definitions for deploying the vote application stack with all components/tiers which include,

  * vote ui
  * redis
  * worker
  * db
  * results ui

## Tasks

  * Create deployments for all applications
  * Define services for each tier
  * Launch/appy the definitions


Following table depicts the state of readiness of the above services.

| App     | Deployment     | Service |
| :------------- | :------------- | :------------- |
| vote       | ready       | ready       |
| redis       | in progress       | ready       |
| worker       | in progress       | in progress       |
| db       | in progress       | todo       |
| results       | todo       | todo       |

### Deploying the sample application

To create deploy the sample applications,

```
kubectl create -f apps/voting/dev
```

Sample output is like:

```
deployment "db" created
service "db" created
deployment "redis" created
service "redis" created
deployment "vote" created
service "vote" created
deployment "worker" created
deployment "results" created
service "results" created
```



#### To Validatecheck it:

```
kubectl get svc -n dev
```

Sample Output is:
```
kubectl get service voting-app
NAME         CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
vote   10.97.104.243   <pending>     80:31808/TCP   1h
```
Here the port assigned is 31808, go to the browser and enter
```
masterip:31808
```
![alt text](images/Vote.png "Front-End")

This will load the page where you can vote.

To check the result:
```
kubectl get service result
```
Sample Output is:
```
kubectl get service result
NAME      CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
result    10.101.112.16   <pending>     80:32511/TCP   1h
```
Here the port assigned is 32511, go to the browser and enter
```
masterip:32511
```

![alt text](images/Result.png "Result Page")

This is the page where we can see the results of the vote.

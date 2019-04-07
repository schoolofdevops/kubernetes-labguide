# Mini Project: Deploying Multi Tier Application Stack

In this project , you would write definitions for deploying the vote application stack with all components/tiers which include,

  * vote ui
  * redis
  * worker
  * db
  * results ui

## Tasks

  * Create deployments for all applications
  * Define services for each tier applicable
  * Launch/apply the definitions


Following table depicts the state of readiness of the above services.

| App     | Deployment     | Service |
| :------------- | :------------- | :------------- |
| vote       | ready       | ready       |
| redis       | ready       | ready       |
| worker       | TODO       | n/a       |
| db       | ready       | ready       |
| results       | TODO       | TODO       |


**Specs:**

  * worker
    * image: schoolofdevops/worker:latest
  * results
    * image: schoolofdevops/vote-result
    * port: 80
    * service type: NodePort



#### To Validate:

```
kubectl get svc -n instavote
```
Sample Output is:
```
kubectl get service vote
NAME         CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
vote   10.97.104.243   <pending>     80:30000/TCP   1h
```
Here the port assigned is 31808, go to the browser and enter
```
masterip:30000
```

![Front-End.\label{fig:captioned_image}](images/vote-rc.png)

This will load the page where you can vote.

To check the result:
```
kubectl get service result
```
Sample Output is:
```
kubectl get service result
NAME      CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
result    10.101.112.16   <pending>     80:30100/TCP   1h
```
Here the port assigned is 32511, go to the browser and enter
```
masterip:30100
```
![Result Page.\label{fig:captioned_image}](images/Result.png)

This is the page where you should see the results for the vote application stack.

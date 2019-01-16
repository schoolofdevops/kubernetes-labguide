## Kubernetes Visualizer

In this chapter we will see how to set up kubernetes visualizer that will show us the changes in our cluster in real time.

### Set up

Fork the repository

```
yum install -y git #only if you use play-with-k8s

git clone  https://github.com/schoolofdevops/kube-ops-view

cd  kube-ops-view
```

Deploy the visualizer on kubernetes

```
kubectl apply -f deploy/

[ouput]
serviceaccount "kube-ops-view" created
clusterrole "kube-ops-view" created
clusterrolebinding "kube-ops-view" created
deployment "kube-ops-view" created
ingress "kube-ops-view" created
deployment "kube-ops-view-redis" created
service "kube-ops-view-redis" created
service "kube-ops-view" created
```

Get the nodeport for the service.

```
kubectl get svc

[output]
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kube-ops-view         NodePort    10.107.204.74   <none>        80:**30073**/TCP   1m
kube-ops-view-redis   ClusterIP   10.104.50.176   <none>        6379/TCP       1m
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP        8m
```

In my case, port **30073** is the nodeport.

Visit the port from the browser.

```
http://<NODE_IP:NODE_PORT>
```

![kube-visualizer](images/kube-visualizer.png)

### Test

Let us test how kubernetes cluster works.

Clone kubernetes code repository from School of Devops.

```
git clone https://github.com/schoolofdevops/k8s-code

cd k8s-code
```

Create the namespace.

```
cd projects/instavote/

kubectl apply -f instavote-ns.yaml

[output]
namespace "instavote" created
```

Deploy vote application on kubernetes

```
cd dev

kubectl apply  -f vote-deploy.yaml

[output]
deployment "vote" created
```

![kube-visualizer-deployment](images/kube-visualizer-deployment.png)

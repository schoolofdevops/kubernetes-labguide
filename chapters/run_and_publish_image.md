# Quck Deploy a Docker Image with Kubernetes  


```
kubectl run vote --image=schoolofdevops/vote --port 80

kubectl scale --replicas=10 deploy/vote

kubectl expose deploy vote --port 80 --target-port 80 --type NodePort


```

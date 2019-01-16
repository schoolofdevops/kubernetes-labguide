# Helm Package Manager
## Install Helm
To install helm you can follow following instructions. 

```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

Verify the installtion is successful,
```
helm --help
```

### Set RBAC for Tiller

`file: tiller-rbac.yaml`
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

Apply the ClusterRole and ClusterRoleBinding.
```
kubectl apply -f tiller-rbac.yaml

```

### Initialize
This is where we actually initialize Tiller in our Kubernetes cluster.
```
helm init --service-account tiller
```

[sample output]

```
Creating /root/.helm
Creating /root/.helm/repository
Creating /root/.helm/repository/cache
Creating /root/.helm/repository/local
Creating /root/.helm/plugins
Creating /root/.helm/starters
Creating /root/.helm/cache/archive
Creating /root/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```

### Install Wordpress with Helm
Search Helm repository for Wordpress chart
```
helm search wordpress
```

Fetch the chart to your local environment and change directory.
```
helm fetch --untar stable/wordpress
cd wordpress
```

Create a copy of default vaules file and edit it.
```
cp values.yaml my-values.yaml
vim my-values.yaml
```

Run it as a dry run to check for errors.
```
helm install --name blog --values my-values.yaml . --dry-run
```

Deploy the Wordpress stack.
```
helm install --name blog --values my-values.yaml .
```

### Install Prometheus with Helm
Official Prometheus Helm Chart repository.
```
https://github.com/helm/charts/tree/master/stable/prometheus
```

Official Grafana Helm Chart repository.
```
https://github.com/helm/charts/tree/master/stable/grafana
```
#### Grafana Deployment

Download Grafana charts to your local machine and change directory.
```
helm fetch --untar stable/grafana
cd grafana
```

Create a copy of default vaules file and edit it.
```
cp values.yaml myvalues.yaml
vim myvalues.yaml
```

Make sure your charts doesn't have any error.
```
helm install --name grafana --values myvalues.yaml --namespace instavote . --dry-run
```

Deploy Grafana on your K8s Cluster.
```
helm install --name grafana --values myvalues.yaml --namespace instavote .
```

#### Prometheus Deployment
Download Prometheus charts to your local machine and change directory.
```
helm fetch --untar stable/prometheus
cd prometheus
```

Create a copy of default vaules file and edit it.
```
cp values.yaml myvalues.yaml
vim myvalues.yaml
```

Make sure your charts doesn't have any error.
```
helm install --name prometheus --values myvalues.yaml --namespace instavote . --dry-run
```

Deploy Prometheus on your K8s Cluster.
```
helm install --name prometheus --values myvalues.yaml --namespace instavote .
```

### Install heapster with helm

```
helm install stable/heapster --namespace kube-system --name heapster --set image.tag=v1.5.1 --set rbac.create=true
```

[output]
```
NAME:   heapster
LAST DEPLOYED: Tue May 22 11:46:44 2018
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Role
NAME                         AGE
heapster-heapster-pod-nanny  2s

==> v1beta1/RoleBinding
NAME                         AGE
heapster-heapster-pod-nanny  2s

==> v1/Service
NAME      TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S)   AGE
heapster  ClusterIP  10.96.63.82  <none>       8082/TCP  2s

==> v1beta1/Deployment
NAME               DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
heapster-heapster  1        0        0           0          2s

==> v1/Pod(related)
NAME                                READY  STATUS   RESTARTS  AGE
heapster-heapster-696df57b44-zjf78  0/2    Pending  0         1s

==> v1/ServiceAccount
NAME               SECRETS  AGE
heapster-heapster  1        2s

==> v1beta1/ClusterRoleBinding
NAME               AGE
heapster-heapster  2s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace kube-system -l "app=heapster-heapster" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace kube-system port-forward $POD_NAME 8082
```

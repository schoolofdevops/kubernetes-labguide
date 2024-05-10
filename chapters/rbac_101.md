
Clone API Tester app and launch it

```
git clone https://github.com/schoolofdevops/k8s-api-tester.git
cd k8s-api-tester

kubectl apply -f api-tester-deploy.yaml
```

Now list the pod and check the logs

```

kubectl get pods

kubectl logs -f api-tester-xxxx
```

You shall see error messages with access to all api resources e.g. pods, deployments, services, pvs, events denied.

Lets look at how to configure the RBAC policies to get this app to work.

## Adding Roles and ClusterRoles

Add the following permissions for `api-tester` app

  * It should be able to list pods, deplyoments and services in namespace `default`


File `api-tester-sa.yaml`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-tester
  namespace: default
```

File : `api-tester-role.yaml`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: api-tester
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "Services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps", "extensions"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]
```

File : `api-tester-rolebinding.yaml`
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: api-tester
  namespace: default
subjects:
- kind: ServiceAccount
  name: api-tester
  namespace: default
roleRef:
  kind: Role
  name: api-tester
  apiGroup: rbac.authorization.k8s.io
```

apply

```
kubectl apply -f api-tester-sa.yaml -f api-tester-role.yaml -f api-tester-rolebinding.yaml
```

Now update the deployment spec to refer to Service Account as:

File : `api-tester-deploy.yaml`
```
...
..

spec:
  replicas: 1
  selector:
    matchLabels:
      app: api-tester
  template:
    metadata:
      labels:
        app: api-tester
    spec:
      serviceAccountName: api-tester
      containers:
      - name: api-tester
        image: docker.io/schoolofdevops/api-tester:latest

..
...
```


apply

```
kubectl apply -f api-tester-deploy.yaml
```

## Adding CLusterRoles and ClusterRoleBindings

Add the following permissions for `api-tester` app

  * It should be able to list `persistentvolumeclaims` in all namespaces
  * It should have ability to create and delete `persistentvolumes` in all namespaces
  * It should have ability to read and write `events` cluster wide


File: `api-tester-clusterrole.yaml`
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: api-tester-cluster-role
rules:
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

```

File: `api-tester-clusterrolebinding.yaml`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: api-tester-cluster-role-binding
subjects:
- kind: ServiceAccount
  name: api-tester
  namespace: default
roleRef:
  kind: ClusterRole
  name: api-tester-cluster-role
  apiGroup: rbac.authorization.k8s.io
```

apply

```
kubectl apply -f api-tester-clusterrole.yaml  -f api-tester-clusterrolebinding.yaml
```

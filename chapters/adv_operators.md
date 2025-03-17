# Operators 


##  Step 1: Install Operator SDK

Since you already have **Docker, KIND, Helm, and kubectl**, let's install **Operator SDK** with the following command 

```sh
curl -LO https://github.com/operator-framework/operator-sdk/releases/latest/download/operator-sdk_linux_amd64
chmod +x operator-sdk_linux_amd64
sudo mv operator-sdk_linux_amd64 /usr/local/bin/operator-sdk
```

Verify installation:

```sh
operator-sdk version
```


## Step 2: Create the Operator 


Create a new **Operator project**:

```
mkdir static-site-operator
cd static-site-operator
```

Initialize the project:

```sh
operator-sdk init --plugins=helm --domain=example.com
```

Examine the generated files:

```
sudo apt install tree
tree
```

### **📌 Create a Custom Resource Definition (CRD)**
We'll call our operator `StaticSiteOperator`:

```sh
operator-sdk create api --group web --version v1alpha1 --kind StaticSiteOperator
```

This generates:

A Custom Resource Definition (CRD) (staticsiteoperators.web.example.com).
A Helm chart in helm-charts/staticsiteoperator/.


## Step 3: Modify the Helm Chart for Nginx

Move into the Helm chart directory:

```
cd helm-charts/staticsiteoperator
```

Edit `values.yaml` to define Nginx parameters:

```
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: latest

service:
  type: NodePort
  port: 80
  nodePort: 30400 

volumeMounts:
  - name: static-content
    mountPath: /usr/share/nginx/html
    readOnly: true

volumes:
  - name: static-content
    configMap:
      name: static-site-content

```

Edit `templates/deployment.yaml` to include volume mounts:

Replace:

```
volumeMounts: []
```

With : 
```
volumeMounts:
  - name: static-content
    mountPath: /usr/share/nginx/html
    readOnly: true

```

And update:

```
volumes: []
```

with

```
volumes:
  - name: static-content
    configMap:
      name: static-site-content
```

Also add the following `checksum/config` annotation to the deployment:
```     
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
    
```

In `tempalte/service.yaml` add `nodePort` to the service:

```
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
      nodePort: {{ .Values.service.nodePort }}

```


---
## Step 4: Deploy the Operator

**📌 Install Cert Manager (needed for CRD validation)**

Run:

```sh
kubectl apply -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml
```

Wait for cert-manager pods to be ready:

```sh
kubectl get pods -n cert-manager
```

### **📌 Deploy the Operator**

Build the controller image and load it into KIND:
```
cd ~/static-site-operator
make docker-build
kind load docker-image controller:latest
```

Edit `config/manager/manager.yaml` and add `imagePullPolicy` as : 
```
      containers:
      - args:
          - --leader-elect
          - --leader-election-id=static-site-operator
          - --health-probe-bind-address=:8081
        image: controller:latest
        imagePullPolicy: IfNotPresent
```

Run:

```sh
sudo apt install make
make deploy
```

Check if it's running:

```sh
kubectl get pods -n static-site-operator-system
```

---
## **🛠 Step 5: Create a Custom Resource**
Now, let’s **define our Static Site Custom Resource (CR)**.

Create a file `config/samples/web_v1alpha1_staticsiteoperator.yaml`:

```yaml
apiVersion: web.example.com/v1alpha1
kind: StaticSiteOperator
metadata:
  name: my-static-site
spec:
  replicaCount: 1
```

Apply it:

```sh
kubectl apply -f config/samples/web_v1alpha1_staticsiteoperator.yaml
```

Check if the Operator is running:

```sh
kubectl get pods
```

---
## **🛠 Step 6: Create the ConfigMap**
This **ConfigMap** contains our **HTML content**:

File : `config/samples/static-site-configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: static-site-content
data:
  index.html: |
    <html>
    <head><title>My Static Site</title></head>
    <body>
    <h1>Welcome to My Kubernetes Static Site!</h1>
    <p>This page is dynamically loaded from a ConfigMap.</p>
    </body>
    </html>
```

Apply it:

```sh
kubectl apply -f config/samples/static-site-configmap.yaml
```


Find the Service and NodePort:

```sh
kubectl get svc my-static-site-staticsiteoperator
```

You should see something like:

```sh
NAME                                TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
my-static-site-staticsiteoperator   NodePort   10.96.92.215   <none>        80:30400/TCP   3m10s
```

Access the site by opening **http://localhost:30400** in your browser! 🎉

---
## **🛠 Step 8: Test Dynamic Updates**
To change the **HTML content**:

1. **Edit the ConfigMap**:
   ```sh
   kubectl edit configmap static-site-content
   ```
2. Modify the HTML and save.
3. Restart the pod:
   ```sh
   kubectl delete pod -l app.kubernetes.io/name=nginx
   ```

After a few seconds, refresh **http://localhost:30400** and see the update!

---
## **🛠 Step 9: Cleanup**
To remove everything:

```sh
make undeploy
kubectl delete all --all
```

---
## **🚀 Summary**
- Installed **Operator SDK**.
- Created a **Helm-based Operator**.
- Used **ConfigMap** for **dynamic updates**.
- Exposed the **Nginx site via NodePort**.
- Successfully updated content **without redeploying the Operator**.

---
## **🔗 What’s Next?**
- Add **Secret-based authentication** for protected pages.
- Use **Ingress** instead of NodePort for production use.
- Implement **autoscaling** based on CPU usage.

---

🔥 **Congrats! You've built your first Kubernetes Operator!** 🔥  
Let me know if you want more enhancements! 🚀

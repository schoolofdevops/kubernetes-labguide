#  Writing your own CRDs and Controllers 

In this lab, we aim to understand, create, and deploy Custom Resource Definitions (CRDs) and Controllers in Kubernetes. By the end of the labs, you will:

  * Understand CRDs & Controllers – Learn the role of CRDs in extending Kubernetes and how controllers automate actions based on these resources.  
  * Deploy an Existing CRD & Controller – See a real-world example by deploying an application that uses CRDs, such as ArgoCD.  
  * Build a Simple CRD & Controller – Define a custom Kubernetes resource and create a Bash-based controller to process it.  
  * Run the Controller Inside Kubernetes – Package the controller into a container and deploy it as a Kubernetes-native workload.
  * Monitor and Respond to Kubernetes Events – Develop a Kubernetes alerting system that detects common pod issues (Pending, ImagePullBackOff, CrashLoopBackOff) and logs alerts.


## Writing Kubealert CRD and Controller


Lets build  **KubeAlert**, a lightweight **Kubernetes-native alerting system** that detects and reports critical pod issues such as `Pending`, `ImagePullBackOff`, and `CrashLoopBackOff`. 

The **KubeAlert Controller**, deployed inside a Kubernetes cluster, continuously monitors all namespaces for problematic pod states and logs alerts to **stdout** for easy visibility via `kubectl logs`. This simple yet effective controller showcases how **Custom Resource Definitions (CRDs) and Controllers** can extend Kubernetes functionality, enabling proactive issue detection without relying on external monitoring tools.  

While this use case is simple, it demonstrates the power of Kubernetes extensibility and automation. Moreover, it can be further extended to integrate with external alerting tools such as Slack, PagerDuty, or OpsGenie, enhancing the alerting capabilities of the system.


Create a new directory for the project

```
mkdir kubealert
cd kubealert
```

Lets begin with creating a CRD definition for the `KubeAlert` resource.

File `kubealert-crd.yaml`

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: kubealerts.example.com
spec:
  group: example.com
  names:
    kind: KubeAlert
    listKind: KubeAlertList
    plural: kubealerts
    singular: kubealert
    shortNames:
      - ka
      - kal 
      - kalert
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              conditions:
                type: array
                items:
                  type: string
```


```
kubectl apply -f kubealert-crd.yaml
```

validate : 

```
kubectl get crds
kubectl  api-resources | grep -i kubealert
```


File : `kubealert-instance.yaml`

```
apiVersion: example.com/v1
kind: KubeAlert
metadata:
  name: alert-rule-1
spec:
  conditions:
    - Pending
    - ImagePullBackOff
    - CrashLoopBackOff
```

```
kubectl apply -f kubealert-instance.yaml

```

validate 

```
kubectl get kubealerts
kubectl describe kubealert alert-rule-1
```

Write the controller code 

File : `kubealert-controller.sh`

```
#!/bin/bash

echo "🚀 Starting Kubernetes Alert Controller..."
echo "🔍 Monitoring all namespaces for: Pending, ImagePullBackOff, CrashLoopBackOff"

while true; do
    # Fetch pods that are not in Running phase across all namespaces
    ALERT_PODS=$(kubectl get pods --all-namespaces --field-selector=status.phase!=Running -o wide | grep -E 'Pending|ImagePullBackOff|CrashLoopBackOff')

    # If ALERT_PODS is not empty, log to stdout
    if [[ ! -z "$ALERT_PODS" ]]; then
        echo "🚨 ALERT: Issue detected in Kubernetes Pods!"
        ALERT_MSG="🚨 KubeAlert: Pod Issue Detected (Cluster-wide)\n$ALERT_PODS"

        # Log to stdout (container logs)
        echo -e "$(date) - $ALERT_MSG"
    else
        echo "$(date) - No alerts detected."
    fi

    # Sleep before checking again
    sleep 10
done
```

Run the controller as 

```
chmod +x kubealert-controller.sh
nohup ./kubealert-controller.sh &
```


Step 5: Simulate and Test the Alerts
Now, let's create some test scenarios to see our controller in action.


Lets engineer failure. Create a Pod with a Non-Existent Image

```
kubectl run bad-image-test --image=doesnotexist:v1
```

Create a Pod That CrashLoops
```
kubectl run crash-test --image=busybox --command -- sh -c "exit 1"
```

Clean up 

```
pkill -f kubealert-controller.sh

```


Run the same conroller in Kubernetes 

```
kubectl apply -f https://raw.githubusercontent.com/advk8s/kubealert/refs/heads/main/k8s/kubealert-rbac.yaml

kubectl apply -f https://raw.githubusercontent.com/advk8s/kubealert/refs/heads/main/k8s/kubealert-deploy.yaml


```

validate 

```
kubectl  get pods -n kube-system
kubectl logs   -n kube-system -l "app=kubealert"
```


cleaning up 

```
kubectl delete -f https://raw.githubusercontent.com/advk8s/kubealert/refs/heads/main/k8s/kubealert-rbac.yaml

kubectl delete -f https://raw.githubusercontent.com/advk8s/kubealert/refs/heads/main/k8s/kubealert-deploy.yaml

kubectl delete pods bad-image-test crash-test

```

## Summary

Kubernetes allows developers to extend its capabilities using Custom Resource Definitions (CRDs) and Controllers, enabling automation and management of custom resources beyond the built-in objects. In this lab series, we will first deploy an existing CRD-based application (e.g., ArgoCD) to understand how Kubernetes extensions work. Next, we will create our own CRD and Controller, starting with a simple Bash-based implementation that monitors pod failures. Finally, we will deploy the controller inside Kubernetes to operate as a native workload. This hands-on approach will provide a strong foundation in Kubernetes extensibility, automation, and real-world operations.
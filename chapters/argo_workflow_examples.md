# Argo Workflow 

* Workflow Spec Structure  : https://argo-workflows.readthedocs.io/en/latest/walk-through/the-structure-of-workflow-specs/


### How to run the Workflows 

Launch Killerkoda Environment with Workflow Examples [argoproj](https://killercoda.com/argoproj/course/argo-workflows/workflow-examples)

To submit and watch the workflow use the following syntax, 

```
argo submit -n argo --watch xxx.yaml
```

where, `xxx.yaml` is your workflow template. 

to check the outcome/logs from the latest workflow, 
```
argo logs @latest
```

## Workflow Practice Labs 

```
cd exmaples 
```

Try these labs : 

* Simple Workflow Template with a Container : https://argo-workflows.readthedocs.io/en/latest/walk-through/hello-world/


```
argo submit -n argo --watch hello-world.yaml
argo logs @latest
```

* Container Workflow Template which takes input parameter : https://argo-workflows.readthedocs.io/en/latest/walk-through/parameters/

```
argo submit arguments-parameters.yaml -p message="Awesome Argo Workshop !" --watch
argo logs @latest

```

* Workflow with Steps : https://argo-workflows.readthedocs.io/en/latest/walk-through/steps/


```
argo submit --watch steps.yaml
argo logs @latest

```


* DAG Template : https://argo-workflows.readthedocs.io/en/latest/walk-through/dag/


```
argo submit --watch dag-diamond.yaml
argo submit --watch dag-diamond-steps.yaml
[watch the execution from workflow UI]
```

* Input/Output with Artifact Handling : https://argo-workflows.readthedocs.io/en/latest/walk-through/artifacts/


```
argo submit --watch artifact-passing.yaml 
[watch the execution from workflow UI]

```


* Script Workflow: https://argo-workflows.readthedocs.io/en/latest/walk-through/scripts-and-results/


```
argo submit --watch scripts-bash.yaml 
argo logs @latest

```


* Volumes: https://argo-workflows.readthedocs.io/en/latest/walk-through/volumes/


```
argo submit --watch volumes-pvc.yaml
argo logs @latest

```


* Suspend and Resume: https://argo-workflows.readthedocs.io/en/latest/walk-through/suspending/


```
argo submit --watch suspend-template.yaml
argo list
argo resume @latest
```


* Managing Kubernetes Resources via Workflows : https://argo-workflows.readthedocs.io/en/latest/walk-through/kubernetes-resources/

```
argo submit --watch k8s-jobs.yaml 
argo logs @latest 
argo delete @latest 
```



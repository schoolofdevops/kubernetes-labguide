# DaemonSets & CronJobs

In this lab you will explore about additional controllers such as DaemonSets, Jobs and CronJobs.

Create a new Namespace and switch to it

```
kubectl create namespace demo
kubectl config set-context --current --namespace=demo
kubectl config get-contexts
```

## Converting Vote Deployment to a DaemonSet


```
cp vote-deploy.yaml vote-ds.yaml
```

Edit `vote-ds.yaml`

  * Change Kind to `DaemonSet`
  * Remove `spec.replicas`
  * Remove `strategy`

Save file and apply as,

```
kubectl apply -f vote-ds.yaml
```

validate

```
kubectl get ds,pods -o wide
```

You shall notice that `vote-xxx-yyy` pods are running now via a DaemonSet which schedules one and only one instance of it on every node currently available for scheduling (Control Plane nodes are not schedulable by default).


Once done, you could delete it as

```
kubectl delete ds vote
```

## Creating a CronJob

Jobs and CronJobs are similar, the one difference between the two is Cron Jobs as the name suggest runs on a schedule, versus Jobs are run once and run to completion. What that means is once the jobs is completed, there is no need keep running it unlike the pods which are created by the deployment etc. which keep running all the time.


filename: `every-minute-job.yaml`

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: every-minute-job
spec:
  schedule: "* * * * *"  # This schedule runs at every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure

```

Here’s a breakdown of the key components in this CronJob spec:

  * apiVersion: batch/v1: Specifies the API version for the CronJob resource.
  * kind: CronJob: Defines the kind of Kubernetes resource, which in this case is a CronJob.
  * metadata: Metadata about the CronJob, such as its name.
  * spec.schedule: The schedule on which the job should be run. The * * * * * pattern specifies that the job should run every minute.
  * jobTemplate: The template for the job to be created. This specifies what the job will do when it runs.
  * containers: A list of containers to run within the pod created by the job. The container uses the busybox image and executes a command to print the current date and time along with a custom message.
  * restartPolicy: OnFailure: Specifies that the container should only be restarted if it fails.

apply this code as,

```
kubectl apply -f every-minute-job.yaml

```

validate and watch the job run every minute with

```
watch kubectl get pods,cronjobs
```

[Sample Output after 2/3 minutes]
```
Every 2.0s: kubectl get pods,cronjobs                                                                                                         ci-01: Sun May  5 11:46:02 2024

NAME                                  READY   STATUS      RESTARTS   AGE
pod/every-minute-job-28581825-mpxl6   0/1     Completed   0          62s
pod/every-minute-job-28581826-528v4   0/1     Completed   0          2s

NAME                             SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/every-minute-job   * * * * *   False     1        2s              92s

```

Observe how many instances of the pod are retained and figure out why.  Once done, clean up with

```
kubectl delete cronjob every-minute-job
```

## Clean up and Switch the Namespace back to Instavote


```
kubectl delete namespace demo
kubectl config set-context --current --namespace=instavote  
kubectl config get-contexts
```

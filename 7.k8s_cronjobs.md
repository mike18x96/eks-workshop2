# Cron Job
## Create Cron Job
A Cron Job is a job that runs on a given schedule, written in Cron format. There are two primary use cases:

Run jobs once at a specified point in time

Repeatedly at a specified point in time

Here is the job specification:

cronjob.yaml
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: hello-cronpod
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello World!
          restartPolicy: OnFailure
```

This job prints the current timestamp and the message “Hello World” every minute.

Create the Cron Job as shown in the command:
```
kubectl apply -f cronjob.yaml
```
Watch the status of the job as shown:

```
kubectl get -w cronjobs
```

In another terminal window, watch the status of pods created:

```
kubectl get -w pods -l app=hello-cronpod
```

Get logs from one of the pods:

```
kubectl logs hello-12312ERD3
```


## Delete Cron Job

```
kubectl delete -f cronjob.yaml
```

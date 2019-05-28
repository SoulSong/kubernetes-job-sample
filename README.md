[TOC]
# Introduce
This is a simple sample to show how the job and cron-job work in the kubernetes.

# Require
Create a namespace for the job test.
```bash
$ kubectl apply -f ./kubernetes/namespace.yaml
```

# JOB
Create the job with `kubectl apply` command:
```bash
$ kubectl apply -f ./kubernetes/job.yaml
```
Then get the status of the job, here we desired 5 pods and 0 pod had been successful.
```bash
$ kubectl get jobs -n job-ns
NAME       DESIRED   SUCCESSFUL   AGE
job-test   5         0            6s
```
Just tracking the execution of all pods. From the stdout as follows, 
it explains how `completions` and `parallelism` specs working on the job.
```bash
$ kubectl get pods -l job-name=job-test -n job-ns
NAME             READY     STATUS              RESTARTS   AGE
job-test-b2bmb   0/1       ContainerCreating   0          36s
job-test-gdcq9   0/1       ContainerCreating   0          15s
job-test-vczzt   0/1       Completed           0          36s

$ kubectl get pods -l job-name=job-test -n job-ns
NAME             READY     STATUS              RESTARTS   AGE
job-test-b2bmb   0/1       Completed           0          42s
job-test-gdcq9   0/1       ContainerCreating   0          21s
job-test-jbwhk   0/1       ContainerCreating   0          6s
job-test-vczzt   0/1       Completed           0          42s

$ kubectl get pods -l job-name=job-test -n job-ns
NAME             READY     STATUS              RESTARTS   AGE
job-test-b2bmb   0/1       Completed           0          1m
job-test-bjr5q   0/1       ContainerCreating   0          19s
job-test-gdcq9   0/1       Completed           0          49s
job-test-jbwhk   0/1       Completed           0          34s
job-test-vczzt   0/1       Completed           0          1m

$ kubectl get pods -l job-name=job-test -n job-ns
NAME             READY     STATUS      RESTARTS   AGE
job-test-b2bmb   0/1       Completed   0          1m
job-test-bjr5q   0/1       Completed   0          33s
job-test-gdcq9   0/1       Completed   0          1m
job-test-jbwhk   0/1       Completed   0          48s
job-test-vczzt   0/1       Completed   0          1m
```
View the contents of any pod, the output is in line with expectations.
```bash
$ kubectl logs job-test-vczzt -c test-container -n job-ns
2019-05-28 15:50:04 job-test-vczzt
```
Finally, we will find 5 pods have been successfully completed.
```bash
$ kubectl get jobs -n job-ns
NAME       DESIRED   SUCCESSFUL   AGE
job-test   5         5            1m
```

# Cron JOb
Create a cron-job with the schedule `*/1 * * * *`
```bash
$ kubectl apply -f ./kubernetes/cron-job.yaml 
```
Tracking the execution of all jobs, they will execute a new job every minute.
```bash
$ kubectl get jobs -n job-ns
NAME                       DESIRED   SUCCESSFUL   AGE
cron-job-test-1559063400   1         1            21s

$ kubectl get jobs -n job-ns
NAME                       DESIRED   SUCCESSFUL   AGE
cron-job-test-1559063400   1         1            1m
cron-job-test-1559063460   1         0            9s

$ kubectl get jobs -n job-ns
NAME                       DESIRED   SUCCESSFUL   AGE
cron-job-test-1559063400   1         1            2m
cron-job-test-1559063460   1         1            1m
cron-job-test-1559063520   1         1            58s

$ kubectl get jobs -n job-ns
NAME                       DESIRED   SUCCESSFUL   AGE
cron-job-test-1559063400   1         1            2m
cron-job-test-1559063460   1         1            1m
cron-job-test-1559063520   1         1            1m
cron-job-test-1559063580   1         0            0s
```

# Clean up
Clean up all resources:
```bash
$ kubectl delete -f ./kubernetes/job.yaml
$ kubectl delete -f ./kubernetes/cron-job.yaml 
$ kubectl delete -f ./kubernetes/namespace.yaml
```

# Reference
- [job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)
- [cronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
# III.	Application Observability And Maintenance

## Understand API Deprecations

**1. Create the CronJob using the file below and fix any issues that occur.**

<details><summary>File</summary>

<p>

busycronjob.yaml

```YAML
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: busycronjob
spec:
  jobTemplate:
    metadata:
      name: busycronjob
    spec:
      template:
        spec:
          containers:
          - image: busybox
            name: busycronjob
            command: ["sh", "-c", "echo I do my job...; sleep 10; echo Done!"]
            resources: {}
          restartPolicy: Never
  schedule: '*/1 * * * *'
```

</p>
</details>

<details><summary>Solution</summary>

<p>

```bash
kubectl apply -f busycronjob.yaml #should not work
kubectl explain cronjob
```

```YAML
apiVersion: batch/v1 #change 
kind: CronJob
metadata:
  name: busycronjob
spec:
  jobTemplate:
    metadata:
      name: busycronjob
    spec:
      template:
        spec:
          containers:
          - image: busybox
            name: busycronjob
            command: ["sh", "-c", "echo I do my job...; sleep 10; echo Done!"]
            resources: {}
          restartPolicy: Never
  schedule: '*/1 * * * *'
```

</p>
</details>

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

## Implement probes and health checks

**1. Create a pod using the wordpress image. Implement a readiness probe that executes the command 'cat readme.html'. The kubelet should wait 15 seconds before performing the first check, then run the check every 5 seconds. When did the pod's status change to READY?**

<details><summary>Solution</summary>

<p>

```bash
kubectl run wordpress --image=wordpress --dry-run=client -o yaml > wordpress.yaml
vim wordpress.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: wordpress
  name: wordpress
spec:
  containers:
  - image: wordpress
    name: wordpress
    resources: {}
    readinessProbe:              #add
      exec:                      #add
        command:                 #add
        - cat                    #add
        - readme.html            #add
      initialDelaySeconds: 15    #add
      periodSeconds: 5           #add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
The pod's status changes to READY when the readiness probe succeeds

</p>
</details>

<br/>

**2.	Create a pod using the equivalent/health_check_nginx image. Implement a liveness probe that calls the /health-check endpoint using the HTTP GET method on port 80. The kubelet should wait 10 seconds before performing the first check, and then execute the check every 5 seconds.**

<details><summary>Solution</summary>

<p>

```YAML
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-health
  name: nginx-health
spec:
  containers:
  - image: equivalent/health_check_nginx
    name: nginx-health
    livenessProbe:
      httpGet:
        path: /health-check
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

</p>
</details>
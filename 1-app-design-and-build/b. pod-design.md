
## Choose and use the right workload resource

* [Pods](https://kubernetes.io/docs/concepts/workloads/pods/ "Pods")
* [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/ "ReplicaSets")
* [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ "Deployments")
* [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/ "Jobs")
* [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/ "CronJobs")

### Pods

**1. Create a new Pod using the <code>busybox</code> image and display all Pods in the default namespace**

<details><summary>Solution</summary>
<p>

```bash
kubectl run busybox --image=busybox 
kubectl get pods #the namespace is 'default' unless specified otherwise
```

</p>
</details>


**2. Create a new Pod using the <code>busybox</code> image and set <code>restartPolicy</code> to <code>Never</code>.**

<details><summary>Solution</summary>
<p>

```bash
kubectl run busybox2 --image=busybox --restart=Never 
kubectl get pods
```

</p>
</details>


**3. Create an nginx Pod and save its logs to app.log**

<details><summary>Solution</summary>
<p>

```bash
kubectl run nginx --image=nginx
kubectl logs nginx > app.log
```

</p>
</details>


**4. Execute into the <code>nginx</code> Pod and list all environment variables**

<details><summary>Solution</summary>
<p>

```bash
kubectl exec -it nginx /bin/bash -- env
```

</p>
</details>


**5. Show the <code>nginx</code> Pod's IP**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -o wide #solution 1
kubectl describe pods nginx | grep -i IP #solution 2
```

</p>
</details>


**6. Force the <code>nginx</code> Pod to be deleted immediately**

<details><summary>Solution</summary>
<p>

```bash
kubectl delete pods nginx --force --grace-period=0
```

</p>
</details>


**7. Create a new <code>nginx</code> Pod. Then, create a <code>busybox</code> Pod that sends a request to the <code>nginx</code> Pod using <code>wget {podIp}:80</code>**

<details><summary>Solution</summary>
<p>

```bash
kubectl run nginx --image=nginx
kubectl get pods nginx -o wide #copy pod ip
kubectl run busybox --image=busybox --restart=Never -it --rm -- wget -O- {podIp}:80 #nginx uses by default port 80
```

</p>
</details>


**8. Create a YAML config file for an <code>nginx</code> Pod without applying it. Name the container <code>nginx-container</code>. Then, create the Pod**

<details><summary>Solution</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx-container # add 
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl apply -f nginx.yaml
kubectl describe pod nginx | grep -i nginx-container
```

</p>
</details>


### ReplicaSets

**1. Create a ReplicaSet with 3 replicas using the <code>wordpress</code> image**

<details><summary>Solution</summary>
<p>

ReplicaSets cannot be created imperatively, so you need to copy a template from the k8s docs and modify it

```YAML
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: wordpress-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress                                        
```

```bash
kubectl apply -f wordpress.yaml
kubectl get rs
kubectl get pods
```

</p>
</details>


**2. Create a ReplicaSet from the following file. Do you notice any errors? Can they be fixed?**

<details><summary>File</summary>
<p>

```YAML
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx                      
```

</p>
</details>


<details><summary>Solution</summary>
<p>

```YAML
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx #change
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx                      
```

</p>
</details>


### Deployments

**1. Create a namespace called <code>production</code>**

<details><summary>Solution</summary>
<p>

```bash
kubectl create ns production
```

</p>
</details>


**2. Set the namespace to <code>production</code> for the current Kubernetes context**

<details><summary>Solution</summary>
<p>

```bash
kubectl config set-context --current --namespace=production
```

</p>
</details>


**3. Create the YAML file for a Deployment named <code>nginx</code> that uses the <code>nginx:1.27</code> image with 2 replicas. Do not create the Deployment yet**

<details><summary>Solution</summary>
<p>

```bash
kubectl create deploy nginx --image=nginx:1.27 --replicas=2 --dry-run=client -o yaml > deploy.yaml
```

</p>
</details>


**4. Create the deployment in the <code>production</code> namespace, list all the Pods in that namespace**

<details><summary>Solution</summary>
<p>

Since you set the current namespace to production, there is no need to specify -n production at the end of the following commands

```bash
kubectl apply -f deploy.yaml
kubectl get pods 
```

</p>
</details>


**5. Scale the deployment to 4 replicas and display all the Pods in the namespace**

<details><summary>Solution</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=4
kubectl get pods 
```

</p>
</details>


**6. Set the image of the deployment to <code>nginx:2.224</code> and show the status of the deployment. Why does the deployment fail?**

<details><summary>Solution</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:2.224
kubectl rollout status deploy nginx
```
The deploy fails because the image does not exist

```bash
kubectl describe pod {podName}
```


</p>
</details>


**7. Show all revisions of the deployment and revert it to a working one**

<details><summary>Solution</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=1
kubectl rollout history deploy nginx --revision=2
kubectl rollout undo deploy nginx --to-revision=1
kubectl get deploy nginx -o wide #the image version should be nginx:1.27
kubectl get pods # all pods should be running
```
</p>
</details>


### Jobs

**1. Switch back to the <code>default</code> namespace**

<details><summary>Solution</summary>
<p>

```bash
kubectl config set-context --current --namespace=default
```
</p>
</details>


**2. Create a YAML file for a job that uses the <code>busybox</code> image and executes the following command: <code>["sh", "-c", "echo I do my job...; sleep 10; echo Done!"]</code>**

<details><summary>Solution</summary>
<p>

```bash
 kubectl create job busyjob --image=busybox --dry-run=client -o yaml > job.yaml
```

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: busyjob
spec:
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: busybox
        name: busyjob
        resources: {}
        command: ["sh", "-c", "echo I do my job...; sleep 10; echo Done!"] #add
      restartPolicy: Never
status: {}
```
</p>
</details>


**3. Create the Job and watch its status**

<details><summary>Solution</summary>
<p>

```bash
kubectl apply -f job.yaml
kubectl get jobs --watch
```
</p>
</details>


**4. Delete the job and recreate it, ensuring that if the job is not completed within 10 seconds, the Pods will terminate**

<details><summary>Solution</summary>
<p>

```bash
 kubectl delete job busyjob
```
```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: busyjob
spec:
  activeDeadlineSeconds: 10 #add
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: busybox
        name: busyjob
        resources: {}
        command: ["sh", "-c", "echo I do my job...; sleep 10; echo Done!"] #add
      restartPolicy: Never
status: {}
```
```bash
 kubectl apply -f job.yaml
 kubectl get jobs --watch
```
</p>
</details>


**5. Delete the previously created job, change the deadline to 1 minute, and recreate it with <code>completions</code> set to 4 and <code>parallelism</code> set to 2**

<details><summary>Solution</summary>
<p>

```bash
 kubectl delete job busyjob
```
```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: busyjob
spec:
  activeDeadlineSeconds: 60 #change
  parallelism: 2 #add
  completions: 4 #add
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: busybox
        name: busyjob
        resources: {}
        command: ["sh", "-c", "echo I do my job...; sleep 10; echo Done!"]
      restartPolicy: Never
status: {}
```
```bash
 kubectl apply -f job.yaml
 kubectl get jobs --watch
```
</p>
</details>


**6. Check the logs of one of the Pods**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods
kubectl log {podName}
```
</p>
</details>


### CronJobs

**1. Create a CronJob that uses the <code>busybox</code> image and executes the following command once every minute: <code>["sh", "-c", "echo One minute has passed!"]</code>**

<details><summary>Solution</summary>
<p>

```bash
kubectl create cronjob busycronjob --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml > cronjob.yaml
```

```YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: busycronjob
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: busycronjob
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - image: busybox
            name: busycronjob
            command: ["sh", "-c", "echo One minute has passed!"] #add
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
status: {}
```
```bash
kubectl apply -f cronjob.yaml
```

</p>
</details>


**2. If the Job misses its scheduled time by 60 seconds, ensure that the CronJob skips the current iteration**

<details><summary>Solution</summary>
<p>

```YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: busycronjob
spec:
  startingDeadlineSeconds: 60 #add
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: busycronjob
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - image: busybox
            name: busycronjob
            command: ["sh", "-c", "echo One minute has passed!"]
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
status: {}
```
</p>
</details>


**3. Create another CronJob that uses the <code>busybox</code> image and executes the following command every Friday: <code>["sh", "-c", "echo Today is Friday!"]</code>**

<details><summary>Solution</summary>
<p>

```bash
kubectl create cronjob fridaycronjob --image=busybox --schedule="* * * * 5" --dry-run=client -o yaml > fridaycronjob.yaml
```

```YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: fridaycronjob
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: fridaycronjob
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - image: busybox
            name: fridaycronjob
            resources: {}
            command: ["sh", "-c", "echo Today is Friday!"]
          restartPolicy: OnFailure
  schedule: '* * * * 5'
status: {}
```
```bash
kubectl apply -f fridaycronjob.yaml
```

</p>
</details>


**4. Create a Job from the CronJob created in step 3**
<details><summary>Solution</summary>
<p>

```bash
kubectl create job jobfromcronjob --from=cronjob/fridaycronjob
```

</p>
</details>


### Multi-container Pod design patterns

**1. Create a Pod with 2 containers: one init container that uses the <code>busybox</code> image and executes the following command: <code>["sh", "-c", "echo 'Some things need to be initialized before the main container starts';", "sleep 20"]</code>, the second container should be named <code>main-app</code> and use the image: <code>nginx</code>.**

<details><summary>Solution</summary>
<p>

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multipod
  name: multipod
spec:
  initContainers:
  - image: busybox
    name: busybox
    command: ["sh", "-c", "echo 'Some things have to be initialized before the main container starts';", "sleep 20"]
  containers:
  - image: nginx
    name: main-app
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl get pods
```
</p>
</details>


**2. Create a Pod with 2 containers, both using the <code>busybox</code> image and executing the command: <code>['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']</code>. Exec into the first container and list all environment variables**

<details><summary>Solution</summary>
<p>

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
  - image: busybox
    name: busybox2
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl exec -it busybox --container=busybox -- env 
```
</p>
</details>

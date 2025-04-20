## Understand Requests, Limits, Quotas

### Requests, Limits

**1.	Create a wordpress pod with resource requests set to 64Mi of memory and 0.2 CPU core, and limits set to 128Mi of memory and 0.5 CPU, applied at the container level**

<details><summary>Solution</summary>

<p>

```bash
kubectl run wordpress --image=wordpress --dry-run=client -o yaml > wordpress.yaml
```
wordpress.yaml

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
    resources:
      requests:               #add
        memory: "64Mi"        #add
        cpu: 0.2              #add
      limits:                 #add
        memory: "128Mi"       #add
        cpu: 0.5              #add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>

<br/>

**2.	Force the immediate deletion of the pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod wordpress --force --grace-period=0
```

</p>
</details>

<br/>

### LimitRanges

**1.	Create a LimitRange for Pods for CPU usage. Set the maximum to 0.5 CPU and the minimum to 100m CPU.**

<details><summary>Solution</summary>

<p>
limitrange.yaml

```YAML
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - max: 
      cpu: "0.5"
    min:
      cpu: 100m
    type: Pod
```
```bash
kubectl apply -f limitrange.yaml
kubectl get limitrange
```
</p>
</details>

<br/>

**2.	Create a wordpress pod with 64Mi of memory and 0.2 CPU core for requests, and limits of 128Mi of memory and 0.6 CPU. Was the pod creation successful?**

<details><summary>Solution</summary>

<p>

wordpress2.yaml

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
    resources:
      requests:               #add
        memory: "64Mi"        #add
        cpu: 0.2              #add
      limits:                 #add
        memory: "128Mi"       #add
        cpu: 0.6              #add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
```bash
kubectl apply -f wordpress2.yaml #should display Forbidden error
```
</p>
</details>

<br/>

**3.	Delete the LimitRange**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete limitrange cpu-resource-constraint
```

</p>
</details>

<br/>

### Quotas

**1. Create a namespace named 'test-quota'**

<details><summary>Solution</summary>

<p>

```bash
kubectl create ns test-quota
kubectl get ns
```

</p>
</details>

<br/>

**2.	Create a ResourceQuota that sets the following hard limits for the 'test-quota' namespace: cpu: 5, memory: 5Gi, and pods: 2**

<details><summary>Solution</summary>

<p>

```bash
kubectl create quota myquota --hard=cpu=5,memory=5Gi,pods=2 -n test-quota
kubectl get quota -n test-quota
```

</p>
</details>

<br/>

**3.	Create a deployment using the nginx image with 3 replicas in the 'test-quota' namespace. Set resource requests to 64Mi of memory and 0.2 CPU core, and limits to 128Mi of memory and 0.5 CPU, applied at the container level. Was the creation successful? Are all pods running?**

<details><summary>Solution</summary>

<p>

```bash
kubectl create deploy nginx --image=nginx --replicas=3 -n test-quota --dry-run=client -o yaml > nginx.yaml
```
nginx.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
  namespace: test-quota
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: 
          limits:              #add
            cpu: 0.5           #add
            memory: "128Mi"    #add
          requests:            #add
            cpu: 0.2           #add
            memory: "64Mi"     #add
status: {}
```
```bash
kubectl apply -f nginx.yaml 
kubectl get pods -n test-quota #only 2 pods should be running
```
</p>
</details>

<br/>

**4.	Adjust the ResourceQuota so that all 3 pods can run**

<details><summary>Solution</summary>

<p>

```bash
kubectl edit quota myquota -n test-quota
```

```YAML
...
spec:
  hard:
    cpu: "5"
    memory: 5Gi
    pods: "3" #change
...
```
```bash
kubectl rollout restart deploy nginx -n test-quota
kubectl get pods -n test-quota #should display 3 nginx pods
```

</p>
</details>

<br/>
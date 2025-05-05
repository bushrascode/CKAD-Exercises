# II. Application Deployment

## Deployment strategies

### Blue/Green Deployment

**1. Create a <code>blue</code> Deployment and a Service to expose it by applying the following files:**

<details><summary>Files</summary>
<p>

blue-deployment.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: blue
  name: blue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blue
  template:
    metadata:
      labels:
        app: blue
    spec:
      containers:
      - image: nginx:1.26
        name: nginx
```

</p>

node-port-service.yaml

<p>

```YAML
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: mysvc
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: blue    
```

</p>
</details>


<details><summary>Solution</summary>
<p>

```bash
kubectl apply -f blue-deployment.yaml
kubectl get pods #check if pods are running
kubectl apply -f node-port-service.yaml
kubectl get svc #check if service is created
```

</p>
</details>


**2. Find the IP of the node where the <code>nginx</code> Pod is running. Then, access the Pod using the NodePort service you created by running: <code>curl {nodeIP}:{nodePort}</code>.**

<details><summary>Solution</summary>
<p>

```bash
kubectl describe pod {bluePodName} | grep -i node 
kubectl get nodes -o wide 
curl {nodeIp}:30008
```

</p>
</details>


**3. Create a YAML file for a Deployment named <code>green</code> that uses the <code>nginx:1.27</code> image with 1 Replica. Then, create the Deployment.**

<details><summary>Solution</summary>
<p>

```bash
kubectl create deploy green --image=nginx:1.27 --replicas=1 --dry-run=client -o yaml > green-deployment.yaml
kubectl apply -f green-deployment.yaml
```

green-deployment.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: green
  name: green
spec:
  replicas: 1
  selector:
    matchLabels:
      app: green
  template:
    metadata:
      labels:
        app: green
    spec:
      containers:
      - image: nginx:1.27
        name: nginx
```

</p>
</details>


**4. Update the NodePort service to point to the <code>green</code> Deployment instead of the <code>blue</code> one.**

<details><summary>Solution</summary>

node-port-service.yaml

<p>

```YAML
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: mysvc
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: green #change    
```
```bash
kubectl apply -f node-port-service.yaml
curl {nodeIp}:30008
```
</p>
</details>


**5. Delete the <code>green</code> Deployment, then try sending a request to the <code>green</code> Pod using the NodePort service.**

<details><summary>Solution</summary>

```bash
kubectl delete deploy green
curl {nodeIp}:30008 #should not respond
```
</p>
</details>


**6. Update the NodePort service to point back to the <code>blue</code> Deployment.**
<details><summary>Solution</summary>

node-port-service.yaml

<p>

```YAML
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: mysvc
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: blue #change    
```
```bash
kubectl apply -f node-port-service.yaml
curl {nodeIp}:30008 #should respond
```
</p>
</details>


### Canary Deployment

**1. Create the stable <code>httpd</code> Deployment and a service to expose it by applying the following files:**

<details><summary>Files</summary>
<p>

stable-deployment.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: httpd
  name: httpd
spec:
  replicas: 4
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
        purpose: web-server
    spec:
      containers:
      - image: httpd:alpine
        name: httpd
```

</p>

svc.yaml

<p>

```YAML
apiVersion: v1
kind: Service
metadata:
  labels:
    app: httpd
  name: my-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    purpose: web-server
```

</p>
</details>


<details><summary>Solution</summary>
<p>

```bash
kubectl apply -f stable-deployment.yaml
kubectl get pods #check if pods are running
kubectl apply -f svc.yaml
kubectl get svc #check if service is created
```

</p>
</details>


**2.	Now you have to switch to a new version of the application that uses the <code>caddy:alpine</code> image. The transition should be gradual rather than immediate or automatic, using the Canary approach: <br>  &nbsp;&nbsp;&nbsp;&nbsp; a)	25% of requests should be directed to the new <code>caddy:alpine</code> image <br>  &nbsp;&nbsp;&nbsp;&nbsp; b)	75% of requests should continue to go to the old <code>httpd</code> image <br> &nbsp;&nbsp;&nbsp;&nbsp; c)	Ensure there are a total of 4 Pods running**

<details><summary>Solution</summary>
<p>

```bash
kubectl scale deploy httpd --replicas=3 #scale down the httpd deployment to 3 pods
kubectl create deploy caddy --image=caddy --replicas=1 --dry-run=client -o yaml > canary-deployment.yaml
```

Edit the canary deployment file:


```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: caddy
  name: caddy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: caddy
  template:
    metadata:
      labels:
        app: caddy
        purpose: web-server #set this label 
    spec:
      containers:
      - image: caddy
        name: caddy
```

```bash
kubectl apply -f canary-deployment.yaml
```

</p>
</p>
</details>


**3. Get the ClusterIP. Then, create a <code>busybox</code> Pod that hits the ClusterIP. Execute the request multiple times. The responses should come from different containers.**

<details><summary>Solution</summary>
<p>

```bash
kubectl get svc
kubectl run busybox --image=busybox --restart=Never -it --rm -- wget -O- {clusterIp}:80 
```

</p>
</details>

# II. Application Deployment

## Deployment strategies

### Blue/Green Deployment

**1. Create the 'blue' deployment and a service to expose it by applying the following files:**

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
<br>

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
<br>

**2. Find the IP of the node where the nginx pod is running. Then, access the pod using the NodePort service you created by running: curl {nodeIP}:{nodePort}.**

<details><summary>Solution</summary>
<p>

```bash
kubectl describe pod {bluePodName} | grep -i node 
kubectl get nodes -o wide 
curl {nodeIp}:30008
```

</p>
</details>
<br>

**3. Create a YAML file for a deployment named 'green' that uses the nginx:1.27 image with 1 replica. Then, create the deployment.**

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
<br>

**4. Update the NodePort service to point to the 'green' deployment instead of the 'blue' one**

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
<br>

**5. Delete the 'green' deployment, then try sending a request to the 'green' pod using the NodePort service**

<details><summary>Solution</summary>

```bash
kubectl delete deploy green
curl {nodeIp}:30008 #should not respond
```
</p>
</details>
<br>

**6. Update the NodePort service to point back to the 'blue' deployment**
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
<br>

### Canary Deployment

**1. Create the stable 'httpd' deployment and a service to expose it by applying the following files:**

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
<br>

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
<br>

**2.	Now you have to switch to a new version of the application that uses the caddy:alpine image. The transition should be gradual rather than immediate or automatic, using the Canary approach: <br>  &nbsp;&nbsp;&nbsp;&nbsp; a)	25% of requests should be directed to the new caddy:alpine image <br>  &nbsp;&nbsp;&nbsp;&nbsp; b)	75% of requests should continue to go to the old httpd image <br> &nbsp;&nbsp;&nbsp;&nbsp; c)	Ensure there are a total of 4 pods running**

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
<br>

**3. Get the ClusterIP. Then, create a busybox pod that hits the ClusterIP. Execute the request multiple times. The responses should come from different containers**

<details><summary>Solution</summary>
<p>

```bash
kubectl get svc
kubectl run busybox --image=busybox --restart=Never -it --rm -- wget -O- {clusterIp}:80 
```

</p>
</details>
<br>

## Helm

**1. Add the Bitnami charts repository to your Helm configuration using the URL: https://charts.bitnami.com/bitnami**

<details><summary>Solution</summary>
<p>

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

</p>
</details>
<br>

**2. List all the charts in the repo**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami
```

</p>
</details>
<br>

**3.	Search for the Jenkins chart in the repo**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami/jenkins
```

</p>
</details>
<br>

**4. Show all values of the Jenkins chart**

<details><summary>Solution</summary>
<p>

```bash
helm show values bitnami/jenkins
```

</p>
</details>
<br>

**5.	Show all versions of the Jenkins chart**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami/jenkins --versions
```

</p>
</details>
<br>

**6.	Install the Jenkins chart with chart version 12.0.0 and 4 replicas. Then, list all pods.**

<details><summary>Solution</summary>
<p>

```bash
helm install jenkins bitnami/jenkins --version 12.0.0 --set replicaCount=4
kubectl get pods
```

</p>
</details>
<br>

**7.	List all releases, including broken ones**

<details><summary>Solution</summary>
<p>

```bash
helm ls -a
```

</p>
</details>
<br>

**8.	Upgrade the Jenkins release to version 13.0.0**

<details><summary>Solution</summary>
<p>

```bash
helm upgrade jenkins bitnami/jenkins --version 13.0.0
```

</p>
</details>
<br>

**9.	Uninstall the Jenkins release**

<details><summary>Solution</summary>
<p>

```bash
helm uninstall jenkins
```

</p>
</details>
<br>


**10.	Pull the latest Redis chart to your home folder and unzip it**

<details><summary>Solution</summary>
<p>

```bash
helm repo update
cd ~
helm pull bitnami/redis --untar
```

</p>
</details>
<br>

**11.	Show the environment variables that Helm uses for configuration**

<details><summary>Solution</summary>
<p>

```bash
helm env
```

</p>
</details>
<br>
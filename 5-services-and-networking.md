# V.	Services and Networking

## Network Policies

**1. Create an nginx pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl run nginx --image=nginx
```
</p>
</details>
<br/>

**2. Create a NetworkPolicy named 'nginx-netpol' that targets pods labeled with 'run: nginx'. This policy allows incoming traffic (ingress) on port 80 using the TCP protocol, but only from pods labeled with 'access: allowed'**

<details><summary>Solution</summary>

<p>

netpol.yaml

```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx-netpol
spec:
  podSelector:
    matchLabels:
      run: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: allowed
    ports:
    - protocol: TCP
      port: 80
  policyTypes:
  - Ingress
```
```bash
kubectl apply -f netpol.yaml
kubectl get netpol
```
</p>
</details>
<br/>

**3. Get the IP address of the nginx pod. Then, create a busybox pod that never restarts and runs the command 'sleep 3600'. Exec into the busybox pod and send a request to the nginx pod. Is there a response to the request?**

<details><summary>Solution</summary>

<p>

```bash
kubectl get pods -o wide
kubectl run busybox --image=busybox --restart=Never --command -- sleep 3600
kubectl exec -it busybox -- sh
wget {nginxPodIp}:80 #should not respond

```
</p>
</details>
<br/>

**4. Label the busybox pod with access=allowed, then try again to send a request to the nginx pod from the busybox pod. Is there a response to the request?**

<details><summary>Solution</summary>

<p>

```bash
kubectl label pod busybox access=allowed
kubectl exec -it busybox -- sh
wget {nginxPodIp}:80 #should respond
```
</p>
</details>
<br/>

## Services

**1. Create an nginx pod named 'nginx' in the default namespace and set containerPort=80. Expose it using a ClusterIP service named 'nginx-clusterip' on port 80. Then, list all services and endpoints in the default namespace. Next, create a busybox pod that never restarts, exec into it, and send a request to the ClusterIP address of the nginx pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl run nginx --image=nginx --port=80
kubectl expose pod nginx --name=nginx-clusterip --port=80
kubectl get svc
kubectl get endpoints
kubectl run busybox --image=busybox --restart=Never -it -- sh
wget -O- {clusterIp} #should respond
```
</p>
</details>
<br/>

**2. Create a pod using the 'httpd:alpine' image and expose it via a NodePort service named 'httpd-nodeport' on node port 30001, setting both the port and targetPort to 80. Then, get the node the pod is assigned to, display its IP address, and access the pod using {NodeIP}:30001**

<details><summary>Solution</summary>

<p>

```bash
kubectl run httpd --image=httpd:alpine
kubectl expose pod httpd --name=httpd-nodeport --port=80 --target-port=80 --type=NodePort --dry-run=client -o yaml > httpd-svc.yaml
```
httpd-svc.yaml

```YAML
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: httpd
  name: httpd-nodeport
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30001   #add
  selector:
    run: httpd
  type: NodePort
status:
  loadBalancer: {}
```
```bash
kubectl apply -f httpd-svc.yaml
kubectl get svc
kubectl describe pod httpd | grep -i node #get node IP
curl {nodeIp}:30001 #should respond
```
</p>
</details>
<br/>

**3. Create a Deployment using the nginx image with 2 replicas, and expose it via a NodePort service named 'nginx-nodeport' on port 20001, setting both the port and targetPort to 80. Were you able to successfully create the NodePort service? If any issues occurred, correct them accordingly.**

<details><summary>Solution</summary>

<p>

```bash
kubectl create deploy nginx --image=nginx --replicas=2
kubectl expose deploy nginx --name=nginx-nodeport --port=80 --target-port=80 --type=NodePort --dry-run=client -o yaml > nginx-svc.yaml
```
nginx-svc.yaml

```YAML
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx-nodeport
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 20001     #add
  selector:
    app: nginx
  type: NodePort
status:
  loadBalancer: {}
```
```bash
kubectl apply -f httpd-svc.yaml #should display error; resolve it by setting the nodePort to a number within the valid range
```
</p>
</details>
<br/>
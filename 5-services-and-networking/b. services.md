## Services

* [Services](https://kubernetes.io/docs/concepts/services-networking/service/ "Services")

**1. Create an nginx Pod named 'nginx' in the default namespace and set containerPort=80. Expose it using a ClusterIP service named 'nginx-clusterip' on port 80. Then, list all services and endpoints in the default namespace. Next, create a busybox Pod that never restarts, exec into it, and send a request to the ClusterIP address of the nginx Pod**

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


**2. Create a Pod using the 'httpd:alpine' image and expose it via a NodePort service named 'httpd-nodeport' on node port 30001, setting both the port and targetPort to 80. Then, get the node the Pod is assigned to, display its IP address, and access the Pod using {NodeIP}:30001**

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

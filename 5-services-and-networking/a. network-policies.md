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
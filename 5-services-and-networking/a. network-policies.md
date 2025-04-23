## Network Policies

* [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/ "Network Policies")

**1. Create an nginx Pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl run nginx --image=nginx
```
</p>
</details>


**2. Create a NetworkPolicy named <code>nginx-netpol</code> that targets Pods labeled with <code>run: nginx</code>. This policy allows incoming traffic (ingress) on port 80 using the TCP protocol, but only from Pods labeled with <code>access: allowed</code>.**

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


**3. Get the IP address of the <code>nginx</code> Pod. Then, create a <code>busybox</code> Pod that never restarts and runs the command <code>sleep 3600</code>. Exec into the <code>busybox</code> Pod and send a request to the <code>nginx</code> Pod. Is there a response to the request?**

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


**4. Label the <code>busybox</code> Pod with <code>access=allowed</code>, then try again to send a request to the <code>nginx</code> Pod from the <code>busybox</code> Pod. Is there a response to the request?**

<details><summary>Solution</summary>

<p>

```bash
kubectl label pod busybox access=allowed
kubectl exec -it busybox -- sh
wget {nginxPodIp}:80 #should respond
```
</p>
</details>

## Understand Application Security

* [Security Contexts](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ "Security Contexts")

**1.	Create a busybox Pod named <code>busybox</code> that never restarts and runs as a non-root user. Set <code>runAsUser: 1000</code>, <code>runAsGroup: 3000</code>, and <code>fsGroup: 2000</code>. Set <code>allowPrivilegeEscalation: false</code> at the container level. The container should execute the command: <code>sleep 3600</code>**

<details><summary>Solution</summary>

<p>

```bash
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml > busybox.yaml
```
busybox.yaml

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  securityContext:                              #add
    runAsUser: 1000                             #add
    runAsGroup: 3000                            #add
    fsGroup: 2000                               #add
  containers:
  - image: busybox
    name: busybox
    command: ["sh", "-c", "sleep 3600"]         #add
    securityContext:                            #add
      allowPrivilegeEscalation: false           #add
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
kubectl apply -f busybox.yaml
kubectl get pods
```
</p>
</details>



**2.	Create another <code>busybox</code> Pod named <code>busybox2</code>. The container should execute the command <code>sleep 3600</code> and be configured with the Linux capabilities <code>NET_ADMIN</code> and <code>SYS_TIME</code>**

<details><summary>Solution</summary>

<p>

```bash
kubectl run busybox2 --image=busybox --restart=Never --dry-run=client -o yaml > busybox2.yaml
```
busybox2.yaml

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox2
  name: busybox2
spec:
  containers:
  - image: busybox
    name: busybox2
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
kubectl apply -f busybox2.yaml
kubectl get pods
```
</p>
</details>




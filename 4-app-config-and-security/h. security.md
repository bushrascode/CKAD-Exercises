## Understand Application Security

* [Security Contexts](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ "Security Contexts")

**1.	Create a busybox Pod named 'busybox' that never restarts and runs as a non-root user. Set 'runAsUser: 1000', 'runAsGroup: 3000', and 'fsGroup: 2000'. Set 'allowPrivilegeEscalation: false' at the container level. The container should execute the command: 'sleep 3600'**

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



**2.	Create another busybox Pod named 'busybox2'. The container should execute the command 'sleep 3600' and be configured with the Linux capabilities "NET_ADMIN" and "SYS_TIME"**

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




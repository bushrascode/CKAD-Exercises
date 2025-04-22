## Understand ServiceAccounts

* [Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/ "Service Accounts")

**1.	Create a ServiceAccount named 'my-sa'**

<details><summary>Solution</summary>

<p>

```bash
kubectl create sa my-sa
```
</p>
</details>



**2.	Create an nginx Pod and assign the 'my-sa' ServiceAccount to it.**

<details><summary>Solution</summary>

<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
```

nginx.yaml

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: my-sa    #add
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
```bash
kubectl apply -f nginx.yaml
kubectl describe pod nginx | grep -i "Service Account" #should display 'my-sa'
```
</p>
</details>



**3.	Exec into the nginx Pod and navigate to the folder that contains the token associated with the 'my-sa' ServiceAccount. Display the contents of the token file.**

<details><summary>Solution</summary>

<p>

```bash
k exec -it nginx -- sh
cd /var/run/secrets/kubernetes.io/serviceaccount
cat token
```
</p>
</details>




**4.	Create another ServiceAccount named 'pod-lister-sa'**

<details><summary>Solution</summary>

<p>

```bash
kubectl create sa pod-lister-sa
kubectl get sa
```
</p>
</details>



**5.	Create a Role named 'pod-lister' that grants permission to list Pods**

<details><summary>Solution</summary>

<p>

role.yaml

```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-lister
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```
```bash
kubectl apply -f role.yaml
kubectl get roles
```

</p>
</details>



**6.	Create a RoleBinding named 'sa-role-binding' that binds the 'pod-lister' Role to the ServiceAccount 'pod-lister-sa**

<details><summary>Solution</summary>

<p>

rolebinding.yaml

```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-role-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: pod-lister-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-lister
  apiGroup: rbac.authorization.k8s.io

```
```bash
kubectl apply -f rolebinding.yaml
kubectl get rolebindings
```
</p>
</details>



**7.	Create a new nginx Pod named 'nginx2' and assign the 'pod-lister-sa' ServiceAccount to it**

<details><summary>Solution</summary>

<p>
nginx2.yaml

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx2
  name: nginx2
spec:
  serviceAccountName: pod-lister-sa    #add
  containers:
  - image: nginx
    name: nginx2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
```bash
kubectl apply -f nginx2.yaml
kubectl describe pod nginx | grep -i "Service Account" #should display 'pod-lister-sa'
```
</p>
</details>


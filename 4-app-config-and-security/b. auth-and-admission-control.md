## Understand authentication, authorization and admission control

* [Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/ "Admission Controllers")
* [RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/ "RBAC Authorization")

### Admission controllers

**1. Create a Pod using the <code>nginx</code> image. Which ServiceAccount is associated with the Pod?**

<details><summary>Solution</summary>

<p>

```bash
kubectl run nginx --image=nginx
kubectl describe pod nginx | grep -i "Service Account" #should display 'default'
```

</p>
</details>



**2. Disable the <code>ServiceAccount</code> admission controller. Note: it may take a few minutes for the cluster to restart.**

<details><summary>Solution</summary>

<p>

Open etc/kubernetes/manifests/kube-apiserver.yaml and add the following line to the list of commands: </br>
```bash
--disable-admission-plugins=ServiceAccount 
```
Save the file and wait for the cluster to restart

</p>
</details>



**3. Create another Pod using the <code>nginx</code> image. Which service account is associated with the new Pod?**

<details><summary>Solution</summary>

<p>

```bash
kubectl run nginx2 --image=nginx
kubectl describe pod nginx2 | grep -i "Service Account" #should return nothing
```

</p>
</details>



### Roles and RoleBindings

**1. Create a namespace named <code>test-roles</code>.**

<details><summary>Solution</summary>

<p>

```bash
kubectl create ns test-roles
kubectl get ns
```

</p>
</details>



**2.	Set this namespace to be used in the current context.**

<details><summary>Solution</summary>

<p>

```bash
kubectl config set-context --current --namespace=test-roles
```

</p>
</details>



**3.	Create the <code>deployment-manager</code> Role in the <code>test-roles</code> namespace  that allows the following operations on Deployments: <code>get, list, watch, create,</code> and <code>update</code>.**

<details><summary>Solution</summary>

<p>

deployment-manager-role.yaml

```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: test-roles
  name: deployment-manager
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update"]
```
```bash
kubectl apply -f deployment-manager-role.yaml
kubectl get roles
```
</p>
</details>



**4.	Create the <code>deployment-manager-sa</code> ServiceAccount in the <code>test-roles</code> namespace. Use the imperative command to create the ServiceAccount.**

<details><summary>Solution</summary>

<p>

```bash
kubectl create sa deployment-manager-sa
kubectl get sa
```

</p>
</details>



**5.	Create the <code>deployment-manager-binding</code> RoleBinding to bind the role <code>deployment-manager</code> to the <code>deployment-manager-sa</code> ServiceAccount.**

<details><summary>Solution</summary>

<p>
deployment-manager-binding.yaml

```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployment-manager-binding
  namespace: test-roles
subjects:
- kind: ServiceAccount
  name: deployment-manager-sa
  namespace: test-roles
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f deployment-manager-binding.yaml
kubectl get rolebindings
```
</p>
</details>



**6.	Test your work by impersonating the ServiceAccount to list the Deployments using the following command:**

```bash
kubectl auth can-i list deployments --namespace=test-roles --as=system:serviceaccount:test-roles:deployment-manager-sa #should display 'yes'
```
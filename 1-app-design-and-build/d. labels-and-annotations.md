## Labels And Annotations

### Labels

**1. Create a wordpress pod and add the following labels: environment=production and role=website-builder. Then, create an nginx pod and label it with environment=pre-production**

<details><summary>Solution</summary>
<p>

```bash
kubectl run wordpress --image=wordpress
kubectl label pod wordpress environment=production role=website-builder
kubectl run nginx --image=nginx 
kubectl label pod nginx environment=pre-production
```

</p>
</details>
<br>

**2. Show all pods where the environment is either production or pre-production**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -l 'environment in (production, pre-production)'
```

</p>
</details>
<br>

**3. Show all pods where the 'run' label is set to nginx**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -l run=nginx
```

</p>
</details>
<br>

**4. Show all pods where the 'run' label is not set to nginx**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -l run!=nginx
```

</p>
</details>
<br>

**5. List all pods with the environment=production label and the role=website-builder label.**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -l environment=production,role=website-builder
```

</p>
</details>
<br>

**6. Get the labels of the wordpress pod.**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pod wordpress --show-labels
```

</p>
</details>
<br>

**7. List all pods with an additional column displaying the value of the 'environment' label for each pod.**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -L environment
```

</p>
</details>
<br>

**8. Remove the environment=pre-production label from the nginx pod.**

<details><summary>Solution</summary>
<p>

```bash
kubectl label pod nginx environment-
```

</p>
</details>
<br>

### Annotations

**1. Create an nginx pod and display its annotations.**

<details><summary>Solution</summary>
<p>

```bash
kubectl run nginx --image=nginx
kubectl describe pod nginx
```

</p>
</details>
<br>

**2. Annotate the nginx pod with contact=test@mail.com and deployed-by="Test Actions"**

<details><summary>Solution</summary>
<p>

```bash
kubectl annotate pod nginx contact=test@mail.com deployed-by="Test Actions"
kubectl describe pod nginx
```

</p>
</details>
<br>

**3. Remove the annotations from the pod and delete the pod.**

<details><summary>Solution</summary>
<p>

```bash
kubectl annotate pod nginx contact- deployed-by-
kubectl describe pod nginx
kubectl delete pod nginx
```

</p>
</details>
<br>
## Labels And Annotations

* [Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "Labels")
* [Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/ "Annotations")

### Labels

**1. Create a <code>wordpress</code> Pod and add the following labels: <code>environment=production</code> and <code>role=website-builder</code>. Then, create an <code>nginx</code> Pod and label it with <code>environment=pre-production</code>**

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


**2. Show all Pods where the environment is either <code>production</code> or <code>pre-production</code>**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -l 'environment in (production, pre-production)'
```

</p>
</details>


**3. Show all Pods where the <code>run</code> label is set to <code>nginx</code>**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -l run=nginx
```

</p>
</details>


**4. Show all Pods where the <code>run</code> label is not set to <code>nginx</code>**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -l run!=nginx
```

</p>
</details>


**5. List all Pods with the <code>environment=production</code> label and the <code>role=website-builder</code> label.**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -l environment=production,role=website-builder
```

</p>
</details>


**6. Get the labels of the <code>wordpress</code> Pod.**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pod wordpress --show-labels
```

</p>
</details>


**7. List all Pods with an additional column displaying the value of the <code>environment</code> label for each Pod.**

<details><summary>Solution</summary>
<p>

```bash
kubectl get pods -L environment
```

</p>
</details>


**8. Remove the <code>environment=pre-production</code> label from the <code>nginx</code> Pod.**

<details><summary>Solution</summary>
<p>

```bash
kubectl label pod nginx environment-
```

</p>
</details>


### Annotations

**1. Create an <code>nginx</code> Pod and display its annotations.**

<details><summary>Solution</summary>
<p>

```bash
kubectl run nginx --image=nginx
kubectl describe pod nginx
```

</p>
</details>


**2. Annotate the <code>nginx</code> Pod with <code>contact=test@mail.com</code> and <code>deployed-by="Test Actions"</code>**

<details><summary>Solution</summary>
<p>

```bash
kubectl annotate pod nginx contact=test@mail.com deployed-by="Test Actions"
kubectl describe pod nginx
```

</p>
</details>


**3. Remove the annotations from the Pod and delete the Pod.**

<details><summary>Solution</summary>
<p>

```bash
kubectl annotate pod nginx contact- deployed-by-
kubectl describe pod nginx
kubectl delete pod nginx
```

</p>
</details>

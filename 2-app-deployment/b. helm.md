## Helm

**1. Add the Bitnami charts repository to your Helm configuration using the URL: https://charts.bitnami.com/bitnami**

<details><summary>Solution</summary>
<p>

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

</p>
</details>


**2. List all the charts in the repo**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami
```

</p>
</details>


**3.	Search for the <code>jenkins</code> chart in the repo**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami/jenkins
```

</p>
</details>


**4. Show all values of the <code>jenkins</code> chart**

<details><summary>Solution</summary>
<p>

```bash
helm show values bitnami/jenkins
```

</p>
</details>


**5.	Show all versions of the <code>jenkins</code> chart**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami/jenkins --versions
```

</p>
</details>


**6.	Install the <code>jenkins</code> chart with chart version <code> 12.0.0</code>  and 4 Replicas. Then, list all Pods.**

<details><summary>Solution</summary>
<p>

```bash
helm install jenkins bitnami/jenkins --version 12.0.0 --set replicaCount=4
kubectl get pods
```

</p>
</details>


**7.	List all releases, including broken ones**

<details><summary>Solution</summary>
<p>

```bash
helm ls -a
```

</p>
</details>


**8.	Upgrade the <code>jenkins</code> release to version <code> 13.0.0</code>**

<details><summary>Solution</summary>
<p>

```bash
helm upgrade jenkins bitnami/jenkins --version 13.0.0
```

</p>
</details>


**9.	Uninstall the <code>jenkins</code> release**

<details><summary>Solution</summary>
<p>

```bash
helm uninstall jenkins
```

</p>
</details>



**10.	Pull the latest <code> redis</code>  chart to your home folder and unzip it**

<details><summary>Solution</summary>
<p>

```bash
helm repo update
cd ~
helm pull bitnami/redis --untar
```

</p>
</details>


**11.	Show the environment variables that Helm uses for configuration**

<details><summary>Solution</summary>
<p>

```bash
helm env
```

</p>
</details>

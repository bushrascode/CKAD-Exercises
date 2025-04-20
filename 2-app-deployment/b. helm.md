## Helm

**1. Add the Bitnami charts repository to your Helm configuration using the URL: https://charts.bitnami.com/bitnami**

<details><summary>Solution</summary>
<p>

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

</p>
</details>
<br>

**2. List all the charts in the repo**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami
```

</p>
</details>
<br>

**3.	Search for the Jenkins chart in the repo**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami/jenkins
```

</p>
</details>
<br>

**4. Show all values of the Jenkins chart**

<details><summary>Solution</summary>
<p>

```bash
helm show values bitnami/jenkins
```

</p>
</details>
<br>

**5.	Show all versions of the Jenkins chart**

<details><summary>Solution</summary>
<p>

```bash
helm search repo bitnami/jenkins --versions
```

</p>
</details>
<br>

**6.	Install the Jenkins chart with chart version 12.0.0 and 4 replicas. Then, list all pods.**

<details><summary>Solution</summary>
<p>

```bash
helm install jenkins bitnami/jenkins --version 12.0.0 --set replicaCount=4
kubectl get pods
```

</p>
</details>
<br>

**7.	List all releases, including broken ones**

<details><summary>Solution</summary>
<p>

```bash
helm ls -a
```

</p>
</details>
<br>

**8.	Upgrade the Jenkins release to version 13.0.0**

<details><summary>Solution</summary>
<p>

```bash
helm upgrade jenkins bitnami/jenkins --version 13.0.0
```

</p>
</details>
<br>

**9.	Uninstall the Jenkins release**

<details><summary>Solution</summary>
<p>

```bash
helm uninstall jenkins
```

</p>
</details>
<br>


**10.	Pull the latest Redis chart to your home folder and unzip it**

<details><summary>Solution</summary>
<p>

```bash
helm repo update
cd ~
helm pull bitnami/redis --untar
```

</p>
</details>
<br>

**11.	Show the environment variables that Helm uses for configuration**

<details><summary>Solution</summary>
<p>

```bash
helm env
```

</p>
</details>
<br>
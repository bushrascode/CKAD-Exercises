## Understand Configmaps

* [Configmaps](https://kubernetes.io/docs/concepts/configuration/configmap/ "Configmaps")

### Configmap from literal values

**1.	Create a ConfigMap named <code>username-cm</code> imperatively with the key <code>username</code> and the value <code>myuser</code>**

<details><summary>Solution</summary>

<p>

```bash
kubectl create cm username-cm --from-literal=username=myuser
kubectl get cm
```
</p>
</details>



**2.	Mount the ConfigMap <code>username-cm</code> in a new nginx Pod under the path <code>/etc/username</code>, and set <code>readOnly: true</code>. Create the Pod**

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
  containers:
  - image: nginx
    name: nginx
    volumeMounts:                   #add
    - name: myvolume                #add
      mountPath: "/etc/username"    #add
      readOnly: true                #add
  volumes:                          #add
  - name: myvolume                  #add
    configMap:                      #add
      name: username-cm             #add
```
</p>
</details>



**3.	Exec into the <code>nginx</code> Pod and verify the ConfigMap in the <code>/etc/username</code> folder**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it nginx -- sh
cd etc/username
cat username #should display 'myuser'
```
</p>
</details>



**4.	Delete the <code>nginx</code> Pod and the Configmap**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod nginx --force --grace-period=0
kubectl delete cm username-cm
```
</p>
</details>



### Configmap from env file

**1. Create a <code>config.env</code> file that contains the following key-value pair: <code>DATABASE_URL=postgres://user:password@localhost:5432/mydb</code>**

<details><summary>Solution</summary>

<p>

```bash
vim config.env
#then, paste DATABASE_URL=postgres://user:password@localhost:5432/mydb into the file
```
</p>
</details>



**2. Create a ConfigMap named <code>database-cm</code> imperatively using the <code>config.env</code> file**

<details><summary>Solution</summary>

<p>

```bash
kubectl create cm -h
kubectl create cm database-cm --from-env-file=config.env
kubectl get cm
```
</p>
</details>



**3. Create an <code>nginx</code> Pod and configure the container to use environment variables from the ConfigMap <code>database-cm</code>**

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
  containers:
  - image: nginx
    name: nginx
    envFrom:                    #add
      - configMapRef:           #add
          name: database-cm     #add
```
```bash
kubectl apply -f nginx.yaml
```
</p>
</details>



**4. Exec into the <code>nginx</code> Pod and list all environment variables**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it nginx -- sh
env #should display DATABASE_URL=postgres://user:password@localhost:5432/mydb among other env variables
```
</p>
</details>



**5.	Delete the <code>nginx</code> Pod and the <code>database-cm</code> ConfigMap**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod nginx --force --grace-period=0
kubectl delete cm database-cm
```
</p>
</details>



### Configmap from text file

**1.	Create a <code>config.txt</code> file that contains the value <code>myuser2</code>.**

<details><summary>Solution</summary>

<p>

```bash
vim config.txt
#then, paste 'myuser2' into the file
```
</p>
</details>



**2.	Create a ConfigMap named <code>username2-cm</code> using the <code>config.txt</code> file, specifying <code>username2</code> as key.**

<details><summary>Solution</summary>

<p>

```bash
kubectl create cm username2-cm --from-file=username2=config.txt
kubectl get cm
```
</p>
</details>



**3.	Create an <code>nginx</code> Pod and set an env variable named <code>CONFIGMAP_USERNAME2</code> that takes the value associated with the key <code>username2</code> from the ConfigMap <code>username2-cm</code>**

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
  containers:
  - image: nginx
    name: nginx
    env:                            #add
    - name: CONFIGMAP_USERNAME2     #add
      valueFrom:                    #add
        configMapKeyRef:            #add
          name: username2-cm        #add
          key: username2            #add
                                
```

</p>
</details>



**4.	Exec into the <code>nginx</code> Pod and list all environment variables**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it nginx -- sh
env #should display CONFIGMAP_USERNAME2=myuser2 among other env variables
```
</p>
</details>



**5.	Delete the Pod and the Configmap**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod nginx --force --grace-period=0
kubectl delete cm username2-cm
```
</p>
</details>


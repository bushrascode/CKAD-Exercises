## Understand Configmaps

### Configmap from literal values

**1.	Create a ConfigMap named 'username-cm' imperatively with the key 'username' and the value 'myuser'**

<details><summary>Solution</summary>

<p>

```bash
kubectl create cm username-cm --from-literal=username=myuser
kubectl get cm
```
</p>
</details>

<br/>

**2.	Mount the ConfigMap 'username-cm' in a new nginx pod under the path '/etc/username', and set readOnly: true. Create the pod**

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

<br/>

**3.	Exec into the nginx pod and verify the ConfigMap in the '/etc/username' folder**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it nginx -- sh
cd etc/username
cat username #should display 'myuser'
```
</p>
</details>

<br/>

**4.	Delete the nginx pod and the configmap**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod nginx --force --grace-period=0
kubectl delete cm username-cm
```
</p>
</details>

<br/>

### Configmap from env file

**1. Create a config.env file that contains the following key-value pair: DATABASE_URL=postgres://user:password@localhost:5432/mydb**

<details><summary>Solution</summary>

<p>

```bash
vim config.env
#then, paste DATABASE_URL=postgres://user:password@localhost:5432/mydb into the file
```
</p>
</details>

<br/>

**2. Create a ConfigMap named 'database-cm' imperatively using the config.env file**

<details><summary>Solution</summary>

<p>

```bash
kubectl create cm -h
kubectl create cm database-cm --from-env-file=config.env
kubectl get cm
```
</p>
</details>

<br/>

**3. Create an nginx pod and configure the container to use environment variables from the ConfigMap 'database-cm'**

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

<br/>

**4. Exec into the nginx pod and list all environment variables**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it nginx -- sh
env #should display DATABASE_URL=postgres://user:password@localhost:5432/mydb among other env variables
```
</p>
</details>

<br/>

**5.	Delete the nginx pod and the 'database-cm' ConfigMap**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod nginx --force --grace-period=0
kubectl delete cm database-cm
```
</p>
</details>

<br/>

### Configmap from text file

**1.	Create a config.txt file that contains the value 'myuser2'.**

<details><summary>Solution</summary>

<p>

```bash
vim config.txt
#then, paste 'myuser2' into the file
```
</p>
</details>

<br/>

**2.	Create a ConfigMap named 'username2-cm' using the config.txt file, specifying 'username2' as key.**

<details><summary>Solution</summary>

<p>

```bash
kubectl create cm username2-cm --from-file=username2=config.txt
kubectl get cm
```
</p>
</details>

<br/>

**3.	Create an nginx pod and set an env variable named CONFIGMAP_USERNAME2 that takes the value associated with the key 'username2' from the ConfigMap 'username2-cm'**

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

<br/>

**4.	Exec into the nginx pod and list all environment variables**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it nginx -- sh
env #should display CONFIGMAP_USERNAME2=myuser2 among other env variables
```
</p>
</details>

<br/>

**5.	Delete the pod and the configmap**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod nginx --force --grace-period=0
kubectl delete cm username2-cm
```
</p>
</details>

<br/>
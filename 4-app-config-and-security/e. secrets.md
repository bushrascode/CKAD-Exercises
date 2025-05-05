## Create & Consume Secrets

* [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/ "Secrets")

### Create generic Secret

**1.	Create a Secret named <code>password-secret</code> from literal values containing the key-value pair <code>password=pass123!</code> , and display the Secret as YAML.**

<details><summary>Solution</summary>

<p>

```bash
kubectl create secret generic password-secret --from-literal=password=pass123!
kubectl get secret password-secret -o yaml
```
</p>
</details>



**2.	Create a <code>busybox</code> Pod that has an environment variable named <code>PASSWORD</code>, which takes its value from the Secret <code>password-secret</code>. The <code>busybox</code> container should execute the command: <code>echo $PASSWORD; sleep 3600</code>**

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
  containers:
  - image: busybox
    name: busybox
    command: ["sh", "-c", "echo $PASSWORD; sleep 3600"]    #add
    env:                                                   #add
    - name: PASSWORD                                       #add
      valueFrom:                                           #add
        secretKeyRef:                                      #add
          name: password-secret                            #add
          key: password                                    #add
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
</p>
</details>



**3. Check the logs of the <code>busybox</code> Pod.**

<details><summary>Solution</summary>

<p>

```bash
kubectl logs busybox #should display pass123!
```
</p>
</details>



**4.	Delete the <code>busybox</code> Pod.**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod busybox --force --grace-period=0 
```
</p>
</details>



**5.	Modify the definition of the <code>busybox</code> Pod created in step 1 so that the Secret <code>password-secret</code> is mounted as a volume to the Pod using <code>mountPath: /etc/secrets</code>. The volume should be named <code>secret-volume</code>.**

<details><summary>Solution</summary>

<p>
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
  containers:
  - image: busybox
    name: busybox
    command: ["sh", "-c", "echo $PASSWORD; sleep 3600"]
    volumeMounts:                           #add
      - name: secret-volume                 #add
        mountPath: "/etc/secrets"           #add
  volumes:                                  #add
    - name: secret-volume                   #add
      secret:                               #add
        secretName: password-secret         #add
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
</p>
</details>



**6.	Create the new <code>busybox</code> Pod, exec into the Pod, and check the contents of the <code>/etc/secrets</code> folder.**

<details><summary>Solution</summary>

<p>

```bash
kubectl apply -f busybox.yaml
kubectl exec -it busybox -- sh
cd etc/secrets
cat password #should display pass123!
```
</p>
</details>



### Create TLS Secret

**1. Create a TLS certificate using the following command: <code>openssl req -x509 -newkey rsa:2048 -keyout tls.key -out tls.crt -days 365 -nodes</code>.**

**2.	Create a Secret named <code>tls-secret</code> using the certificate and key files.**

<details><summary>Solution</summary>

<p>

```bash
kubectl create secret -h #check for syntax
kubectl create secret tls -h #check for syntax
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
kubectl get secrets
```
</p>
</details>



**3.	Create the YAML file for a new <code>busybox</code> Pod named <code>busybox2</code> that never restarts. Mount the Secret <code>tls-secret</code> in the <code>busybox2</code> Pod using the <code>mountPath: /etc/tls-certs</code>. The Pod should execute the command <code>sleep 3600</code>. Create the Pod.**

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
    command: ["sh","-c","sleep 3600"]        #add
    resources: {}
    volumeMounts:                            #add
      - name: secret-volume                  #add
        readOnly: true                       #add
        mountPath: "/etc/tls-certs"          #add
  volumes:                                   #add
    - name: secret-volume                    #add
      secret:                                #add
        secretName: tls-secret               #add
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
kubectl apply -f busybox2.yaml
```
</p>
</details>



**4.	Exec into the Pod and check the <code>/etc/tls-certs</code> folder.**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it busybox2 -- sh
cd etc/tls-certs/
ls #should display the key and the cert file
```
</p>
</details>


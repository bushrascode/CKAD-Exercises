## Create & Consume Secrets

* [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/ "Secrets")

### Create generic secret

**1.	Create a secret named 'password-secret' from literal values containing the key-value pair password=pass123!, and display the secret as YAML.**

<details><summary>Solution</summary>

<p>

```bash
kubectl create secret generic password-secret --from-literal=password=pass123!
kubectl get secret password-secret -o yaml
```
</p>
</details>



**2.	Create a busybox Pod that has an environment variable named PASSWORD, which takes its value from the secret 'password-secret'. The busybox container should execute the command: 'echo $PASSWORD; sleep 3600'**

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



**3. Check the logs of the busybox Pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl logs busybox #should display pass123!
```
</p>
</details>



**4.	Delete the busybox Pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod busybox --force --grace-period=0 
```
</p>
</details>



**5.	Modify the definition of the busybox Pod created in step 1 so that the secret 'password-secret' is mounted as a volume to the Pod using mountPath: /etc/secrets. The volume should be named 'secret-volume'.**

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



**6.	Create the new busybox Pod, exec into the Pod, and check the contents of the /etc/secrets folder.**

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



### Create TLS secret

**1. Create a TLS certificate using the following command: 'openssl req -x509 -newkey rsa:2048 -keyout tls.key -out tls.crt -days 365 -nodes'**

**2.	Create a secret named 'tls-secret' using the certificate and key files**

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



**3.	Create the YAML file for a new busybox Pod named 'busybox2' that never restarts. Mount the secret 'tls-secret' in the 'busybox2' Pod using the mount path '/etc/tls-certs'. The Pod should execute the command 'sleep 3600'. Create the Pod.**

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



**4.	Exec into the Pod and check the '/etc/tls-certs' folder**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it busybox2 -- sh
cd etc/tls-certs/
ls #should display the key and the cert file
```
</p>
</details>


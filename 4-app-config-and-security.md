# IV.	Application Environment Configuration And Security

## Discover and use resources that extend Kubernetes (CRD, Operators)

**1. Create a CustomResourceDefinition for a Car with the following specifications: <br/> &nbsp;&nbsp;&nbsp;&nbsp; a) Kind: Car <br/> &nbsp;&nbsp;&nbsp;&nbsp; b)	Name: cards.stable.example.com <br/> &nbsp;&nbsp;&nbsp;&nbsp; c)	Scope: Namespaced <br/> &nbsp;&nbsp;&nbsp;&nbsp; d)	Singular name: car <br/> &nbsp;&nbsp;&nbsp;&nbsp; e)	Plural name: cars <br/> &nbsp;&nbsp;&nbsp;&nbsp; f)	Properties: <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; i. color: string <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ii. brand: string <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; iii. model: string <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; iv. year: int <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; v. electric: bool**

<details><summary>Solution</summary>

<p>

car-definition.yaml

```YAML
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: cars.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: cars
    singular: car
    kind: Car
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              color:
                type: string
              brand:
                type: string
              model:
                type: string
              year:
                type: integer
              electric:
                type: boolean
```

```bash
kubectl apply -f car-definition.yaml
kubectl get crd
```

</p>
</details>
<br/>

**2.	Create a custom Car object with the following properties: <br/> &nbsp;&nbsp;&nbsp;&nbsp; a) Color: black <br/> &nbsp;&nbsp;&nbsp;&nbsp; b) Brand: Tesla <br/> &nbsp;&nbsp;&nbsp;&nbsp; c) Model: 3 <br/> &nbsp;&nbsp;&nbsp;&nbsp; d) Year: 2024 <br/> &nbsp;&nbsp;&nbsp;&nbsp; e) Electric: true**

<details><summary>Solution</summary>

<p>

car.yaml

```YAML
apiVersion: stable.example.com/v1
kind: Car
metadata:
  name: tesla
spec:
  color: black
  brand: Tesla
  model: "3"
  year: 2024
  electric: true
```
```bash
kubectl apply -f car.yaml
```
</p>
</details>
<br/>

**3. List all Car resources in the namespace**

<details><summary>Solution</summary>

<p>

```bash
kubectl get cars
```

</p>
</details>

<br/>

## Understand authentication, authorization and admission control

### Admission controllers

**1. Create a pod using the nginx image. Which service account is associated with the pod?**

<details><summary>Solution</summary>

<p>

```bash
kubectl run nginx --image=nginx
kubectl describe pod nginx | grep -i "Service Account" #should display 'default'
```

</p>
</details>

<br/>

**2. Disable the ServiceAccount admission controller. Note: it may take a few minutes for the cluster to restart**

<details><summary>Solution</summary>

<p>

Open etc/kubernetes/manifests/kube-apiserver.yaml and add the following line to the list of commands: </br>
```bash
--disable-admission-plugins=ServiceAccount 
```
Save the file and wait for the cluster to restart

</p>
</details>

<br/>

**3. Create another pod using the nginx image. Which service account is associated with the new pod?**

<details><summary>Solution</summary>

<p>

```bash
kubectl run nginx2 --image=nginx
kubectl describe pod nginx2 | grep -i "Service Account" #should return nothing
```

</p>
</details>

<br/>

### Roles and RoleBindings

**1. Create a namespace named 'test-roles'**

<details><summary>Solution</summary>

<p>

```bash
kubectl create ns test-roles
kubectl get ns
```

</p>
</details>

<br/>

**2.	Set this namespace to be used in the current context**

<details><summary>Solution</summary>

<p>

```bash
kubectl config set-context --current --namespace=test-roles
```

</p>
</details>

<br/>

**3.	Create the 'deployment-manager' Role in the 'test-roles' namespace  that allows the following operations on Deployments: get, list, watch, create, and update**

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

<br/>

**4.	Create the 'deployment-manager-sa' ServiceAccount in the 'test-roles' namespace. Use the imperative command to create the ServiceAccount**

<details><summary>Solution</summary>

<p>

```bash
kubectl create sa deployment-manager-sa
kubectl get sa
```

</p>
</details>

<br/>

**5.	Create the 'deployment-manager-binding' RoleBinding to bind the role 'deployment-manager' to the 'deployment-manager-sa' ServiceAccount**

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

<br/>

**6.	Test your work by impersonating the ServiceAccount to list the deployments using the following command:**

```bash
kubectl auth can-i list deployments --namespace=test-roles --as=system:serviceaccount:test-roles:deployment-manager-sa #should display 'yes'
```

## Understand Requests, Limits, Quotas

### Requests, Limits

**1.	Create a wordpress pod with resource requests set to 64Mi of memory and 0.2 CPU core, and limits set to 128Mi of memory and 0.5 CPU, applied at the container level**

<details><summary>Solution</summary>

<p>

```bash
kubectl run wordpress --image=wordpress --dry-run=client -o yaml > wordpress.yaml
```
wordpress.yaml

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: wordpress
  name: wordpress
spec:
  containers:
  - image: wordpress
    name: wordpress
    resources:
      requests:               #add
        memory: "64Mi"        #add
        cpu: 0.2              #add
      limits:                 #add
        memory: "128Mi"       #add
        cpu: 0.5              #add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>

<br/>

**2.	Force the immediate deletion of the pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod wordpress --force --grace-period=0
```

</p>
</details>

<br/>

### LimitRanges

**1.	Create a LimitRange for Pods for CPU usage. Set the maximum to 0.5 CPU and the minimum to 100m CPU.**

<details><summary>Solution</summary>

<p>
limitrange.yaml

```YAML
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - max: 
      cpu: "0.5"
    min:
      cpu: 100m
    type: Pod
```
```bash
kubectl apply -f limitrange.yaml
kubectl get limitrange
```
</p>
</details>

<br/>

**2.	Create a wordpress pod with 64Mi of memory and 0.2 CPU core for requests, and limits of 128Mi of memory and 0.6 CPU. Was the pod creation successful?**

<details><summary>Solution</summary>

<p>

wordpress2.yaml

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: wordpress
  name: wordpress
spec:
  containers:
  - image: wordpress
    name: wordpress
    resources:
      requests:               #add
        memory: "64Mi"        #add
        cpu: 0.2              #add
      limits:                 #add
        memory: "128Mi"       #add
        cpu: 0.6              #add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
```bash
kubectl apply -f wordpress2.yaml #should display Forbidden error
```
</p>
</details>

<br/>

**3.	Delete the LimitRange**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete limitrange cpu-resource-constraint
```

</p>
</details>

<br/>

### Quotas

**1. Create a namespace named 'test-quota'**

<details><summary>Solution</summary>

<p>

```bash
kubectl create ns test-quota
kubectl get ns
```

</p>
</details>

<br/>

**2.	Create a ResourceQuota that sets the following hard limits for the 'test-quota' namespace: cpu: 5, memory: 5Gi, and pods: 2**

<details><summary>Solution</summary>

<p>

```bash
kubectl create quota myquota --hard=cpu=5,memory=5Gi,pods=2 -n test-quota
kubectl get quota -n test-quota
```

</p>
</details>

<br/>

**3.	Create a deployment using the nginx image with 3 replicas in the 'test-quota' namespace. Set resource requests to 64Mi of memory and 0.2 CPU core, and limits to 128Mi of memory and 0.5 CPU, applied at the container level. Was the creation successful? Are all pods running?**

<details><summary>Solution</summary>

<p>

```bash
kubectl create deploy nginx --image=nginx --replicas=3 -n test-quota --dry-run=client -o yaml > nginx.yaml
```
nginx.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
  namespace: test-quota
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: 
          limits:              #add
            cpu: 0.5           #add
            memory: "128Mi"    #add
          requests:            #add
            cpu: 0.2           #add
            memory: "64Mi"     #add
status: {}
```
```bash
kubectl apply -f nginx.yaml 
kubectl get pods -n test-quota #only 2 pods should be running
```
</p>
</details>

<br/>

**4.	Adjust the ResourceQuota so that all 3 pods can run**

<details><summary>Solution</summary>

<p>

```bash
kubectl edit quota myquota -n test-quota
```

```YAML
...
spec:
  hard:
    cpu: "5"
    memory: 5Gi
    pods: "3" #change
...
```
```bash
kubectl rollout restart deploy nginx -n test-quota
kubectl get pods -n test-quota #should display 3 nginx pods
```

</p>
</details>

<br/>

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

## Create & Consume Secrets

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

<br/>

**2.	Create a busybox pod that has an environment variable named PASSWORD, which takes its value from the secret 'password-secret'. The busybox container should execute the command: 'echo $PASSWORD; sleep 3600'**

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

<br/>

**3. Check the logs of the busybox pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl logs busybox #should display pass123!
```
</p>
</details>

<br/>

**4.	Delete the busybox pod**

<details><summary>Solution</summary>

<p>

```bash
kubectl delete pod busybox --force --grace-period=0 
```
</p>
</details>

<br/>

**5.	Modify the definition of the busybox pod created in step 1 so that the secret 'password-secret' is mounted as a volume to the pod using mountPath: /etc/secrets. The volume should be named 'secret-volume'.**

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

<br/>

**6.	Create the new busybox pod, exec into the pod, and check the contents of the /etc/secrets folder.**

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

<br/>

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

<br/>

**3.	Create the YAML file for a new busybox pod named 'busybox2' that never restarts. Mount the secret 'tls-secret' in the 'busybox2' pod using the mount path '/etc/tls-certs'. The pod should execute the command 'sleep 3600'. Create the pod.**

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

<br/>

**4.	Exec into the pod and check the '/etc/tls-certs' folder**

<details><summary>Solution</summary>

<p>

```bash
kubectl exec -it busybox2 -- sh
cd etc/tls-certs/
ls #should display the key and the cert file
```
</p>
</details>

<br/>

## Understand ServiceAccounts

**1.	Create a ServiceAccount named 'my-sa'**

<details><summary>Solution</summary>

<p>

```bash
kubectl create sa my-sa
```
</p>
</details>

<br/>

**2.	Create an nginx pod and assign the 'my-sa' ServiceAccount to it.**

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

<br/>

**3.	Exec into the nginx pod and navigate to the folder that contains the token associated with the 'my-sa' ServiceAccount. Display the contents of the token file.**

<details><summary>Solution</summary>

<p>

```bash
k exec -it nginx -- sh
cd /var/run/secrets/kubernetes.io/serviceaccount
cat token
```
</p>
</details>

<br/>


**4.	Create another ServiceAccount named 'pod-lister-sa'**

<details><summary>Solution</summary>

<p>

```bash
kubectl create sa pod-lister-sa
kubectl get sa
```
</p>
</details>

<br/>

**5.	Create a Role named 'pod-lister' that grants permission to list pods**

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

<br/>

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

<br/>

**7.	Create a new nginx pod named 'nginx2' and assign the 'pod-lister-sa' ServiceAccount to it**

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

<br/>

## Understand Application Security

**1.	Create a busybox pod named 'busybox' that never restarts and runs as a non-root user. Set 'runAsUser: 1000', 'runAsGroup: 3000', and 'fsGroup: 2000'. Set 'allowPrivilegeEscalation: false' at the container level. The container should execute the command: 'sleep 3600'**

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
  securityContext:                              #add
    runAsUser: 1000                             #add
    runAsGroup: 3000                            #add
    fsGroup: 2000                               #add
  containers:
  - image: busybox
    name: busybox
    command: ["sh", "-c", "sleep 3600"]         #add
    securityContext:                            #add
      allowPrivilegeEscalation: false           #add
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
kubectl apply -f busybox.yaml
kubectl get pods
```
</p>
</details>

<br/>

**2.	Create another busybox pod named 'busybox2'. The container should execute the command 'sleep 3600' and be configured with the Linux capabilities "NET_ADMIN" and "SYS_TIME"**

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
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
kubectl apply -f busybox2.yaml
kubectl get pods
```
</p>
</details>

<br/>


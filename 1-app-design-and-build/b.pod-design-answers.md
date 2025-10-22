**Pods**
1. Create a Pod using the busybox image, then display all Pods in the default namespace.

kubectl run bushra-pod --image busybox
kubectl get pods

2. Create a new busybox Pod named busybox2 with restartPolicy set to Never.

kubectl run busybox2 --image busybox --restart Never
kubectl get pods

3. Create an nginx Pod and save its logs to app.log.

kubectl run pod1 --image nginx 
kubectl logs pod1 > app.log

4. Execute into the nginx Pod and list all environment variables.

kubectl exec -it pod1 -- bin/bash env 
exit to exit out of the container

5. Show the nginx Pod's IP.

kubectl get pods -o wide
kubectl describe pod pod1 | grep -i ip

here -i makes it case insensitive

6. Force the nginx Pod to be deleted immediately.

kubectl delete pods pod1 --force

7. Create a new nginx Pod. Then, create a busybox Pod that sends a request to the nginx Pod using wget {podIp}:80.



8. Create a YAML config file for an nginx Pod without applying it. Name the container nginx-container. Then, create the Pod.

kubectl run pod-1 --image nginx --dry-run=client -o yaml > pod-1.yaml
vi pod-1.yaml

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
            name: nginx-container # add 
            resources: {}
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        status: {}


kubectl apply -f pod-1.yaml


* Difference between docker and kubectl 
kubectl run <pod-name> --image <image-name>
-- here the pod name is the same as the container name
In docker you need to specify the container name so,
docker run --name <container-name> <tagged-image>



**ReplicaSets**
1. create a replicaset with 3 replicas using the wordpress image
You can't create a replicaset imperatively!!
vi wordpress.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: wordpress
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: wordpress
  template:
    metadata:
      labels:
        tier: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress

kubectl create -f wordpress.yaml
kubectl get rs    
kubectl get pods

2. create a replicaset from the following file, do you notice any errors? can they be fixed?
vi nginx-rs.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pod ** fix value
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx                      

kubectl create -f nginx-rs.yaml - error labels not matching 
vi nginx-rs.yaml - fix value
kubectl create -f nginx-rs.yaml


**Deployments**

1. Create a namespace called production

kubectl create ns production
kubectl get ns -A

2. Set the namespace to production for the current Kubernetes context.

kubectl config set-context current --namespace=production

3. Create the YAML file for a Deployment named nginx that uses the nginx:1.27 image with 2 replicas. Do not create the Deployment yet

kubectl create deploy nginx --image=nginx:1.27 --replicas=2 --dry-run=client -o yaml > nginx-deploy.yaml

4. Create the deployment in the production namespace, list all the Pods in that namespace.

kubectl create -f nginx-deploy.yaml --namespace=production 
kubectl get pods -n=production

5. Scale the deployment to 4 replicas and display all the Pods in the namespace

kubectl scale deploy nginx --replicas=4
kubectl get pods -n=production

6. Set the image of the deployment to nginx:2.224 and show the status of the deployment. Why does the deployment fail?

kubectl set image deploy nginx nginx=nginx:2.224
kubectl rollout status deploy nginx

7. Show all revisions of the deployment and revert it to a working one

kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=1
kubectl rollout history deploy nginx --revision=2
kubectl rollout undo deploy nginx --to-revision=1
kubectl get deploy nginx -o wide #the image version should be nginx:1.27
kubectl get pods # all pods should be running














**Multi-container Pod design patterns**

1. Create a Pod with 2 containers: one init container that uses the busybox image and executes the following command: ["sh", "-c", "echo 'Some things need to be initialized before the main container starts';", "sleep 20"], the second container should be named main-app and use the image: nginx

kubectl run pod1 --image=busybox --dry-run=client -o yaml > pod1.yaml
vi pod1.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  initContainers:
    - image: busybox
      name: initial-app
      command: ["sh", "-c", "echo 'Some things need to be initialized before the main container starts';", "sleep 20"]
  containers:
  - image: nginx
    name: main-app

kubectl create -f pod1.yaml
kubectl get pods

2. Create a Pod with 2 containers, both using the busybox image and executing the command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']. Exec into the first container and list all environment variables.

kubectl run pod2 --image=busybox --dry-run=client -o yaml > pod2.yaml
vi pod2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - image: busybox
    name: container1
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
  - image: busybox
    name: container2
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

kubectl exec -it pod2 --container=container2 -- env


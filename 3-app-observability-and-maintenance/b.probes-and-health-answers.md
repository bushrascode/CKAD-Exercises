**Implement probes and health checks**

1. Create a pod using the wordpress image. Implement a Readiness Probe that executes the command cat readme.html. The kubelet should wait 15 seconds before performing the first check, then run the check every 5 seconds. When did the pod's status change to READY?

kubectl run wordpress --image=wordpress --dry-run=client -o yaml > wordpress.yaml
vi wordpress.yaml

apiVersion: v1
kind: Pod
metadata:
  name: wordpress
spec:
  containers:
  - image: wordpress
    name: wordpress
    resources: {}
    readinessProbe:
       exec:
        command:
        - cat
        - readme.html
       initialDelaySeconds: 15 
       periodSeconds: 5 --- check after every 5 seconds
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl create -f wordpress.yaml 

2. Create a Pod using the equivalent/health_check_nginx image. Implement a Liveness Probe that calls the /health-check endpoint using the HTTP GET method on port 80. The kubelet should wait 10 seconds before performing the first check, and then execute the check every 5 seconds.

kubectl run nginx --image=equivalent/health_check_nginx --dry-run=client -o yaml > nginx.yaml
vi nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: equivalent/health_check_nginx
    name: nginx
    resources: {}
    livenessProbe:
      httpGet:
        path: /health-check
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
kubectl create -f nginx.yaml


**Application Deployment**

**Blue/Green Deployment**

1. Create a blue Deployment and a Service to expose it by applying the following files:

vi blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: blue
  name: blue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blue
  template:
    metadata:
      labels:
        app: blue
    spec:
      containers:
      - image: nginx:1.26
        name: nginx
kubectl create -f blue-deployment.yaml
kubectl get pods 
vi node-port-service.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: mysvc
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: blue    
kubectl create -f node-port-service.yaml
kubectl get svc

2. Find the IP of the node where the nginx Pod is running. Then, access the Pod using the NodePort service you created by running: curl {nodeIP}:{nodePort}

kubectl get pods -o wide
kubectl describe pod blue-6859bbf9-fgpd4 - its on node: minikube/192.168.49.2
kubectl get nodes -o wide  - 192.168.49.2 is the internal ip 
curl 192.168.49.2:30008

3. Create a YAML file for a Deployment named green that uses the nginx:1.27 image with 1 Replica. Then, create the Deployment.

kubectl create deploy green --image=nginx:1.27 --replicas=1 --dry-run=client -o yaml > green-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: green
  name: green
spec:
  replicas: 1
  selector:
    matchLabels:
      app: green
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: green
    spec:
      containers:
      - image: nginx:1.27
        name: nginx
        resources: {}
status: {}
kubectl create -f green-deploy.yaml
kubectl get deploy 

4. Update the NodePort service to point to the green Deployment instead of the blue one.

vi node-port-service.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: mysvc
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: green #change    
kubectl delete svc mysvc
kubectl create -f node-port-service.yaml
curl 192.168.49.2:30008

5. Delete the green Deployment, then try sending a request to the green Pod using the NodePort service.

kubectl delete deploy green 
curl 192.168.49.2:30008

6. Update the NodePort service to point back to the blue Deployment

kubectl delete svc mysvc
vi node-port-service.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: mysvc
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30008
  selector:
    app: blue #change    
kubectl create -f node-port-service.yaml
curl 192.168.49.2:30008


**Canary Deployment**

1. Create the stable httpd Deployment and a service to expose it by applying the following files:

vi stable-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: httpd
  name: httpd
spec:
  replicas: 4
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
        purpose: web-server
    spec:
      containers:
      - image: httpd:alpine
        name: httpd
kubectl create -f stable-deployment.yaml
kubectl get pods

vi svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: httpd
  name: my-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    purpose: web-server
kubectl create -f svc.yaml
kubectl get svc

2. Now you have to switch to a new version of the application that uses the caddy:alpine image. The transition should be gradual rather than immediate or automatic, using the Canary approach:
     a) 25% of requests should be directed to the new caddy:alpine image
     b) 75% of requests should continue to go to the old httpd image
     c) Ensure there are a total of 4 Pods running


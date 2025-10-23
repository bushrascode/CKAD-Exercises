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







**Canary Deployment**


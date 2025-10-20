1. Create an nginx Pod named nginx in the default namespace and set containerPort=80. Expose it using a ClusterIP service named nginx-clusterip on port 80. Then, list all services and endpoints in the default namespace. Next, create a busybox Pod that never restarts, exec into it, and send a request to the ClusterIP address of the nginx Pod

kubectl run nginx --image=nginx --port=80 --namespace=default 
kubectl expose pod nginx --port=80 --name=nginx-clusterip
kubectl get svc 
kubectl get ep - pods real ip and port 
* the service creates a stable IP and port, that forwards to one or more Endpoints (which are the actual pods ip and port)
kubectl expose pod nginx --port=80
kubectl get svc -- found 10.110.95.183
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
kubectl exec -it busybox -- sh
 wget -O- 10.110.95.183

2. Create a Pod using the httpd:alpine image and expose it via a NodePort service named httpd-nodeport on node port 30001, setting both the port and targetPort to 80. Then, get the node the Pod is assigned to, display its IP address, and access the Pod using {NodeIP}:30001

kubectl run httpd --image=httpd:alpine
kubectl expose pod httpd --port=80 --target-port=80 --type=NodePort --name=httpd-nodeport -o yaml > httpd-nodeport.yml
vi httpd-nodeport.yml
kubectl create -f httpd-nodeport.yml
kubectl get pods -o wide - pod httpd on minikube node and its ip 10.244.0.41


3. Create a Deployment using the nginx image with 2 Replicas, and expose it via a NodePort service named nginx-nodeport on port 20001, setting both the port and targetPort to 80. Were you able to successfully create the NodePort service? If any issues occurred, correct them accordingly.


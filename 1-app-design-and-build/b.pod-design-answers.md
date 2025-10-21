Pods
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




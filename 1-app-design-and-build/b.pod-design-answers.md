Pods

1)Create a Pod using the busybox image, then display all Pods in the default namespace

- kubectl run busybox <pod-name> --image=busybox
- kubectl get pods

2)Create a new busybox Pod named busybox2 with restartPolicy set to Never

- kubectl run busybox2 --image=busybox --restart=Never
- kubectl get pods 

3)Create an nginx Pod and save its logs to app.log

- kubectl run nginx --image=nginx
- kubectl logs nginx > app.log

4)Execute into the nginx Pod and list all environment variables.

- kubectl exec -it nginx /bin/bash -- env 
exec, lets you run a command inside a container that’s part of a Pod - think of it like SSH into a pod container

-it, is two flags i for interactive and t for terminal - together, -it means “give me an interactive shell inside this container.”

without -it, the shell wouldn’t feel interactive — you could run commands, but not “stay inside.”

/bin/bash , this is the program you’re starting inside the container, most linux containers have either bash (/bin/bash) or sh (/bin/sh)

--, this is a common CLI convention meaning: “everything after this goes to the command inside the container, not to kubectl.”
Without it, kubectl might think env is one of its own flags

5)Show the nginx Pod's IP.

- kubectl get pods nginx -o wide

6)Force the nginx Pod to be deleted immediately

- kubectl delete pod nginx --grace-period=0 --force 

--grace-period=0 → tell K8s “don’t wait, kill instantly.”

--force → required if you’re bypassing normal graceful termination


ReplicaSets
1)Create a ReplicaSet with 3 replicas using the wordpress image

-
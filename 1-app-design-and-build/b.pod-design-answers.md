Pods

1)Create a Pod using the busybox image, then display all Pods in the default namespace

- kubectl run busybox <pod-name> --image=busybox
- kubectl get pods

2)Create a new busybox Pod named busybox2 with restartPolicy set to Never

- kubectl run busybox2 --image=busybox --restart=Never
- kubectl get pods 




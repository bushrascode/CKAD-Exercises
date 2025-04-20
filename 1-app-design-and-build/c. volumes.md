### Volumes – ephemeral and persistent

* [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/ "Persistent Volumes")
* [Ephemeral Volumes](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/ "Ephemeral Volumes")

**1. Create a pod with 2 containers, one called 'writer' and one called 'reader'. Both containers should use the busybox image. The pod should have an ephemeral volume called 'shared-volume'. Both containers will mount this volume at the path /data. Additionally, the 'writer' container will execute the following command: echo 'Hello from the writer container!' > /data/message.txt && sleep 3600. Create the pod, then exec into the 'reader' container and check if the message.txt file is created**

<details><summary>Solution</summary>
<p>

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-volume-demo
spec:
  containers:
    - name: writer
      image: busybox
      command:
        - sh
        - "-c"
        - "echo 'Hello from the writer container!' > /data/message.txt && sleep 3600"
      volumeMounts:
        - mountPath: /data
          name: shared-volume
    - name: reader
      image: busybox
      command:
        - sh
        - "-c"
        - "sleep 5 && cat /data/message.txt && sleep 3600"
      volumeMounts:
        - mountPath: /data
          name: shared-volume
  volumes:
    - name: shared-volume
      emptyDir: {}
```

```bash
kubectl exec -it ephemeral-volume-demo --container=writer -- sh   
cd data
```
</p>
</details>
<br>

**2.	Create a pod that mounts a persistent volume (PV) using a persistent volume claim (PVC). The data will persist even if the pod is deleted.**
1. Create a persistent volume with access mode ReadWriteOnce and 1Gi storage. Specify /mnt/data as the hostPath
2. Create a persistent volume claim with access mode ReadWriteOnce, storageClass set to "", requesting 1Gi storage, and bound to the volume created in a)
3. Create a pod that uses the busybox image and mounts the volume referenced by the persistent volume claim. The mounted volume should be called 'my-storage'. The container will mount the volume at the path /data and execute the following command: echo 'Hello, Persistent Volume!' > /data/message.txt && sleep 3600. Create the pod's YAML, then create the pod. Next, change the command to "echo 'Hello, Persistent Volume!' && sleep 3600", delete the pod itself, and create it again. Exec into the pod and check if yo stil find the message.txt file in the /data folder

<details><summary>Solution</summary>
<p>

1.

```YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

2.

```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  volumeName: mypv
  storageClassName: ""
```

3.

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
    command: ["sh","-c","echo 'Hello, Persistent Volume!' > /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: my-storage
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
    - name: my-storage
      persistentVolumeClaim:
        claimName: mypvc
status: {}
```

```bash
kubectl exec -it busybox -- sh   
cd data
```

</p>
</details>
<br>
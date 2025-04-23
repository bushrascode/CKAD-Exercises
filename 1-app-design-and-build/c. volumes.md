### Volumes – ephemeral and persistent

* [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/ "Persistent Volumes")
* [Ephemeral Volumes](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/ "Ephemeral Volumes")

**1. Create a Pod with 2 containers, named <code>writer</code> and <code>reader</code>. Both containers should use the <code>busybox</code> image. The Pod should have an ephemeral volume named <code>shared-volume</code>. Both containers will mount this volume at the path <code>/data</code>. Additionally, the <code>writer</code> container should execute the following command: <code>echo 'Hello from the writer container!' > /data/message.txt && sleep 3600</code>. Create the Pod, then exec into the <code>reader</code> container and check if the message.txt file is created**

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


**2.	Create a Pod that mounts a Persistent Volume (PV) using a Persistent Volume Claim (PVC). The data will persist even if the Pod is deleted.**
1. Create a Persistent Volume with access mode <code>ReadWriteOnce</code> and <code>1Gi</code> storage. Specify <code>/mnt/data</code> as the <code>hostPath</code>
2. Create a Persistent Volume claim with access mode <code>ReadWriteOnce</code>, <code>storageClass</code> set to <code>""</code>, requesting <code>1Gi</code> storage, and bound to the volume created in step 1.
3. Create a Pod that uses the <code>busybox</code> image and mounts the volume referenced by the Persistent Volume claim. The mounted volume should be named <code>my-storage</code>. The container will mount the volume at the path <code>/data</code> and execute the following command: <code>echo 'Hello, Persistent Volume!' > /data/message.txt && sleep 3600</code>. Create the Pod's YAML, then create the Pod. Next, change the command to <code>"echo 'Hello, Persistent Volume!' && sleep 3600"</code>, delete the Pod itself, and create it again. Exec into the Pod and check if you still find the message.txt file in the <code>/data</code> folder

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

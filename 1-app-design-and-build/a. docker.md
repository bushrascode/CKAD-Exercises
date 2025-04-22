## Define, build and modify container images

The exam expects you to be comfortable using both Docker and Podman. These two are syntactically quite similar. Feel free to solve the following exercises using either Docker or Podman. <br>

**1. Build a Docker image using the following Dockerfile and tag it as <code> my-username/alpine </code>**

```YAML
FROM alpine:latest 
CMD ["sh", "-c", "echo 'Container is running...'; sleep 3600"]
```

<details><summary>Solution</summary>
<p>

Create a Dockerfile that contains the code above, then run the following command:

```bash
docker build -t my-username/alpine .
```

</p>
</details>

**2. List all Docker images**

<details><summary>Solution</summary>
<p>

```bash
docker image list
```

</p>
</details>

**3. Create a container using the <code> my-username/alpine </code> image, run it in the background, and name it <code> my-container </code>**

<details><summary>Solution</summary>
<p>

```bash
docker run -d --name=my-container my-username/alpine:latest
```

</p>
</details>

**4. Show all running containers**

<details><summary>Solution</summary>
<p>

```bash
docker ps
```

</p>
</details>

**5. Write the logs of the container to logs.txt**

<details><summary>Solution</summary>
<p>

```bash
docker logs my-container > logs.txt
```

</p>
</details>

**6. Stop the container**

<details><summary>Solution</summary>
<p>

```bash
docker stop my-container
```

</p>
</details>


**7. List all containers. Do you see the stopped one?**

<details><summary>Solution</summary>
<p>

```bash
docker ps -a
```

</p>
</details>


**8. Export the container to a TAR archive**

<details><summary>Solution</summary>
<p>

```bash
docker export my-container > my-container.tar
```

</p>
</details>


**9. Delete the container**

<details><summary>Solution</summary>
<p>

```bash
docker rm my-container
```

</p>
</details>


**10. Save the image as a TAR file**

<details><summary>Solution</summary>
<p>

```bash
docker save my-username/alpine:latest > my-image.tar
```

</p>
</details>


**11. Delete the image**

<details><summary>Solution</summary>
<p>

```bash
docker image rm my-username/alpine:latest
```

</p>
</details>


**12. Load the deleted image from the archive file and retag it as <code> my-username/alpine:v1 </code>**

<details><summary>Solution</summary>
<p>

```bash
docker load < my-image.tar
docker tag my-username/alpine:latest my-username/alpine:v1
docker image list
```

</p>
</details>

* PRACTICE AGAIN!!

1. Build a Docker image using the Dockerfile below and tag it as my-username/alpine

mkdir -p ~/ckad-practice
vi ~/ckad-practice/Dockerfile
cd ~/ckad-practice
docker build -t bushra-image/alpine .

2. List all Docker images

docker images

3. Create a container using the my-username/alpine image, run it in the background, and name it my-container

docker run -d --name bushra-container bushra-image/alpine

4. Show all running containers

docker ps

5. Write the logs of the container to logs.txt

docker log bushra-container > logs.txt

6. Stop the container

docker stop bushra-container

7. List all containers. Do you see the stopped one?

docker ps -a

8. Export the container to a TAR archive.

docker export bushra-container > archive.tar

9. Delete the container

docker rm bushra-container 

10. Save the image as a TAR file

docker save bushra-image/alpine > image.tar

11. Delete the image

docker rmi bushra-image/alpine

12. Load the deleted image from the archive file and re-tag it as my-username/alpine:v1

docker load < image.tar
docker tag bushra-image/alpine bushra-image/alpine:v1
docker images 


notes:

| Command             | Works With               | Purpose                                                                  |
| ------------------- | ------------------------ | ------------------------------------------------------------------------ |
| **`docker save`**   | 🧱 Docker **images**     | Archive/export an image (for backup, sharing, moving between systems)    |
| **`docker load`**   | 🧱 Docker **images**     | Import a saved image `.tar` file back into Docker                        |
| **`docker export`** | 🚀 Docker **containers** | Export the **filesystem** of a running/stopped container (not an image!) |





1)Build a Docker image using the Dockerfile below and tag it as my-username/alpine
FROM alpine:latest 
CMD ["sh", "-c", "echo 'Container is running...'; sleep 3600"]

- docker build -t bushra-fatima/alpine 

2)List all docker images

- docker images 
can also use docker image list

3)Create a container using the my-username/alpine image, run it in the background, and name it my-container

- docker run -d --name=bushra-container bushra-fatima/alpine

4)Show all running containers

- docker ps

5)Write the logs of the container to logs.txt

- docker logs <container_id> > logs.txt

6)Stop the container

- docker stop <container_id>

7)List all containers. Do you see the stopped one?

- docker ps -a

8)Export the container to a TAR archive

- docker export <container_id> > bushra-container.tar

9)Delete the container

- docker rm <first 2 char of container id>

10)Save the image as a TAR file

- docker save <image_id> > <file_name>.tar

11)Delete the image

- docker rmi <image_id>

12)Load the deleted image from the archive file and retag it as my-username/alpine:v1

- docker load < <image-filename>.tar
- docker tag <image-sha> bushra-fatima/alpine:v1
- docker images (to check it is done correctly)
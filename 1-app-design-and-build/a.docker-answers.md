1)Build a Docker image using the Dockerfile below and tag it as my-username/alpine
FROM alpine:latest 
CMD ["sh", "-c", "echo 'Container is running...'; sleep 3600"]

- docker build -t bushra-fatima/alpine 

2)List all docker images

- docker images 
can also use docker image list

3)Create a container using the my-username/alpine image, run it in the background, and name it my-container

- docker run -d --name=bushra-container bushra-fatima/alpine

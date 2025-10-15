* PRACTICE AGAIN!!

1. Build a Docker image using the Dockerfile below and tag it as my-username/alpine
2. List all Docker images
3. Create a container using the my-username/alpine image, run it in the background, and name it my-container
4. Show all running containers
5. Write the logs of the container to logs.txt
6. Stop the container
7. List all containers. Do you see the stopped one?
8. Export the container to a TAR archive.
9. Delete the container
10. Save the image as a TAR file
11. Delete the image
12. Load the deleted image from the archive file and re-tag it as my-username/alpine:v1


















notes:

| Command             | Works With               | Purpose                                                                  |
| ------------------- | ------------------------ | ------------------------------------------------------------------------ |
| **`docker save`**   | 🧱 Docker **images**     | Archive/export an image (for backup, sharing, moving between systems)    |
| **`docker load`**   | 🧱 Docker **images**     | Import a saved image `.tar` file back into Docker                        |
| **`docker export`** | 🚀 Docker **containers** | Export the **filesystem** of a running/stopped container (not an image!) |





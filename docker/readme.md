## Images & Containers

**Images** are **blueprints** / **templates** for containers. They are **read-only** and contain the application as well as the necessary application environment.

**Images** **do not run themselves**, instead, they can be executed as **containers**.

**Images** are either **pre-built** (e.g. official Images you find on DockerHub) or you build your own Images by defining a Dockerfile. Dockerfiles contain instructions which are executed when an image is built ( `docker build` . ), every instruction then creates a **layer** in the image. Layers are used to efficiently rebuild and share images. The **CMD** instruction **is special**: **It's not executed when the image is built but when a container is created and started based on that image**

**Containers**
Containers are running instances of Images. When you create a container (via docker run ), a thin read-write layer is added on top of the Image. Multiple Containers can therefore be started based on one and the same Image. All Containers run in isolation.


Every instruction in a Dockerfile is considerd a "Layer" and are cached.



## Key Docker Commands

full list of available commands:
 

    docker --help

Build a Dockerfile and create your own Image based on the DockerFile:

    docker build .

Assign a NAME and a TAG to an image

       docker build . -t $NAME:$TAG 

Create and start a new container based on image IMAGENAME (or use the image id).

    docker run $IMAGE_NAME

Run the container in detached mode.

     docker run $IMAGE_NAME -d
Run in interactive mode

    docker run $IMAGE_NAME -it

Automatically remove the container when stopped.

    docker run $IMAGE_NAME --rm 

List all running containers

    docker ps
List all containers including stopped ones.

    docker ps -a 
    
List all locally stored images

    docker images
      
Remove a container with the name CONTAINER (you can also use the container ID)

    docker rm $CONTAINER

Remove an image by name /id 

    docker rmi $IMAGE
Remove all untagged images

    docker image prune

Remove all locally stored images

    docker image prune -a

Push an image to DockerHub (image name tag must include the repo name and url)

    docker push $IMAGE
Pull image from DockerHub

    docker pull $IMAGE

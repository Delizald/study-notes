## Images & Containers

**Images** are **blueprints** / **templates** for containers. They are **read-only** and contain the application as well as the necessary application environment.

**Images** **do not run themselves**, instead, they can be executed as **containers**.

**Images** are either **pre-built** (e.g. official Images you find on DockerHub) or you build your own Images by defining a Dockerfile. Dockerfiles contain instructions which are executed when an image is built ( `docker build` . ), every instruction then creates a **layer** in the image. Layers are used to efficiently rebuild and share images. The **CMD** instruction **is special**: **It's not executed when the image is built but when a container is created and started based on that image**

**Containers**
Containers are running instances of Images. When you create a container (via docker run ), a thin read-write layer is added on top of the Image. Multiple Containers can therefore be started based on one and the same Image. All Containers run in isolation.

Every instruction in a Dockerfile is considerd a "Layer" and are cached.


**Volumes**
Volumes are managed by docker.
Docker sets a path in local machine managed via "docker volume" command
Anonymous volumes: only exists when container is running
anonymous volumes are removed automatically, when a container is removed.
This happens when you start / run a container with the --rm option.
If you start a container without that option, the anonymous volume would NOT be removed, even if you remove the container (with docker rm ...)

Named Volumes: A defined path in the container is mapped to the created volume/mount

Bind Mounts (Code Sharing): You define a folder/path on host machine. Good for editable, persistent data.

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

Start a container

    docker start $CONTAINERNAME 

Attach to a detached container

    docker attach CONTAINER_NAME

See logs printed by container:

    docker logs CONTAINER_NAME

Continously see logs printed by container:

    docker logs -f CONTAINER_NAME

Docker interactive mode with tty:

    docker run -it IMAGE_ID
    docker start -a -i CONTAINER_NAME

To remove containers:

    docker stop CONTAINER_NAME
    docker rm CONTAINER_NAME
    docker container prune // to delete all containers

Inspect images:

    docker image inspect IMAGE_ID

Copy files into and from a container:

    docker cp $fileordir CONTAINER_NAME:/pathwithincontainer
    docker cp CONTAINER_NAME:/pathwithincontainer $fileordir 

list volumes

  docker volume ls

  Create a named volume:
    docker run -d -p 3000:80 --rm --name app -v name:/app/name

remove anonymous volumes:

    docker volume rm VOL_NAME
    docker volumne prune

Create a bind mount:

    docker run -d -p 3000:80 --rm --name app -v name:/app/name -v "pathinhostmachine:/app" -v /app/node_modules

Create a volume

    docker volume create NAME

Remove volume

    docker volume rm NAME


#This is a sample Dockerfile
# Every instruction is considered a 'layer' and are cached
FROM node 

# All commands will be run from this dir on the image.
WORKDIR /app

# Copy from host dir to image/container file system
COPY package.json /app

# run a command 
RUN npm install

COPY . /app

# Expose a port from the container to the host. This is optional but it is a best practice to use it
EXPOSE 80

#CMD will execute when a container is started
CMD ["node", "server.js"]

#build this with
# > docker build . #this will give you a containerID
# > docker run -p 3000:80 $containerID  -> https://localhost:3000 maps to port 80 in the container
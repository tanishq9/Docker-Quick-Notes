## Introduction
### Virtualisation
- We need to have a hypervisor on our (same) machine which lets us run VMs (virtual machines) running our application(s).
- Each VM has its own OS.
- System resources are consumed by this hypervisor and VMs, what is leftover is used by the application running in different VMs on the same host machine/hardware.

### Kernel
- Kernel is the heart of OS.
- It is the bridge between application and system resources/hardware.
- To use the kernel (cannot be used directly) we add a layer with set of utilities, OS = Kernel + layer.
- The difference between linux distros like alpine, ubuntu, centos is the layer, they use same kernel.

### VMs vs Docker
- Each VM has its own OS apart from running our app.
- In Docker, the OS of hardware/host machine is used unlike VM hence leaving us with more system resources to be actually used by our application.

### Note
- Using docker, we can package our application with all its dependencies and runtime, this ensures no coupling between our application and the host machine it runs on.
- Nowadays, we package our application as a docker image, which is one more step than the previous approach wherein we just created jar/war for our application.
- Docker cannot work on Mac/Windows directly, when we install docker on Mac/Windows, docker would install a linux VM to work, docker needs linux kernel to work.


## Docker Terminologies
- docker **build**: Creating snapshot from Dockerfile (human readable instructions for app to run).
- docker **image**: Snapshot (Lightweight VM)
- docker **container**: Running instance of an image, similar to an instance of a java object.
- docker **run**: Creating a docker container, similar to instantiating an instance of a java object.
- docker **network**
   - By default when we create multiple containers, docker would place them in bridge network, problem with this approach is that no name server is involved because of this containers can only talk if they know each other's IP address, we cannot rely on this as it changes on deleting and recreate containers.
   - We will create our custom bridge network instead of using the default one.
- Dockerfile
   - FROM [image], the base image for your docker image.
   - ADD [host-dir] [container-dir]
   - RUN [command], command to execute during the image build process, useful to install any software.
   - ENV [key] [value], sets an environment variable.
   - WORKDIR [path], creates a workspace/default working directory, if we ignore, root directory would be used.
      - The WORKDIR command is used to define the working directory of a Docker container at any given time. The command is specified in the Dockerfile. Any RUN , CMD , ADD , COPY , or ENTRYPOINT command will be executed in the specified working directory.
   - EXPOSE [port], exposes port.
   - CMD [command] and ENTRYPOINT [command], command to be executed when container is created.
      - In Dockerfiles, an ENTRYPOINT instruction is used to set executables that will always run when the container is initiated. Unlike CMD commands, ENTRYPOINT commands cannot be ignored or overridden.
- Layers 
   - Docker image = list of layers, each layer is cached.
   - When there are changes in some layer of the image, and the hash value changes then all subsequent layers are rebuild.
- Docker compose
   - It is a utility to create docker containers, networks, volume/port mappings, passing environment variable etc in a declarative way in a yaml file.
   - In docker-compose.yaml file, we mention the version at the top, which is the specification/convention we will follow.
   - The key difference between the Dockerfile and docker-compose is that the Dockerfile describes how to build Docker images, while docker-compose is used to run Docker containers.
   

## Commands
- docker images: show the list of images you have in your machine.
- docker run [image-name]: to create a container of the image. 
- docker pull [image-name]: to pull the image from DockerHub (default)
  - First we build image (using Dockerfile) then we run image (in docker container)
- docker pull image-name, by default, always pulls the latest version of an image, if version not specified.
   - Important: latest is actually the default version of the image and not the 'latest' one. It is not latest by timestamp. default would have been a better word.
- The docker ps is a Docker command to list the running containers by default; however, we can use different flags to get the list of other containers that are in stopped or exited status i.e. docker ps -a would list all containers whether running or stopped.
- docker run -it <image-name> like: docker run -it ubuntu 
  - This is to start container in interactive mode, -i: std-input and -t: std-output / attach terminal.
  - An obvious note, if we run docker run command then each time we would get a new container (similar to instantiating object of a class).
- docker start -ia [container-name]
   - The same container would be restarted in interactive mode.
- docker exec [container-name] [command]
   - docker exec --> To start a command on a running container.
- Image name format
   - [registry-host:port] / [username] / [image-name] [:tag]
   - Example default: docker.io/library/hello-world:latest 
- Port mapping: Concept using which we can map host port to a container port.
   - We can use -p option in the docker run command to mention this.
   - docker run -p host-port:container-port [image-name]
- docker run -d [image-name]
   - Above command would run container in detached mode i.e. container would run in background, we do not have to keep the terminal open.
- docker logs container
   - To output the container logs, options can be used to like -t (timestamp), -n (tail), -f (follow).
   - Example: docker logs 7f58c871f2d1 --timestamps --until=15m --tail=5
      - 7f58c871f2d1 is the container id.
- docker run -v /host-path:/container-path image-name
   - To map a specific directory to a container directory (use absolute paths).
   - docker run -p 80:80 -v $PWD:/usr/share/nginx/html:ro nginx [For read-only access in docker container]
     docker containers can talk to each other using a concept called docker network.
- docker network create <network-name>
   - When running containers, set flag for --network=<network-name> so that docker containers in that network can refer each other by container name and not IP.
- docker compose up 
   - It will create image name as mentioned in the file but it will prepend directory name and append a number to the name as well like directory_name_image_name_number
   - This command would also create a docker bridge network with name: directory_name_default.
   - If the file name is not docker-compose.yaml then we can specify which file to pick using -f flag like below:
      - docker compose -f example.yaml up
   - Use docker compose down command to remove resources (network and container) which were created using docker compose up command. 
- docker-compose up -d
   - This above command is used to run container in background/detached mode.
   - Example: docker compose -f nginx.yaml up -d
      - -f: It is an option for docker-compose
      - -d: It is an option for up command
   - To know more check: docker compose --help and docker compose up --help.
- docker compose --profile=app up
   - We can add profiles property for a service in services section of the docker compose file so that, that particular docker image is only run in case the profile has that specific value. Usage:
   - Only service having app in profiles would be up as part of above command.


## Example - Dockerfile

Installing java manually in Ubuntu container and then write Dockerfile for it.

Commands used manually:
- uname: Print certain system information
- apt-get update: On Linux operating systems that use the APT package management system, the apt-get command is used to install, remove, and perform other operations on installed software packages.
- apt-get install curl wget: To install curl and wget utilities using apt-get command.
- curl https://download.oracle.com/java/17/archive/jdk-17.0.5_linux-x64_bin.tar.gz --output java17: To download jdk17 version compressed file.
- tar -xzvf file_name: Extract (x) a compressed file (z) with verbose output (v) having file name (f).
- export PATH=$PATH:/jdk-17.0.5/bin: Add path variable for binary of java so that java command is understood in terminal however ./java would work by default in only the file location.

Dockerfile for above

```
FROM ubuntu
WORKDIR java
RUN apt-get update
RUN apt-get install curl -y # Yes to all prompts.
RUN curl https://download.oracle.com/java/17/archive/jdk-17.0.5_linux-x64_bin.tar.gz --output java17.tar.gz
RUN tar -xzvf java17.tar.gz
ENV PATH $PATH:/java/jdk-17.0.5/bin
```

Alternatively achieving below using ADD command.

```
FROM ubuntu
WORKDIR java
ADD https://download.oracle.com/java/17/archive/jdk-17.0.5_linux-x64_bin.tar.gz java17.tar.gz
RUN tar -xzvf java17.tar.gz
ENV PATH $PATH:/java/jdk-17.0.5/b
```


## Example - Docker compose 

- When we use docker compose then by default it creates a network for the project using the directory/folder name and places all the containers in that network so the containers can talk using container name.
- Since container 2 would now talk to container 1 so we can remove the port mapping added for container 1 to expose its port 80 on host machine port 80.

```
version: "3.0"
services:
   my-nginx-app:
      image: "nginx"
      ports:
      - "80:80"
      volumes:
      - "./data:/usr/share/nginx/html"
   util:
      image: "vinsdocker/util"
      depends_on:
      - my-nginx-app
      command: "curl my-nginx-app"
```

An example for env variables:

```
version: "3.0"
services:
   ubuntu:
      image: "ubuntu"
      command: "env"
      environment:
      - app.name=test-service
      - service.url=google.com
      - input=9
```

Important Note
- depends_on property only guarantees the order of docker container creation, there is no clause like wait for this much time, so it can happen that first container is created and the next container is also created subsequently but the creation of first container is still not done.
- We can use restart: always property so as to restart container incase of errors.


## Example - MongoDB persistence and init

- When a MongoDb container is started for the first time it will execute files with extensions .sh and .js that are found in /docker-entrypoint-initdb.d
- More here: https://hub.docker.com/_/mongo

```
version: "3.0"
services:
   mongodb:
      image: "mongo"
      volumes:
      - "./data:/data/db" # this is where MongoDB stores data
      - "./data:/docker-entrypoint-initdb.d" # When a container is started for the first time it will execute files with extensions .sh and .js that are found in /docker-entrypoint-initdb.d
   express:
      image: "mongo-express"
      restart: "always"
      depends_on:
      - mongodb
      ports:
      - "8081:8081"
      environment:
      - ME_CONFIG_MONGODB_SERVER: mongodb # MongoDB container name.
```

## Example - Postgres persistence and init

```
version: '3.1'
services:
   db:
      image: postgres
      volumes:
      # - "./data:/var/lib/postgresql/data" # volume mount
      - "./data:/docker-entrypoint-initdb.d" # init script
      environment:
         POSTGRES_USER: test
         POSTGRES_PASSWORD: test
         POSTGRES_DB: test
   adminer:
      image: adminer
      restart: always
      ports:
      - 8080:8080
```

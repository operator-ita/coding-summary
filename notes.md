# Docker 

- Docker makes it easier to package applications and add to CI/CD pipelines.
- Docker simplifies container technology to make creating and running containers easier. 
- Docker helps you package dependencies with containers. 

## Containers 

- **Containers** are a group of processes run in isolation. All processes MUST be able to run on the shared kernel
-  Each container has its own set of *"namespaces"* (isolated view). E.g. PID - processes IDs
- **cgroups** controls limits and monitoring resources 

It is recommended to install a [Desktop application] (https://docs.docker.com/install).

### Run a container 

1 -  Create a container with Ubuntu

``` bash
$ docker container run -t ubuntu top
```

- `docker container run -t ubuntu top` : You use the `docker container run` command to run a container with the Ubuntu image by using the `top` command.
-  `-t` flag allocates a pseudo-TTY, which you need for the `top` command to work correctly.
- `top` is a Linux utility that prints the processes on a system and orders them by resource consumption.

2 - List and get ID of running containers  

```bash
$ docker container ls 
```

3 - Run `bash` inside that container 

```bash
$ docker container exec -it b3ad2a23fab3 bash 
root@b3ad2a23fab3:/#
```
4 - Inspect running processes 

``` 
$ ps -ef
```
NOTE: 

PID is just one of the Linux namespaces that provides containers with isolation to system resources. Other Linux namespaces include:

- MNT: Mount and unmount directories without affecting other namespaces.
- NET: Containers have their own network stack.
- IPC: Isolated interprocess communication mechanisms such as message queues.
- User: Isolated view of users on the system.
- UTC: Set hostname and domain name per container.

These namespaces provide the isolation for containers that allow them to run together securely and without conflict with other containers running on the same system.

5 - See the logs from the application running in the container (Prints out what is sent to standard out by the application) whith: 
```bash
docker container logs [container id] 
```
- ``
### Run multiple containers

1 - Install NGINX by Official 
```bash 
$ docker container run --detach --publish 8080:80 --name nginx nginx
```
- `--detach` flag will run this container in the background.
-  `--publish` flag is a feature that can expose networking through the container onto the host. Here  publishes port 80 in the container (the default port for NGINX) by using port 8080 on your host
-  `--name` flas that names the container if don't want the dafault name assigned by docker 

Check the server at  http://localhost:8080 

2 - Install MongoDB 
```bash
docker container run --detach --publish 8081:27017 --name mongo mongo:3.4
```
- `:3.4` Use Tags install a specific version instead of the last one which is set as default. See dicomentation of container to see all tags possible. 
- `--flag` In thise particular  case the flag  expose the 27017 Mongo port on the host. And use 8080 since 8080  is already exposed on your host

### Remove containers

0 - List containers 
```bash
$ docker container ls
```
1  - Stop the container by id 
```bash
$ docker container stop [container id]
```
Tip:  You need to enter only enough digits of the ID to be unique. Three digits is typically adequate

2 - Remove the stopped containers,  and remove unused volumes and networks, and dangling images.

```bash
$ docker system prune 
```
- `-a` flag to remove any stopped containers and all unused images (not just dangling images). 
### Summary 

- Containers are composed of Linux **namespaces** and **control groups** that provide isolation from other containers and the host.
- Because of the isolation properties of containers, you can schedule many containers on a single host without worrying about conflicting dependencies. This makes it easier to run multiple containers on a single host: using all resources allocated to that host and ultimately saving server costs.
- You should avoid using unverified content from the Docker Store when developing your own images because these images might contain security vulnerabilities or possibly even malicious software.
- Containers include everything they need to run the processes within them, so you don't need to install additional dependencies on the host.
- It is possible to run Docker containers in other OS rather than linux thaks to **LinuxKit** that is essentially a container-native toolkit that allows organizations to build their own containerized operating systems that are secure, lean, modular and portable.
- An **image** is the blueprint for spinning up containers. An **image** is a TAR of a file system, and a **container** is a file system plus a set of processes running in isolation. 


## Docker Image 
- A `Dockerfile` file is used to create a  **docker image** an  after images are built, they will be sent to a central registry where they can be accessed by all environments (such as a test environment) that need to run instances of that application. 
- A **Docker Image** is a TAR of a file system.

### Create and build the Docker image 
1 - Create an **App**. For these example is a Python App. Run the next command into the terminal and a file name `app.py` will be created. 

```python
echo 'from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "hello world!"
if __name__ == "__main__":
    app.run(host="0.0.0.0")' > app.py
```
2 - Create a **Dockerfile**  which lists the instructions needed to build a Docker image. Create a file named `Dockerfile` and add the content: 
```bash
FROM python:3.6.1-alpine
RUN pip install flask
CMD ["python","app.py"]
COPY app.py /app.py
```
- `FROM` selects the starting image to build layers on top of.
  -  In this case, `python:3.6.1-alpine` is the base layer. The distribution is `alpine` which is smaller. To see more tags check [Python image](https://hub.docker.com/_/python/) on the Docker Hub.  Tip: Best practice to use a specific tag when inheriting a parent image so that changes to the parent dependency are controlled. If no tag is specified, the latest tag takes effect, which acts as a dynamic pointer that points to the latest version of an image.   
- `RUN`executes commands needed to set up your image for your application, such as installing packages, editing files, or changing file permissions.
  - In this case, you are installing Flask. The `RUN` commands are executed at build time and are added to the layers of your image.
- `CMD['','']`  is the command that is executed when you start a container. There can be only one `CMD` per Dockerfile. If you specify more than one `CMD`, then the last `CMD` will take effect
  -  It can be used the official Python image directly to run Python scripts without installing Python on the  host. However, in this case, we are creating a custom image to include the source so that it can build an image with our  application and ship it to other environments.
- `COPY` copies the file in the local directory (where you will run `docker image build`) into a new layer of the image. This instruction is the last line in the `Dockerfile`
  -  TIP: Layers that change frequently, such as copying source code into the image, should be placed near the bottom of the file to take full advantage of the Docker layer cache. This allows you to avoid rebuilding layers that could otherwise be cached. For instance, if there was a change in the `FROM` instruction, it will invalidate the cache for all subsequent layers of this image. You'll see this little later in this lab.

See the [full list of commands](https://docs.docker.com/engine/reference/builder/) that you can put into a Dockerfile.

3 - Build  a `docker image` 
```bash
$ docker image build -t python-hello-world .
```
- `-t`  parameter to name the image. 
- Note: do not forget to include the point at the end of the command. 

Verify the new image shows in the image list 
```bash
docker image ls
```

### Run the Docker image 
1 - Run the Docker image: 
```bash
$ docker run -p 5001:5000 -d python-hello-world
```
- `-p` flag that maps a port running inside the container to the host. 
	- Here, we're mapping the Python app running on port 5000 inside the container to port 5001 on the host	 	
See the results at http://localhost:5001 

### Push to a central registry
1 - Create a free account in `Docker Hub`. Docker Hub is a free service to publicly store available images. You can also pay to store private images. Although, most organizations that use Docker extensively will set up their own **registry** internally. 
2 - Log in to your Docker registry account by entering `docker login` on the terminal: 
```bash 
$ docker login
```
3 - Tag the image with a username 
```bash
$ docker tag python-hello-world [dockerhub username]/[image name]
```
- Where `[dockerhub username]/[image name]` are the variables to change. 
4 - Push the docker image to the Docker Hub registry 
```bash
$ docker push [dockerhub username]/[image name]
```
5 - Check the image on your Docker Hub. Now that your image is on Docker Hub, other developers and operators can use the `docker pull` command to deploy your image to other environment.  E.g. 

```bash
$ docker pull operadorita/python-hello-world 
```
After Pull... How to make a container??? 
```bash
$ docker container run --detach --publish 5001:5000 --name python-hellow-world operadorita/python-hello-world
```



###  Deploy a change 

1 - Modify your `App` . Here, we update `app.py`by replacing the String "Hello World" 

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello Beautiful World!"

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```

2 - **Rebuild**  the app by using your Docker Hub username in the build command:

```bash
docker image build -t [dockerhub username]/[image name] .
```

- Some layers displays "Using cache" , these layers of the Docker image have already been built, and the `docker image build` command will use these layers from the cache instead of rebuilding them.
- There is a caching mechanism in place for pushing layers too. Docker Hub already has all but one of the layers from an earlier push, so it only pushes the one layer that has changed.


### Remove Image 
0 - List images 
```bash
$ docker image ls 
```
1 - Remove an image by id 
```bash
docker rmi [id]
```
### Understanding Image Layers
...

You can inspect which files have been pulled up to the container level with the docker diff command. 

you can use the docker image history command of the Python image you created.

### Summary 

- Use the Dockerfile to create reproducible builds for your application and to integrate your application with Docker into the CI/CD pipeline.
- Docker images can be made available to all of your environments through a central registry. The Docker Hub is one example of a registry, but you can deploy your own registry on servers you control.
- A Docker image contains all the dependencies that it needs to run an application within the image. This is useful because you no longer need to deal with environment drift (version differences) when you rely on dependencies that are installed on every environment you deploy to.
- Docker uses of the union file system and "copy-on-write" to reuse layers of images. This lowers the footprint of storing images and significantly increases the performance of starting containers.
- Image layers are cached by the Docker build and push system. There's no need to rebuild or repush image layers that are already present on a system.
- Each line in a Dockerfile creates a new layer, and because of the layer cache, the lines that change more frequently, for example, adding source code to an image, should be listed near the bottom of the file.

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
docker image build -t [dockerhub username]/[image name]
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

You can inspect which files have been pulled up to the container level with the `docker diff` command. 

you can use the `docker image history` command of the Python image you created.

## Container Orchestration 

Commond problems for  running Dockerized applications in production are: Scheduling services across distributed nodes, Maintaining high avaibility, Implementation reconciliation, Scaling, Loggin, Service descovery, Zero Downtime Deployments, Fault Tolerance, A/B Deployments. 

Containers oschestration helps you with:  Cluster Management, Scheduling, Service discovery, Replication, Health management, Declare Desire State (Active reconcilation). table 

Container ecosystem layers 
| Layer | Description | Software |
|:-------:|:-------------------------------------------:|:-----------------------------------:|
| Layer 6 | Development Workflow Opinionated Containers | DEIS, OPENSHIT |
| Layer 5 | Orquestration/Scheduling Service Model | Kubernets, Docker Swarm, MESOS, MARATHON |
| Layer 4 | Container Engine | Docker, Rocket, OSv |
| Layer 3 | Operating System | Ubuntu, Redhat, CoreOS |
| Layer 2 | Virtual Infrastructure | vmware, AmazonWebServices |
| Layer 1 | Physical Infraestructure | Raw Compute, Network, Storage  |
### Create Swarm with workers 

0 - Create virtual machines with *VirutalMachine* and *docker-machine*  

```bash
$ docker-machine create node1
$ docker-machine create node2
$ docker-machine create node3
```

To access a *VirtualMachine* use the command 

```bash
$ docker-machine ssh node1 
```

1 - Initialize a Swarm in node 1: 

```bash
$ docker swarm init --advertise-addr eth0
Swarm initialized: current node (qybg0qy8aqe6sewpotaotwxnm) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5uuifm5ze6qu9e0hy6e750frqdvbiu6fku7v6c8wi3f47ez02x-bkc3wwmz5cfh2e9nimbkcqzpx 192.168.0.33:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

- `--advertise-addr` Specifies the address in which the other nodes will use to join the swarm
- `docker swarm init`  generates a join token. You need to use this token to join the other nodes to the swarm. 

2 - Paste the token given above in node2 and node3 to add workers 

```bash
$   docker swarm join --token SWMTKN-1-5uuifm5ze6qu9e0hy6e750frqdvbiu6fku7v6c8wi3f47ez02x-bkc3wwmz5cfh2e9nimbkcqzpx 192.168.0.33:2377
This node joined a swarm as a worker.
```

3 - Verify the `three-node-closter` back on node1 by listing existing nodes 

```bash 
$ docker node ls 
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
qybg0qy8aqe6sewpotaotwxnm *   node1               Ready               Active              Leader              19.03.4
pr60yx9e4ucin04zrvbrmcpbt     node2               Ready               Active                                  19.03.4
qwva3mm0tmfl20m8633o3zc6e     node3               Ready               Active                                  19.03.4
```

- The asterisk (*) next to the ID of the node represents the node that handled that specific command. 

- Managers handle commands and manage the state of the swarm. Workers cannot handle commands and are simply used to run containers at scale.  By default, managers are also used to run containers.

  **Note:** Although you control the swarm directly from the  node in which its running, you can control a Docker swarm remotely by  connecting to the Docker Engine of the manager by using the remote API or by activating a remote host from your local Docker installation  (using the `$DOCKER_HOST` and `$DOCKER_CERT_PATH`  environment variables). This will become useful when you want to remotely control production applications, instead of using SSH to  directly control production servers.

### Deploy a Service 

1 - Deploy a service by using NGINX: 

```bash 
$ docker service create --detach=true --name nginx1 --publish 80:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12
pgqdxr41dpy8qwkn6qm7vke0q
```

- `--mount`  is useful to have NGINX print out the hostname of the node it's running on. You will use this later in this lab when you start load balancing  between multiple containers of NGINX that are distributed across  different nodes in the cluster and you want to see which node in the swarm is serving the request.
- `--publish` Command uses the swarm's built-in routing mesh. In this case, port 80 is exposed on every node in
   the swarm. The routing mesh will route a request coming in on port 80 to one of the nodes running the container.

2 - Inspect the service. 

```bash
$ docker service ls 
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
qd8feg8n0qha        nginx1              replicated          1/1                 nginx:1.12          *:80->80/tcp
```

3 - Check the running container of the service.

```bash
$ docker service ps nginx1
```

- If you know which node your container is running on (you can see which node based on the output from `docker service ps`), you can use the command `docker container ls` to see the container running on that specific node.

4 - Test the service 

Because of the routing mesh, you can send a request to any node of the swarm on port 80. This request will be automatically routed to the one node that is running the NGINX container.

Try this command on each node:

```bash
$ curl localhost:80
node1
```

Curling will output the hostname where the container is running. For this example, it is running on node1, but yours might be different.

### Scale your service 

In production, you might need to handle large amounts of traffic to your application, so you'll learn how to scale.

1. Update your service with an updated number of replicas.

   Use the `docker service` command to update the NGINX service that you created previously to include 5 replicas. This is defining a new state for the service.

   ```bash
   $ docker service update --replicas=5 --detach=true nginx1 
   nginx1

   ```

   When this command is run, the following events occur:

   - The state of the service is updated to 5 replicas, which is stored in the swarm's internal storage.
   - Docker Swarm recognizes that the number of replicas that is scheduled now does not match the declared state of 5.
   - Docker Swarm schedules 4 more tasks (containers) in an attempt to meet the declared state for the service.

   This swarm is actively checking to see if the desired state is equal to actual state and will attempt to reconcile if needed.

   -------------------

   Is also possible use the command `scale` ? 

   ```bash
   $ docker service scale nginx1=5 
   ```

   ​

2. Check the running instances.

   After a few seconds, you should see that the swarm did its job and successfully started 4 more containers. Notice that the containers are scheduled across all three nodes of the cluster. The default placement strategy that is used to decide where new containers are to be run is the emptiest node, but that can be changed based on your needs.

   ```bash
   $ docker service ps nginx1

   ```

   ![img](https://courses.cognitiveclass.ai/asset-v1:IBMDeveloperSkillsNetwork+CO0101EN+v1+type@asset+block/lab3-scale_2.png)

3. Send a lot of requests to http://localhost:80.

   The `--publish 80:80` parameter is still in effect for this service; that was not changed when you ran the `docker service update` command. However, now when you send requests on port 80, the routing mesh has multiple containers in which to route requests to. The routing mesh acts as a load balancer for these containers, alternating where it routes requests to.

   Try it out by curling multiple times. Note that it doesn't matter which node you send the requests. There is no connection between the node that receives the request and the node that that request is routed to.

   ```
   $ curl localhost:80
   node3
   $ curl localhost:80
   node3
   $ curl localhost:80
   node2
   $ curl localhost:80
   node1
   $ curl localhost:80
   node1

   ```

   You should see which node is serving each request because of the useful `--mount` command you used earlier.

   **Limits of the routing mesh:** The routing mesh can publish only one service on port 80. If you want multiple services exposed on port 80, you can use an external application load balancer outside of the swarm to accomplish this.

4. Check the aggregated logs for the service.

   Another easy way to see which nodes those requests were routed to is to check the aggregated logs. You can get aggregated logs for the service by using the command `docker service logs [service name]`. This aggregates the output from every running container, that is, the output from `docker container logs [container name]`.

   ```bash
   $ docker service logs nginx1

   ```

   ![img](https://courses.cognitiveclass.ai/asset-v1:IBMDeveloperSkillsNetwork+CO0101EN+v1+type@asset+block/lab3-scale_4.png)

   Based on these logs, you can see that each request was served by a different container.

   In addition to seeing whether the request was sent to node1, node2, or node3, you can also see which container on each node that it was sent to. For example, `nginx1.5` means that request was sent to a container with that same name as indicated in the output of the command `docker service ps nginx1`.

###  Apply rolling updates

Now that you have your service deployed, you'll see a release of your application. You are going to update the version of NGINX to version 1.13.

1. Run the `docker service update` command:

   ```bash
   $ docker service update --image nginx:1.13 --detach=true nginx1

   ```

   This triggers a rolling update of the swarm. Quickly enter the command `docker service ps nginx1` over and over to see the updates in real time.

   You can fine-tune the rolling update by using these options:

   - `--update-parallelism`: specifies the number of containers to update immediately (defaults to 1).
   - `--update-delay:` specifies the delay between finishing updating a set of containers before moving on to the next set.

2. After a few seconds, run the command `docker service ps nginx1` to see all the images that have been updated to nginx:1.13.

   ```bash
   $ docker service ps nginx1

   ```

   ![img](https://courses.cognitiveclass.ai/asset-v1:IBMDeveloperSkillsNetwork+CO0101EN+v1+type@asset+block/lab3-rolling_updates_0.png)

   You have successfully updated your application to the latest version of NGINX.

### Reconcile problems with containers

In the previous section, you updated the state of your service by using the command `docker service update`. You saw Docker Swarm in action as it recognized the mismatch between desired state and actual state, and attempted to solve the issue.

The inspect-and-then-adapt ** model of Docker Swarm enables it to perform reconciliation when something goes wrong. For example, when a node in the swarm goes down, it might take down running containers with it. The swarm will recognize this loss of containers and will attempt to reschedule containers on available nodes to achieve the desired state for that service.

You are going to remove a node and see tasks of your nginx1 service be rescheduled on other nodes automatically.

1. To get a clean output, create a new service by copying the following line. Change the name and the publish port to avoid conflicts with your existing service. Also, add the 

   ```bash
   --replicas
   ```

    option to scale the service with five instances:

   ```bash
   $ docker service create --detach=true --name nginx2 --replicas=5 --publish 81:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12
   aiqdh5n9fyacgvb2g82s412js

   ```

2. On node1, use the ` watch` utility to watch the update from the output of the  command: 

   ```bash
   $ docker service ps
   ```

   **Tip:** `watch` is a Linux utility and might not be available on other operating systems.

   ```bash 
   $ watch -n 1 docker service ps nginx2

   ```

   This command should create output like this:

   ![img](https://courses.cognitiveclass.ai/asset-v1:IBMDeveloperSkillsNetwork+CO0101EN+v1+type@asset+block/lab3-reconcile_2.png)

3. Click node3 and enter the command to leave the swarm cluster:

   ```bash
   $ docker swarm leave

   ```

   **Tip**: This is the typical way to leave the swarm, but you can also kill the node and the behavior will be the same.

4. Click node1 to watch the reconciliation in action. You should see that the swarm attempts to get back to the declared state by rescheduling the containers that were running on node3 to node1 and node2 automatically.

   ![img](https://courses.cognitiveclass.ai/asset-v1:IBMDeveloperSkillsNetwork+CO0101EN+v1+type@asset+block/lab3-reconcile_4.png)

### Determine how many nodes you need

In this lab, your Docker Swarm cluster consists of one master and two worker nodes. This configuration is not highly available. The manager node contains the necessary information to manage the cluster, but if this node goes down, the cluster will cease to function. For a production application, you should provision a cluster with multiple manager nodes to allow for manager node failures.

You should have at least three manager nodes but typically no more than seven. Manager nodes implement the raft consensus algorithm, which requires that more than 50% of the nodes agree on the state that is being stored for the cluster. If you don't achieve more than 50% agreement, the swarm will cease to operate correctly. For this reason, note the following guidance for node failure tolerance:

- Three manager nodes tolerate one node failure.
- Five manager nodes tolerate two node failures.
- Seven manager nodes tolerate three node failures.

It is possible to have an even number of manager nodes, but it adds no value in terms of the number of node failures. For example, four manager nodes will tolerate only one node failure, which is the same tolerance as a three-manager node cluster. However, the more manager nodes you have, the harder it is to achieve a consensus on the state of a cluster.

While you typically want to limit the number of manager nodes to no more than seven, you can scale the number of worker nodes much higher than that. Worker nodes can scale up into the thousands of nodes. Worker nodes communicate by using the gossip protocol, which is optimized to be perform well under a lot of traffic and a large number of nodes.

If you are using Play-with-Docker, you can easily deploy multiple manager node clusters by using the built in templates. Click the **Templates** icon in the upper left to view the available templates.

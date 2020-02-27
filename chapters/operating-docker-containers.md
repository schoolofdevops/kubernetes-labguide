# Lab: Getting Started with Docker Operations

In this chapter, we are going to learn about docker shell, the command line utility and how to use it to
launch containers. We will also learn what it means to run a container, its lifecycle and perform basic
operations such as creating, starting, stopping, removing, pausing containers and checking the status etc.


### Using docker cli  
We can use docker cli to interact with docker daemon. Various functions of docker command is given below. Try this yourself by runnig **$sudo docker** command  

```
docker
```  

[Output]  
```
Usage: docker [OPTIONS] COMMAND [arg...]
       docker [ --help | -v | --version ]

A self-sufficient runtime for containers.

Options:

  --config=~/.docker              Location of client config files
  -D, --debug                     Enable debug mode
  -H, --host=[]                   Daemon socket(s) to connect to
  -h, --help                      Print usage
  -l, --log-level=info            Set the logging level
  --tls                           Use TLS; implied by --tlsverify
  --tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA
  --tlscert=~/.docker/cert.pem    Path to TLS certificate file
  --tlskey=~/.docker/key.pem      Path to TLS key file
  --tlsverify                     Use TLS and verify the remote
  -v, --version                   Print version information and quit

Commands:
    attach    Attach to a running container
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders between a container and the local filesystem
    create    Create a new container
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    exec      Run a command in a running container
    export    Export a container's filesystem as a tar archive
    history   Show the history of an image
    images    List images
    import    Import the contents from a tarball to create a filesystem image
    info      Display system-wide information
    inspect   Return low-level information on a container, image or task
    kill      Kill one or more running containers
    load      Load an image from a tar archive or STDIN
    login     Log in to a Docker registry.
    logout    Log out from a Docker registry.
    logs      Fetch the logs of a container
    network   Manage Docker networks
    node      Manage Docker Swarm nodes
    pause     Pause all processes within one or more containers
    port      List port mappings or a specific mapping for the container
    ps        List containers
    pull      Pull an image or a repository from a registry
    push      Push an image or a repository to a registry
    rename    Rename a container
    restart   Restart a container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save one or more images to a tar archive (streamed to STDOUT by default)
    search    Search the Docker Hub for images
    service   Manage Docker services
    start     Start one or more stopped containers
    stats     Display a live stream of container(s) resource usage statistics
    stop      Stop one or more running containers
    swarm     Manage Docker Swarm
    tag       Tag an image into a repository
    top       Display the running processes of a container
    unpause   Unpause all processes within one or more containers
    update    Update configuration of one or more containers
    version   Show the Docker version information
    volume    Manage Docker volumes
    wait      Block until a container stops, then print its exit code

```

#### Getting Information about Docker Setup  
We can get the information about our Docker setup in several ways. Namely,  

```
docker version

docker -v

docker system info
```  


The **docker system info** command gives a lot of useful information like total number of containers and images along with information about host resource utilization  etc.

### Stream events from the docker daemon   
Docker **events** serves us with the stream of events or interactions that are happening with the docker daemon. This does not stream the log data of application inside the container. That is done by **docker logs** command. Let us see how this command works  
Open an another terminal. Let us call the old terminal as **Terminal 1** and the newer one as **Terminal 2**.

From Terminal 1, execute **docker events**. Now you are getting the data stream from docker daemon  

```
docker system events
```  



### Launching our first container  
Now we have a basic understanding of docker command and sub commands, let us dive straight into launching our very first **container**  

```
docker run alpine:3.4 uptime
```  

Where,  

  * we are using docker **client** to  
  * run a application/command **uptime** using    
  * an image by name **alpine:3.4**  

[Output]  
```
Unable to find image 'alpine:3.4' locally
3.4: Pulling from library/alpine
81033e7c1d6a: Pull complete
Digest: sha256:2532609239f3a96fbc30670716d87c8861b8a1974564211325633ca093b11c0b
Status: Downloaded newer image for alpine:3.4

 15:24:34 up  7:36,  load average: 0.00, 0.03, 0.04
```  

**What happened?**  

This command will  

  * Pull the **alpine** image file from **docker hub**, a cloud registry
  * Create a runtime environment/ container with the above image   
  * Launch a program (called **uptime**) inside that container  
  * Stream that output to the terminal  
  * Stop the container once the program is exited

**Where did my container go?**  

```
docker container  ps

docker container  ps -l
```

The point here to remember is that, when that executable stops running inside the container, the container itself will stop.


Let's see what happens when we run that command again,  

[Output]  
```
docker run alpine uptime
 07:48:06 up  3:15,  load average: 0.00, 0.00, 0.00
```  

Now docker no longer pulls the image again from registry, because **it has stored the image locally** from the previous run. So once an image is pulled, we can make use of that image to create and run as many container as we want without the need of downloading the image again. However it has created a new instance of the iamge/container.

### Checking Status of the containers  

We have understood how docker run commands works. But what if you want to see list of running containers and history of containers that had run and exited? This can be done by executing the following commands  

```
docker ps
```  

[Output]  

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```  
This command doesn't give us any information. Because, **docker ps** command will only show list of container(s) which are **running**  

```
docker ps -l
```  

[Output]  
```
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
988f4d90d604        alpine              "uptime"            About a minute ago   Exited (0) About a minute ago                       fervent_hypatia
```  
the **-l** flag shows the last run container along with other details like image it used, command it executed, return code of that command, etc.,  

```
docker ps -n 2
```  
[Output]  
```
NAMES
988f4d90d604        alpine              "uptime"            About a minute ago   Exited (0) About a minute ago                       fervent_hypatia
acea3023dca4        alpine              "uptime"            3 minutes ago        Exited (0) 3 minutes ago                            mad_darwin
```  
Docker gives us the flexibility to show the desirable number of last run containers. This can be achieved by using **-n #no_of_results** flag  

```
docker ps -a
```  

[Output]  

```
CONTAINER ID        IMAGE                        COMMAND                  CREATED              STATUS                          PORTS                  NAMES
988f4d90d604        alpine                       "uptime"                 About a minute ago   Exited (0) About a minute ago                          fervent_hypatia
acea3023dca4        alpine                       "uptime"                 4 minutes ago        Exited (0) 4 minutes ago                               mad_darwin
60ffa94e69ec        ubuntu:14.04.3               "bash"                   27 hours ago         Exited (0) 26 hours ago                                infallible_meninsky
dd75c04e7d2b        schoolofdevops/ghost:0.3.1   "/entrypoint.sh npm s"   4 days ago           Exited (0) 3 days ago                                  kickass_bardeen
c082972f66d6        schoolofdevops/ghost:0.3.1   "/entrypoint.sh npm s"   4 days ago           Exited (0) 3 days ago           0.0.0.0:80->2368/tcp   sodcblog

```  
This command will show all the container we have run so far.  

### Running Containers in Interactive Mode
We can interact with docker containers by giving -it flags at the run time. These flags stand for  
  * i - Interactive  
  * t - tty

```
docker run -it alpine:3.4 sh
```  

[Output]  

```
Unable to find image 'alpine:3.4' locally
latest: Pulling from library/alpine
ff3a5c916c92: Already exists
Digest: sha256:7df6db5aa61ae9480f52f0b3a06a140ab98d427f86d8d5de0bedab9b8df6b1c0
Status: Downloaded newer image for alpine:latest
/ #
```  

As you see, we have landed straight into **sh** shell of that container. This is the result of using **-it** flags and mentioning that container to run the **sh** shell. Don't try to exit that container yet. We have to execute some other commands in it to understand the next topic  

Namespaced:

Like a full fledged OS, Docker container has its own namespaces  
This enables Docker container to isolate itself from the host as well as other containers  

Run the following commands and see that alpine container has its own namespaces and not inheriting much from **host OS**  

NAMESPACED  

[Run the following commands inside the container]  

```
cat /etc/issue

ps aux

ifconfig

hostname
```




Shared(NON NAMESPACED):  

We have understood that containers have their own namespaces. But will they share something to some extent? the answer is **YES**. Let's run the following commands on both the container and the host machine  

[Run the following commands inside the container]  
```
uptime

uname -a

cat /proc/cpuinfo

date

free
```  

As you can see, the container uses the same Linux Kernel from the host machine. Just like **uname** command, the following commands share the same information as well. In order to avoid repetition, we will see the output of container alone.


Now exit out of that container by running **exit** or by pressing **ctrl+d**  


### Making Containers Persist  

#### Running Containers in Detached Mode  
So far, we have run the containers interactively. But this is not always the case. Sometimes you may want to start a container  without interacting with it. This can be achieved by using **"detached mode"** (**-d**) flag. Hence the container will launch the deafault application inside and run in the background. This saves a lot of time, we don't have to wait till the applications launches successfully. It will happen behind the screen. Let us run the following command to see this in action  

[Command]  

```
docker run -idt schoolofdevops/loop program
```  

-d , --detach : detached mode  

[Output]  

```
2533adf280ac4838b446e4ee1151f52793e6ac499d2e631b2c752459bb18ad5f
```  
This will run the container in detached mode. We are only given with full container id as the output  

Let us check whether this container is running or not  
[Command]  
```
docker ps
```  

[Output]  

```
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS               NAMES
2533adf280ac        schoolofdevops/loop   "program"           37 seconds ago      Up 36 seconds                           prickly_bose
```  
As we can see in the output, the container is running in the background  


#### Checking Logs

To check the logs, find out the container id/name and run the following commands, replacing 08f0242aa61c with your container id


[Commands]  
```
docker container ps

docker container logs <container_id/name>

docker container logs -f  <container_id/name>
```  



#### Connecting to running container to execute commands
We can connect to the containers which are running in detached mode by using these following commands  
[Command]  
```
docker exec -it <container_id/name> sh
```  

[Output]  

```
/ #
```  

You could try running any commands on the shell
e.g.

```
apk update
apk add vim
ps aux
```

Now exit the container.  



### Launching a container with a pre built app image  

To launch vote container run the following command. Don't bother about the new flag **-P** now. We will explain about that flag later in this chapter  
```
docker container run  -idt -P  schoolofdevops/vote
```  
[Output]  

```
Unable to find image 'schoolofdevops/vote:latest' locally
latest: Pulling from schoolofdevops/vote
Digest: sha256:9195942ea654fa8d8aeb37900be5619215c08e7e1bef0b7dfe4c04a9cc20a8c2
Status: Downloaded newer image for schoolofdevops/vote:latest
7d58ecc05754b5fd192c4ecceae334ac22565684c6923ea332bff5c88e3fca2b
```
Lets check the status of the container  
```
docker ps -l
```  
[Output]  

```
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                   NAMES
7d58ecc05754        schoolofdevops/vote   "gunicorn app:app -b…"   27 seconds ago      Up 26 seconds       0.0.0.0:32768->80/tcp   peaceful_varahamihira
```  

### Renaming the container  

We can rename the container by using following command  
```
docker rename 7d58ecc05754 vote
```  
[replace 7d58ecc05754 with the actual container id on your system ]

We have changed container's automatically generated name to **vote**. This new name can be of your choice. The point to understand is this command takes two arguments. The **Old_name followed by New_name**
Run docker ps command to check the effect of changes  
```
docker ps
```  
[Output]  

```
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                   NAMES
7d58ecc05754        schoolofdevops/vote   "gunicorn app:app -b…"   3 minutes ago       Up 3 minutes        0.0.0.0:32768->80/tcp   vote
```

As you can see here, the container is renamed to **vote**. This makes referencing container in cli very much easier.  

### Ready to  vote ?  

Let's see what this **vote** application does by connecting to that application. For that we need,  

  * Host machine's IP  
  * Container's port which is mapped to a host's port

Let's find out the port mapping of container to host. Docker provides subcommand called **port** which does this job  

```
docker port vote  
```  
[Output]  

```
80/tcp -> 0.0.0.0:32768
```  
So whatever traffic the host gets in port **2368** will be mapped to container's port **32768**  

Let's connect to http://IP_ADDRESS:PORT to see the actual application  




### Finding Everything about the running  container
This topic discusses about finding metadata of containers. These metadata include various parameters like,  
  * State of the container  
  * Mounts  
  * Configuration  
  * Network, etc.,  

#### Inspecting
Lets try this inspect subcommand in action  

```
docker inspect vote
```  

Data output by above command contains detailed descriptino of the container an its properties. is represented in JSON format which makes filtering these results easier.  

### Copying files between container and client host  

We can copy files/directories form host to container and vice-versa    
Let us create a file on the host  
```
touch testfile
```  

To copy the testfile **from host machine to ghsot contanier**, try  
```
docker cp testfile vote:/opt  
```  
This command will copy testfile to vote container's **/opt** directory  and will not give any output. To verify the file has been copies or not, let us log into container by running,  

```
docker exec -it vote bash
```  
Change directory into /opt and list the files  

```
cd /opt  
ls
```  

[Output]  

```
testfile
```  

There you can see that file has been successfully copied. Now exit the container  

Now you may try to cp some files **from the container to the host machine**  

```
docker cp vote:/app  .  
ls  
```  


### Checking the Stats  

Docker **stats** command returns a data stream of resource utilization used by containers. The flag **--no-stream** disables data stream and displays only first result  

```
docker stats --no-stream=true vote

docker stats
```  



### Controlling Resources  
Docker provides us the granularity to control each container's **resource utilization**. We have several commands in the inventory to achieve this  

#### Putting limits on Running Containers  
First, let us monitor the utilization

```
docker stats
```  

You can see that **Memory** attribute has **max** as its value, which  means unlimited usage of host's RAM. We can put a cap on that by using **update** command  

```
docker update --memory 400M --memory-swap -1 vote   
```  

[Output]  

```
vote
```  
Let us check whether the change has taken effect or not with docker stats terminal   

```
docker stat
```  

As you can see, the memory utilization of the container is changed from xxxx (unlimited) to 400 mb  

#### Limiting Resources while launching new containers  
The following resources can be limited using the *update* command  
  * CPU
  * Memory
  * Disk IO
  * Capabilities  

Open two terminals, lets call them T1, and T2  
In T1, start monitoring the stats  

```
docker stats
```  

[Output]  
```
CONTAINER           CPU %               MEM USAGE / LIMIT     MM %               NET I/O             BLOCK I/O             PIDS
b28efeef41f8        0.16%               190.1 MiB / 400 MiB   47.51%              1.296 kB / 648 B    86.02 kB / 45.06 kB   0
CONTAINER           CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O             PIDS
b28efeef41f8        0.01%               190.1 MiB / 400 MiB   47.51%              1.296 kB / 648 B    86.02 kB / 45.06 kB   0
```

From T2, launch containers  with different CPU shares as well as **cpus** configurations. Default CPU shares are set to 1024. This is a relative weight. Observe docker stats command after every launch to see the effect of your configurations.

```
[CPU Shares]

docker run -d --cpu-shares 1024 schoolofdevops/stresstest stress --cpu 2

docker run -d --cpu-shares 1024 schoolofdevops/stresstest stress --cpu 2

docker run -d --cpu-shares 512 schoolofdevops/stresstest stress --cpu 2

docker run -d --cpu-shares 512 schoolofdevops/stresstest stress --cpu 2

docker run -d --cpu-shares 4096 schoolofdevops/stresstest stress --cpu 2


[CPUs]

docker run -d --cpus 0.2 schoolofdevops/stresstest stress --cpu 2


```  

Close the T2 terminal once you are done with this lab.  



### Checking disk utilisation by

```

docker system df

```

[output]
```
docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              7                   5                   1.031GB             914.5MB (88%)
Containers          8                   4                   27.97MB             27.73MB (99%)
Local Volumes       3                   2                   0B                  0B
Build Cache                                                 0B                  0B
```

To prune, you could possibly use

```
docker container prune

docker system prune
```

e.g.
```
docker system prune
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all build cache
Are you sure you want to continue? [y/N]
N
```

Make sure you understand what all will be removed before using this command.


### Stopping and Removing Containers
We have learnt about interacting with a container, running a container, pausing and unpausing a container, creating and starting a container. But what if you want to stop the container or remove the container itself  

#### Stop a container  
A container can be stopped using **stop** command. This command will stop the application inside that container hence the container itself will be stopped. This command basically sends a **SIGTERM** signal to the container (graceful shutdown)  

[Command]  

```
docker stop <container_id/name>
```  

#### Kill a container  
This command will send **SIGKILL** signal and kills the container ungracefully  

[Command]  

```
docker kill <container_id/name>
```  

If you want to remove a container, then execute the following command. Before running this command, run docker ps -a to see the list of pre run containers. Choose a container of your wish and then execute docker rm command. Then run docker ps -a again to check the removed container list or not  

[Command]  

```
docker rm <container_id/name>
docker rm -f <container_id/name>

```  


#### Exercises  

### Launching Containers with Elevated  Privileges  
When the operator executes docker run --privileged, Docker will enable to access to all devices on the host as well as set some configuration in AppArmor or SELinux to allow the container nearly all the same access to the host as processes running outside containers on the host.

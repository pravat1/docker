# SETUP :whale: 

## ![](/dockericon.png)
- Spin up a VM, SSH onto the instance!
- Install Docker from script `curl -s https://get.docker.com/ | sudo sh` or refer [DockerInstallation](https://docs.docker.com/engine/installation/)
- `sudo usermod -aG docker your-user` If you would like to use *Docker* as a non-root user
- Check the version of *docker* installed `docker --version`
Running the First Container
Interactive Container
- `docker run busybox echo hello world` which pulls the busybox image from internet and runs it with single process and echo'ed `hello world`
- Run `docker ps -all` to check the status of the container
- Try running a official image from **dockerhub** (say *ubuntu*) `docker run -it ubuntu`
- -it is shorthand for `-i – t`. -i tells Docker to connect us to the container's stdin.-t tells Docker that we want a pseudo-terminal
- Check how many packages are using `dpkg -l | wc -l` inside the container
- `apt-get update` & `apt-get install figlet` 
- Type `figlet hello` to echo **HELLO** in ASCII
Non-Interactive Container
- `docker run jpetazzo/clock` runs a *container* in the background, use`^C` to stop it
- To run the container in the background `docker run -d jpetazzo/clock` `d` - detach
- To see only the last container that was started: `docker ps -l`
- To see only the ID of containers: `docker ps -q`
- `docker logs <container-id>` and can use *prefix* of the full container ID (9a17045c4433 can be run as `docker logs --tail 9a1`)
- To **STOP** a running container use `docker stop <container-id>/<prefix-of-container-id>` 
- The **STOP** & **KILL** commands can take multiple container IDs
- To start a container use `docker start <container-id> and `docker attach <container-id>` to interact with it
Inspecting the container
- Start a container `docker run -it ubuntu` 
- Run `apt-get update` & install figlet using `apt-get install figlet` & `exit`
- Run `docker diff <container-id>` to see the changes
Commit & Tagging
- The docker commit command will create a new layer with those changes, and a new image using this new layer: 
- `docker commit <yourContainerId> <newImageId>`
- `docker run -it <newImageId>` to login to container from *newImage* created
- Tag a name the image:$ docker tag <newImageId> figlet & `docker commit <containerId> figlet`
Dockerfile
- Create a *Dockerfile* with the following contents
```
FROM ubuntu
RUN apt-get update
RUN apt-get install figlet
```
- The **FROM** indicates the base image for our build
- Each **RUN** line will be executed by Docker during the build (RUN commands must be non-interactive like '-y flag to apt-get' )
- Run `docker build -t figlet .` from the location where the *Dockerfile* exists
- To view the history of the image use `docker history <image-id>`
Sending the build context to Docker
- If you want to send a file/folder or any artifacts to docker during the build, we need to place the files in the **.** directory
**CMD** and **ENTRYPOINT**
- Those commands allow us to set the default command to run in a container. It can appear at any point in the file. Each **CMD** will replace and override the previous one, so multiple CMD lines are useless.
 `figlet -f script hello` 
- `-f script` tells figlet to use a fancy font.
- `hello` is the message that we want it to display.
- So new Dockerfile will be like
```
FROM ubuntu
RUN apt-get update
RUN ["apt-get", "install", "figlet"]
CMD figlet -f script hello
```
- The build the image `docker build -t figlet .` and run it `docker run figlet`
```
FROM ubuntu
RUN apt-get update
RUN ["apt-get", "install", "figlet"]
ENTRYPOINT ["figlet", "-f", "script"]
```
- **ENTRYPOINT** defines a base command (and its parameters) for the container.
- The command line arguments are appended to those parameters.
- Like CMD, ENTRYPOINT can appear anywhere, and replaces the previous value.
Copying files during the build
- Create a C program with the following content
```
hello.c:
int main () {
puts("Hello, world!");
return 0;
}
```
- Keep this file along with the `Dockerfile` in the same location
- On Debian and Ubuntu, the package build-essential will get us a compiler
- When installing it, don't forget to specify the -y flag, otherwise the build will fail (since the build cannot be interactive)
- Then we will use **COPY** to place the source file into the container
- Create a `Dockerfile` like,
```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y build-essential
COPY hello.c /
RUN make hello
CMD /hello
```
- Test the C Program
	1. Create hello.c and Dockerfile in the same direcotry
	2. Run docker build -t hello . in this directory
	3. Run docker run hello, you should see Hello, world!
- The **MAINTAINER** instruction tells you who wrote the Dockerfile
- `MAINTAINER Full Name <email>`,It’s optional but recommended
- The RUN instruction can be specified in two ways.
- With shell wrapping, which runs the specified command inside a shell, with /bin/sh -c:
`RUN apt-get update`
- Or using the exec method, which avoids shell string expansion, and allows execution in images that
don't have /bin/sh:
`RUN [ "apt-get", "update" ]`
- It is possible to execute multiple commands in a single step `RUN apt-get update && apt-get install -y wget && apt-get clean`
- The **EXPOSE** instruction tells Docker what ports are to be published in this image
```
EXPOSE 8080
EXPOSE 80 443
EXPOSE 53/tcp 53/udp
```
- All ports are private by default.
- The Dockerfile doesn't control if a port is publicly available.
- When you docker run -p <port> ..., that port becomes public. (Even if it was not declared with **EXPOSE**.)
- When you docker run -P ... (without port number), all ports declared with EXPOSE become public.
- A public port is reachable from other containers and from outside the host
- A private port is not reachable from outside
**ADD** works almost like COPY, but has a few extra features.
**ADD** can get remote files:
`ADD http://www.example.com/webapp.jar /opt/`
- This would download the webapp.jar file and place it in the /opt directory.
- **ADD** will automatically unpack zip files and tar archives `ADD ./assets.zip /var/www/htdocs/assets/`
- This would unpack assets.zip into /var/www/htdocs/assets
- However, ADD will not automatically unpack remote archives
- The **VOLUME** instruction tells Docker that a specific directory should be a volume `VOLUME /var/lib/mysql`
- Docker Networking
- Container Network Model (CNM)
- The CNM adds the notion of a network, and a new top-level command to manipulate and see those networks: *docker network*
- `docker network ls`
- Create a network called *dev*: `docker network create dev`
- The network is now visible with the network `ls` command
```
$ docker network ls
NETWORK ID NAME DRIVER
6bde79dfcf70 bridge bridge
8d9c78725538 none null
eb0eeab782f4 host host
4clff84d6d3f dev bridge
```
- Now, create another container on this network.
- We will create a named container on this network. It will be reachable with its name, search.
- `docker run -d --name search --net dev elasticsearch`
- Now, create another container on this network. `docker run -ti --net dev alpine sh` and Login to the container 
- From this new container, we can resolve and ping the other one, using its assigned name:
```
root@0ecccdfa45ef:/#
/ # ping search
PING search (172.18.0.2) 56(84) bytes of data.
64 bytes from search.dev (172.18.0.2): icmp_seq=1 ttl=64 time=0.221 ms
64 bytes from search.dev (172.18.0.2): icmp_seq=2 ttl=64 time=0.114 ms
64 bytes from search.dev (172.18.0.2): icmp_seq=3 ttl=64 time=0.114 ms
^C
--- search ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.114/0.149/0.221/0.052 ms
root@0ecccdfa45ef:/#
```
- In Docker Engine 1.9, name resolution is implemented with /etc/hosts, and updating it each time containers are added/removed
```
[root@0ecccdfa45ef /]# cat /etc/hosts
172.18.0.3 0ecccdfa45ef
127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.18.0.2 search
172.18.0.2 search.dev
```
- In Docker Engine 1.10, this has been replaced by a dynamic resolver. (This avoids race conditions when updating /etc/hosts.)
Connecting multiple containers together

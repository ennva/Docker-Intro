## Docker Intro

### Docker Flow
* From an <b>image</b> (file made of just enough of the operating systems to do what you need to do). Take a look with `docker images`. IMAGE ID (internal representation of image in docker)
* The `docker run` command take an image and turn it into a living running <b>container</b>. Ex: `docker run -ti <REPOSITORY_name>:<image_TAG> command`, `docker run -ti ubuntu:latest bash`
	- Check the running images: `docker ps`
* The `docker commit` command take stopped container and make a new image out of him (`docker commit <container_ID>`). This allow you to save your container from your modication
	- check you stopped containers: `docker ps -a`, or check the last run `docker ps -l --format <your GO format>`
	- After running commit command, rename your new image `docker tag <hash generated> <new-name>`
	- from a command `docker ps`, you can also create a new image from a container by `docker commit <NAMES of container> <name-of-your-new-image>`
	
### Run processes in containers
* `docker run --rm -ti ubuntu sleep 5` (run a sleep command and exit after 5 seconds)
* `docker run -ti ubuntu bash -c "sleep 3; echo all done"` (run the sleep command and after 3 seconds echo a message and exit)
* `docker -d -it ubuntu bash` (-d: detash, will run in the background not interactively). You can detach with Control-q but it not exit from the container
* With `docker attach <NAMES-of-container>`, you can bring up the container from background to interact with it.
* You can attach multiple windows to the same container. On a running container you can do `docker exec -ti <NAMES-of-container> bash`

### Manage containers
* `docker logs <container_name>` keep the output of the container available. Don't make you container output really huge
* `docker kill <container_name> or <container_ID>` kill container
* `docker rm <container_name> or <container_ID>` remove container
* `docker run --memory <max-allowed-memory> <image-name> command` to limits the memory used by the container
* `docker run --cpu--shares` to cpu relative to other containers. This give a container access to a greater or lesser proportion of host machine's CPU cycles.
* `docker run --cpu-quata <quota>` to hards limit.
* MOST of ORCHESTRATION generally requires resource limiting
* Some best practices:
	- Don't let your containers fetch dependencies when they start. Make your containers include their dependencies inside the cpntainer themselves.
	- Don't leave important things in unnamed stopped containers
	
### Container Networking
* Programs in containers are isolated from the internet by default
* You can group your containers into "private" networks
* You explicitly choose who can connect to whom
* Docker offer the option of exposing a port or publishing a port. that make that port accessible from outside the machine on which Docker is being hosted
	- You may explicitly specifies the port inside the container and outside
	- exposes as many ports as you want
	- requires coordination between containers
	- Makes it easy to find the exposed ports
	- Ex: `docker run --rm -ti -p 45678:45678 -p 45679:45679 --name echo-server ubuntu:14.04 bash`. Here `-p` flag specify to publish on <internal-port-for-container>:<external-port-for-container>
	- You can use NetCat (nc command) to listen on a specific port in you localhost (`nc localhost 45678`) or on docker container (`nc host.docker.internal 45678`) or other machine (`nc <ipaddress-of-your-macchine> <port-tolisten>`)
* To expose ports dynamically, take in consideration these points:
	- The port inside the container is fixed. 
	- The port on the host is chosen from the unused ports. When running the container you can left out the host port, so docker will pick up the avilable one. This allow many containers running the same programs to share the same host. This id often used with some sort of the orchestration or discovery service like kubernetes.
	- Ex: `docker run --rm -ti -p 45678(internal-port) -p 45679(internal-port) --name echo-server ubuntu:14.04 bash`. Without specifing the host port, docker will pick one available. Check it out with `docker port echo-server`.
* Docker use tcp, udp and other protocols as well
	- `docker run -p outside-port:inside-port/protocol`. For protocol ( tcp/udp )
	- Ex `docker run -p 1234/udp --name echo-server ubuntu:14.1 bash`
	
### Container virtual networking
* Docker virtual networking
	- `docker network ls`: bridge=network used by container, host=used to not isolate the container at all and have some security concerns, none=if we want container having no networking 
	- `docker network create learning`.  This would create a new network
	- `docker run --rm -ti --net learning --name catserver ubuntu:14.1 bash`. This would put a container inside our defined network. Names are very  usefull when using private networks in Docker, because different containers inside the network can refer each other by those names,
	- You can move a container from one network to another: `docker network create catsonly` and then `docker network connect catsonly catserver` 
* Legacy Linking connecting ports
	- The linked port can only work one way/direction
	- Environment variables are still shared only one way. Restarts the container sometimes break the links
	- EX: 1- run a container: `docker run --rm -ti -e SECRET=mysecret --name catserver centos7 bash` with evironment variable SECRET
	- EX: 2- connect a container to one created: `docker run --rm -ti --link catserver --name dogserver centos7 bash`

### Docker images
* Tag your image: `docker commit <container_id> name` or `docker tag <container_name> tag_name`. Structure Organisation_name/image-name
* Cleaning images
	- `docker rmi image_id` or
	- `docker rmi image-name:tag`
	
### Volumes (sharing data between containers): it's a Virtual "disc" to store and share data
* Two main variety:
	- Persistent: the data will keep on host
	- Ephemeral: exist as long a container will using them
* There are not part of images, no part of volumes will be included when you download an image, amd no part of volumes will be involved if you upload an image. There are for local data/host
* Shared folders: `docker run -ti -v <absolute-path-on-host>:<path-on-container> centos7 bash`. Ex `docker run -ti -v /Users/dog/example:shared-folder centos7 bash`
* Share file (make sure file exist): `docker run -ti -v <absolute-path-on-host-of-file>:<path-on-container/name-of-file> centos7 bash`
* <b>volumes-from</b> is used to share data between containers. It's a Shared "discs" that exist only as long as they being used and there will be common for the containers that are using them. Ex `docker run --rm -ti --name catserver -v /shared-data centos7 bash` and `docker run --rm -ti --volumes-from catserver`

### Docker Registries (piece of software that manage and distribute images)
* You can upload images to them and download imgaes from them
* Docker company makes these freely available
* You can also run your own registries to ensure your data stays safe and private
* Find image to use `docker search image_name_to_search`. Ex `docker search ubuntu`
* Docker login to log into repositories if you have docker account: `docker login` then username/password
* After login, you can pull an image: `docker pull centos`, tag it as your own one: `docker tag centos ennva/centos` and then push to docker repository `docker push` to all the world
* Dont push images containing your password.

### Dockerfile (https://docs.docker.com/engine/reference/builder/)
* Small program to create an image
* run with `docker build -t name-of-result .`. Note . here, its specifiy the location of the Dockerfile (here in the current directory)
* When build command success, the result image will be present in your local docker registry
* Each step in dockefile is cached
* Dockerfile is not a shell script. Put the steps that you suppose to be change frequently at the end of you dockerfile

	
	

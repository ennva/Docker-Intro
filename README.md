## Docker Intro

### Docker Flow
* From an <b>image</b> (file made of just enough of the operating systems to do what you need to do). Take a look with `docker images`. IMAGE ID (internal representation of image in docker)
* The `docker run` command take an image and turn it into a living running <b>container</b>. Ex: `docker run -ti <REPOSITORY_name>:<image_TAG> command`, `docker run -ti ubuntu:latest bash`
	- Check the running images: `docker ps`
* The `docker commit` command take stopped container and make a new image out of him (`docker commit <container_ID>`). This allow you to save your container from your modication
	- check you stopped containers: `docker ps -a`, or check the last run `docker ps -l --fprmat <your GO format>`
	- After running commit command, rename your new image `docker tag <hash generated> <new-name>`
	- from a command `docker ps`, you can also create a new image from a container by `docker commit <NAMES of container> <name-of-your-new-image>`
	
### Run processes in containers
* `docker run --rm -ti ubuntu sleep 5` (run a sleep command and exit after 5 seconds)
* `docker run -ti ubuntu bash -c "sleep 3; echo all done"` (run the sleep command and after 3 seconds echo a message and exit)
* `docker -d -it ubuntu bash` (-d: detash, will run in the background not interactively). You can detach with Control-q but it not exit from the container
* With `docker attach <NAMES-of-container>`, you can bring up the container from background to interact with it.
* You can attach multiple windows to the same container. On a running container you can do `docker exec -ti <NAMES-of-container> bash`
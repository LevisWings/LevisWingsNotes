# Docker CheatSheet

### Install/Start/Stop Docker

```bash
# Install:
apt-get install docker.io -y # Ubuntu/Debian
pacman -S docker # Arch Linux
# Start
sudo systemctl start docker
# Stop
sudo systemctl stop docker
```

### Essentials

```bash
docker ps # View active containers.
docker images # View the images we have downloaded.
docker run -dit --network bridge --name <NAME> <IMAGE> # Deploy a container.
docker run --rm -it --network bridge --name <NAME> <IMAGE> # Deploy a container and delete it when we exit the session.
docker exec -it <CONTAINER NAME or CONTAINER ID> bash # Enter an interactive session.
```

### Stop and remove the container

```bash
docker ps
docker stop <CONTAINER ID or NAME> # Stop the container.
docker ps -a # It is to check if it was really stopped.
docker ps -a -q # To get the container identifier.
docker rm $(docker ps -a -q) # We delete the container.
```

### Delete image

```bash
docker rmi <IMAGE NAME> # We delete the image.
docker rmi $(docker images -q) # Delete all images.
sudo docker rmi -f $(sudo docker images -q)
```

### Others

```bash
# Build (with Dockerfile):
docker build . -t ubuntu
docker run -dit -p80:80 --network bridge --name application ubuntu
# For buffer overflows:
docker run -dit -p80:80 --network bridge --name application ubuntu --cap-add=SYS_PTRACE --security-opt seccomp=unconfined
# Others:
docker ps -a # View active containers and history.
docker start <CONTAINER ID or NAME> # docker ps -a to see the history and then choose one to go back to that container with the data, similar to a snapshot.
docker run -p 4443:443 -p 8090:80 -p 2222:22 --name <NAME> <IMAGE> # Deploy a container with some ports that communicate with the host.
docker logs <CONTAINER ID or NAME> # View the logs that the container is having.
docker logs -f <CONTAINER ID or NAME> # View the live logs that the container is having.
docker exec -dit <CONTAINER ID or NAME> <COMMAND> # Execute a command on a container in the background.
docker images --filter="dangling=true" # View stale images.
```

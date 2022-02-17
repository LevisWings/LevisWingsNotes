# Creating Dockerfiles

```
touch Dockerfile
nvim Dockerfile
```

### Example Dockerfile

```docker
FROM <IMAGE>:<TAG> # We indicate an image and its tag.
WORKDIR /opt # We place ourselves in the /opt directory
COPY . . # Copy all the files from the host to the docker container (as we changed the WORKDIR, it will be in /opt).
RUN <COMMAND> # Execute a command.
# To execute several commands:
RUN <COMMAND> \
    <COMMAND> \
    <COMMAND>
```

Then, we create the container from the Dockerfile:

```bash
docker build . -t <TAG> # The tag would be the name we want to give it.
```

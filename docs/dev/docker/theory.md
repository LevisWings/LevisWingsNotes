# 📓 Theory

## What is Docker?

Starting in 2013, Docker was introduced to solve the costly and time-consuming process of application development and service delivery. Docker employs what is currently a "hot potato" for developers: containerization, this technology separates applications into their own containers, where they share resources from, but interact with the operating system independently.

Features:

* It is extremely portable, if a computer can run Docker, it can run a Docker container. This means developers only have to write the application once for multiple devices - a big headache solved!
* It has considerably lower resource usage per container than virtual machines (VMs), i.e. RAM and CPU (we'll talk about this later)
* Allows you to set up a complex environment in a few simple steps via Dockerfiles (again, we'll talk about this later)
* It is, above all, very lucrative for a pentester, as containerization has been widely adopted in IT today.

### Basic docker architecture

#### Docker Basic Architecture

This information is from [here](https://stackoverflow.com/questions/41645665/how-containerd-compares-to-runc).

* [containerd](https://containerd.io) is a container runtime that can **manage a full container lifecycle - from image transfer/storage to container execution**, monitoring and networking.
* container-shim handles headless containers, which means that once runc initializes the containers, it exits delivering the containers to container-shim which acts as a middleman.
* [runc](https://github.com/opencontainers/runc) is a lightweight universal runtime container, which is compliant with the OCI specification. **runc is used by containerd to generate and execute containers according to the OCI specification**. It is also the repackaging of libcontainer.
* [grpc](https://www.grpc.io) is used for communication between containerd and docker-engine.
* [OCI](https://opencontainers.org) maintains the OCI specification for runtime and images. Current versions of docker support the OCI runtime and image specifications.

![](../../.gitbook/assets/docker\_engine.png)

## What are Docker containers and why are they used?

As mentioned earlier, containers share computing resources but remain isolated enough to not conflict with each other through the Docker engine. These containers do not run a full operating system, unlike a VM. Let's look at the following diagram to get a better idea:

![](../../.gitbook/assets/docker\_diagram.png)

We can see three containers running their own applications without virtualization. The three applications are isolated from each other, but use the resources of the main operating system. Whereas, compared to running these applications on virtual machines:

![](../../.gitbook/assets/docker\_diagram2.png)

"Guest Operating System" is where resources are exhausted. For example, a recommended minimum Ubuntu install size is 20gb, if you were to run this for three applications, you would need 60GB of storage. While an Ubuntu Docker image has a base size of about 180MB\~. containers can also share base images! This is extremely space efficient.

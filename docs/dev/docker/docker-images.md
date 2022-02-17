# Docker images

### See all available images

{% embed url="https://hub.docker.com/search?q=&type=image" %}

### Clone image

```bash
docker pull <IMAGE>
```

{% hint style="warning" %}
If we directly `docker run <IMAGE>`, it will download the latest version of that image and run it directly.
{% endhint %}

### Tags

Tags are used to identify different versions of an image. The syntax is as follows:

```bash
docker pull <IMAGE>:<TAG>
```

Tags can be found in Docker Hub, in the following tabs:

![](../../.gitbook/assets/docker\_postgres.png)


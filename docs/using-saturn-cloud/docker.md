# Using Docker CLI

In Saturn Cloud you are always running within a container. However sometimes it can be useful to dispatch containerized workloads using the Docker cli. This article describes the steps necessary to do so.

## Enable Docker in Docker support

Since you are running inside a Docker container, you cannot simply start the `dockerd` daemon. Instead you must enable Docker in Docker support in Saturn Cloud. This starts a sidecar docker in docker container and mounts the docker socket within your container. Scroll down to the "Additional Features" section and make sure you "Show Advanced Options".

![docker in docker](/images/docs/dind.webp "doc-image")

Unfortunately Docker in Docker support is only available with Saturn Cloud On-Premise. If you are using Saturn Cloud On-Premise and you do not see this option, please contact support@saturncloud.io to enable it.

## Make sure you can read/write to the docker socket

Due to technical limitations, the docker socket is mounted in your container owned by root. To ensure that you can access it, please use `sudo` to change it's ownership

```
$ sudo chown jovyan:jovyan /var/run/docker.sock
```

## Install the Docker CLI

```
$ sudo apt update
$ sudo apt install docker.io --yes
```

Now you can use the docker CLI

# Docker Volumes

## Problem Statement

It is a very common requirement to persist the data in a Docker container beyond the lifetime of the container. However, the file system
of a Docker container is deleted/removed when the container dies. 

## Solution

There are 2 different ways how docker solves this problem.

1. Volumes
2. Bind Directory on a host as a Mount

### Volumes 

Volumes aims to solve the same problem by providing a way to store data on the host file system, separate from the container's file system, 
so that the data can persist even if the container is deleted and recreated.

![image](https://user-images.githubusercontent.com/43399466/218018334-286d8949-d155-4d55-80bc-24827b02f9b1.png)


Volumes can be created and managed using the docker volume command. You can create a new volume using the following command:

```
docker volume create <volume_name>
```

Once a volume is created, you can mount it to a container using the -v or --mount option when running a docker run command. 

For example:

```
docker run -it -v <volume_name>:/data <image_name> /bin/bash
```

This command will mount the volume <volume_name> to the /data directory in the container. Any data written to the /data directory
inside the container will be persisted in the volume on the host file system.

### Bind Directory on a host as a Mount

Bind mounts also aims to solve the same problem but in a complete different way.

Using this way, user can mount a directory from the host file system into a container. Bind mounts have the same behavior as volumes, but
are specified using a host path instead of a volume name. 

For example, 

```
docker run -it -v <host_path>:<container_path> <image_name> /bin/bash
```

## Key Differences between Volumes and Bind Directory on a host as a Mount

Volumes are managed, created, mounted and deleted using the Docker API. However, Volumes are more flexible than bind mounts, as 
they can be managed and backed up separately from the host file system, and can be moved between containers and hosts.

In a nutshell, Bind Directory on a host as a Mount are appropriate for simple use cases where you need to mount a directory from the host file system into
a container, while volumes are better suited for more complex use cases where you need more control over the data being persisted
in the container.
Docker Volumes and Bind Mounts: Persistent Storage for Containers
This repository contains notes and examples related to Docker volumes and bind mounts, essential for persistent data storage and sharing in Docker containers.

Table of Contents
Introduction
Problems Solved
Bind Mounts vs. Volumes
Working with Docker Volumes
List Volumes
Create a Volume
Inspect a Volume
Delete a Volume
Mount a Volume to a Container
Inspect Container Mounts
Key Takeaways
Introduction
Docker containers are by default ephemeral (short-lived) in nature (1:47). This means any data written inside a container's file system is lost when the container stops or is removed. To address this, Docker provides mechanisms for persistent storage: Bind Mounts and Volumes.

Problems Solved
The video highlights three main problems that Docker volumes and bind mounts solve:

Data Persistence for Logs/User Information: Prevents loss of critical data (like application logs or user details) when a container goes down (1:37, 7:41).
Data Sharing between Containers: Enables multiple containers (e.g., front-end and back-end) to share and access common files (e.g., JSON, YAML, HTML files) (3:09, 8:15).
Reading Files from the Host: Allows containers to read files generated or stored directly on the host machine (6:10, 8:28).
Bind Mounts vs. Volumes
Docker offers two primary solutions for persistent storage:

Bind Mounts (8:51):

Allows you to "bind" a specific directory on your host machine to a specific directory inside a container.
Any changes made in either location are reflected in the other.
Directly ties the container's data to a specific path on the host, making it less portable.
Volumes (11:34):

Managed entirely by Docker, providing a better lifecycle management (11:42).
Docker creates and manages a logical storage area on the host (or external storage) that can be mounted to containers.
Advantages include:
Managed by Docker CLI (create, destroy, inspect) (13:50).
Can be stored on external storage devices (S3, NFS) (14:23).
Easier to back up and migrate (15:19).
Can be shared easily between multiple containers (16:27).
Can offer high performance I/O (17:37).
Recommendation: Volumes are generally preferred over bind mounts due to their enhanced management and portability features (21:18).
Working with Docker Volumes
Here are common Docker commands for managing volumes:

List Volumes
Lists all existing Docker volumes.

docker volume ls
Create a Volume
Creates a new Docker volume with a specified name.

docker volume create 
# Example: docker volume create my-data-volume
Inspect a Volume
Provides detailed information about a specific volume, including its mount point on the host.

docker volume inspect 
# Example: docker volume inspect my-data-volume
Delete a Volume
Removes a Docker volume. Important: You must stop and remove any containers using the volume before deleting it (33:49).

docker volume rm 
# Example: docker volume rm my-data-volume
# You can delete multiple volumes: docker volume rm volume1 volume2
Mount a Volume to a Container
When running a container, use the --mount or -v flag to attach a volume. The --mount flag is more verbose and generally recommended for readability (29:57).

Using --mount (recommended):

docker run -d --mount source=,target= 
# Example: docker run -d --mount source=my-data-volume,target=/app/data nginx:latest
# To make it read-only: docker run -d --mount source=my-data-volume,target=/app/data,readonly nginx:latest
Using -v (shorthand):

docker run -d -v : 
# Example: docker run -d -v my-data-volume:/app/data nginx:latest
Inspect Container Mounts
To see if a container has a volume mounted and its details, inspect the container.

docker inspect  | grep -A 10 "Mounts"
# This will show the 'Mounts' section in the container's configuration.
Key Takeaways
Containers are ephemeral; persistent storage is crucial.
Volumes are Docker's preferred way to manage persistent data due to their lifecycle management and flexibility.
Always stop and remove containers before deleting volumes they are using.

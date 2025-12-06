# Docker Networking

Networking allows containers to communicate with each other and with the host system. Containers run isolated from the host system
and need a way to communicate with each other and with the host system.

By default, Docker provides two network drivers for you, the bridge and the overlay drivers. 

```
docker network ls
```

```
NETWORK ID          NAME                DRIVER
xxxxxxxxxxxx        none                null
xxxxxxxxxxxx        host                host
xxxxxxxxxxxx        bridge              bridge
```


### Bridge Networking

The default network mode in Docker. It creates a private network between the host and containers, allowing
containers to communicate with each other and with the host system.

![image](https://user-images.githubusercontent.com/43399466/217745543-f40e5614-ac34-4b78-85a9-91b24512388d.png)

If you want to secure your containers and isolate them from the default bridge network you can also create your own bridge network.

```
docker network create -d bridge my_bridge
```

Now, if you list the docker networks, you will see a new network.

```
docker network ls

NETWORK ID          NAME                DRIVER
xxxxxxxxxxxx        bridge              bridge
xxxxxxxxxxxx        my_bridge           bridge
xxxxxxxxxxxx        none                null
xxxxxxxxxxxx        host                host
```

This new network can be attached to the containers, when you run these containers.

```
docker run -d --net=my_bridge --name db training/postgres
```

This way, you can run multiple containers on a single host platform where one container is attached to the default network and 
the other is attached to the my_bridge network.

These containers are completely isolated with their private networks and cannot talk to each other.

![image](https://user-images.githubusercontent.com/43399466/217748680-8beefd0a-8181-4752-a098-a905ebed5d2a.png)


However, you can at any point of time, attach the first container to my_bridge network and enable communication

```
docker network connect my_bridge web
```

![image](https://user-images.githubusercontent.com/43399466/217748726-7bb347d0-3736-4f89-bdff-31d240b15150.png)


### Host Networking

This mode allows containers to share the host system's network stack, providing direct access to the host system's network.

To attach a host network to a Docker container, you can use the --network="host" option when running a docker run command. When you use this option, the container has access to the host's network stack, and shares the host's network namespace. This means that the container will use the same IP address and network configuration as the host.

Here's an example of how to run a Docker container with the host network:

```
docker run --network="host" <image_name> <command>
```

Keep in mind that when you use the host network, the container is less isolated from the host system, and has access to all of the host's network resources. This can be a security risk, so use the host network with caution.

Additionally, not all Docker image and command combinations are compatible with the host network, so it's important to check the image documentation or run the image with the --network="bridge" option (the default network mode) first to see if there are any compatibility issues.

### Overlay Networking

This mode enables communication between containers across multiple Docker host machines, allowing containers to be connected to a single network even when they are running on different hosts.

### Macvlan Networking

This mode allows a container to appear on the network as a physical host rather than as a container.
Docker Networking Concepts
This document summarizes key concepts from the Docker Networking video, covering why networking is essential for containers, different network types, and how to achieve container communication and isolation.

Why Docker Networking?
Networking in Docker allows containers to communicate with each other and the host system.

Scenario 1: Container-to-Container Communication: Front-end containers need to talk to back-end containers. This requires a network for them to exchange data.
Scenario 2: Container Isolation: Sometimes, you need to completely isolate one container from another, especially for sensitive applications like payment systems, to enhance security.
Unlike virtual machines (VMs) which have their own complete operating systems and can be assigned to different subnets for isolation, containers are lightweight and share the host's kernel. Docker networking provides mechanisms to manage their communication and isolation.

How a Container Talks to the Host
By default, when you create a container, Docker sets up a virtual Ethernet interface called Docker 0 (also known as a veth or Bridge networking). This acts as a bridge, allowing containers to communicate with the host system and, by extension, external networks. Without this bridge, applications inside containers would not be reachable from the internet, making them unusable.

Docker Network Types
Docker offers several network drivers:

Bridge Network (Default):

This is the default network type when you create a container without specifying a network.
It creates a virtual bridge (docker0) on the host that containers connect to.
Containers on the same bridge network can communicate with each other by default.
Advantage: Simple and out-of-the-box communication for containers on the same host.
Disadvantage (for some use cases): All containers on the default bridge can communicate, which might not be secure enough for sensitive applications that require isolation.
Host Network:

Containers directly use the network stack of the host machine.
They share the host's IP address and port space.
Advantage: Excellent performance as there's no network address translation (NAT) layer.
Disadvantage: Highly insecure. If someone has access to the host, they effectively have direct access to the container's network, breaking container isolation principles. It's generally not recommended for security-sensitive applications.
Overlay Network:

More complex, typically used for Docker Swarm or Kubernetes.
Enables communication between containers running on different Docker hosts (multiple hosts forming a cluster).
Not necessary for single-host Docker deployments.
Achieving Network Isolation with Custom Bridge Networks
While the default bridge network allows all containers to communicate, Docker allows you to create custom bridge networks to achieve logical isolation.

Instead of all containers using the single docker0 bridge, you can create new, separate bridge networks.
Containers connected to different custom bridge networks are isolated from each other.
For example, you could put a sensitive "finance" container on its own custom bridge network, while a "login" container remains on the default bridge or another custom network. This breaks the common communication path and enhances security.
Practical Demonstration Commands
Here are the Linux commands demonstrated in the video to illustrate Docker networking concepts:

Run a container (login container):
docker run -d --name login nginx

Log into a container to install tools:
docker exec -it login /bin/bash
(Inside the container):
apt update
apt install -y iputils-ping
ping -V

Run another container (logout container):
docker run -d --name logout nginx

List running containers:
docker ps

Inspect container network details (e.g., IP address, network type):
docker inspect login
docker inspect logout

Ping between containers on the default bridge network:
(Inside the login container):
ping 172.17.0.3 (replace with actual IP of logout container)

List existing Docker networks:
docker network ls

Delete a custom network (if it exists):
docker network rm test

Create a custom bridge network:
docker network create secure_network

Run a container on a custom network (finance container):
docker run -d --name finance --network secure_network nginx

Verify network of the finance container:
docker inspect finance

Attempt to ping finance container from login container (should fail due to isolation):
(Inside the login container):
ping 172.19.0.2 (replace with actual IP of finance container)

Run a container using the host network:
docker run -d --name host_demo --network host nginx

Inspect the host network container:
docker inspect host_demo (Note: No custom IP address is assigned, it uses the host's IP)





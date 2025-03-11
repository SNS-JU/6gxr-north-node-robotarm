# Docker-Based ROS Container Setup Instructions

This document describes how to set up two machines (Machine A and Machine B) to run two different ROS-related containers:
- **Machine A**: Runs the Babylon-Rosbridge container
- **Machine B**: Runs the UR-rosdriver container

The two machines are connected via a direct link on their `eno2` interfaces (`192.168.10.0/24` network).  
Machine B also connects to a UR robot on `enp3s0` with IP `192.168.1.10`, and both machines have separate interfaces for internet/intranet connectivity.

---

## 1. Overview

### Machine B
- **Network Interfaces**:
  - `enp3s0`: Connected to the UR robot (Robot IP: `192.168.1.10`), Machine B IP: `192.168.1.20/24`
  - `eno2`: Connected to Machine A, Machine B IP: `192.168.10.20/24`
  - `enpUSB`: Internet/intranet connection (e.g., DHCP)

### Machine A
- **Network Interfaces**:
  - `eno2`: Connected to Machine B, Machine A IP: `192.168.10.15/24`
  - `enp3s0`: Internet/intranet connection (e.g., DHCP)

### Goal
- Machine B runs a **UR-rosdriver** container to interface with the UR robot.
- Machine A runs a **Babylon-Rosbridge** container.
- Both machines communicate with each other over the `192.168.10.x` network (via `eno2`).

---

## 2. Installing Docker

Below is an example procedure on Ubuntu.  
If Docker is already installed, you can skip this section.

1. **Update package index**:
   - sudo apt-get update

2. **Install prerequisites**:
   - sudo apt-get install -y \
       apt-transport-https \
       ca-certificates \
       curl \
       software-properties-common

3. **Add Docker’s official GPG key**:
   - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

4. **Add the Docker repository** (example for Ubuntu 20.04 "focal"; adjust if you use another version):
   - sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

5. **Update and install Docker**:
   - sudo apt-get update
   - sudo apt-get install -y docker-ce docker-ce-cli containerd.io

6. **Verify Docker** (optional):
   - docker --version

### Allow Non-Root User to Use Docker

- sudo groupadd docker    (Create `docker` group if it doesn't exist)
- sudo usermod -aG docker remoteuser   (Add your user to the `docker` group)
- Log out and log back in to apply group changes

---

## 3. Machine Networking

Ensure the physical cables and IP addresses are set up so that:
- **Machine B** has:
  - `enp3s0`: `192.168.1.20/24` to communicate with UR robot (`192.168.1.10`).
  - `eno2`: `192.168.10.20/24` to communicate with Machine A.
  - `enpUSB`: Internet or intranet (DHCP or static).
- **Machine A** has:
  - `eno2`: `192.168.10.15/24` to communicate with Machine B.
  - `enp3s0`: Internet or intranet (DHCP or static).

You can verify with `ip a` to see if the addresses match your desired network plan.

---

## 4. Docker Networks

### 4.1 Bridge Network Example

A typical Docker network is the default bridge driver. You can create a custom one:

- docker network create \
  --driver bridge \
  --subnet=172.18.0.0/24 \
  --gateway=172.18.0.1 \
  ur_network

Any container attached to `ur_network` will get an IP in `172.18.0.x`.  
In many cases, simply using Docker’s default `bridge` network is sufficient.

### 4.2 Macvlan Network Example

If you want your container to appear on the same physical network as the host with its own IP address, use macvlan:

- docker network create -d macvlan \
  --subnet=192.168.10.0/24 \
  --gateway=192.168.10.1 \
  -o parent=eno2 \
  ur_network_macvlan

This way, the container can directly obtain an IP on `192.168.10.x`.  
However, macvlan setups may require additional care regarding routing and ARP for direct host-container communication.  
For simple setups, a `bridge` network can be easier to manage.

---

## 5. Launching the Containers

Below are generic examples of how to run the containers on each machine.  
Replace `<YOUR_IMAGE>` with the actual image name or tag you use.

### 5.1 Machine B: UR-rosdriver Container

1. Pull an existing UR-rosdriver image (example):
   - docker pull <YOUR_UR_ROSDRIVER_IMAGE>

2. Run the container, attaching to a chosen network:
   - docker run -d \
       --name ur-rosdriver \
       --network ur_network \
       --privileged \
       <YOUR_UR_ROSDRIVER_IMAGE>

   - `--network ur_network` joins the custom bridge `ur_network` (or macvlan if you prefer).
   - `--privileged` may be necessary if ROS drivers need direct device access.

   If you want the container to directly communicate on `192.168.1.x` with the robot, you could attach it to a macvlan network configured on `enp3s0` instead. Otherwise, it can communicate via the host’s IP routing.

### 5.2 Machine A: Babylon-Rosbridge Container

1. Pull the image:
   - docker pull <YOUR_BABYLON_ROSBRIDGE_IMAGE>

2. Run the container:
   - docker run -d \
       --name babylon-rosbridge \
       --network ur_network \
       <YOUR_BABYLON_ROSBRIDGE_IMAGE>

Machine A’s container is also placed on the same Docker network. Communication between the two containers can happen either:
- Within the same Docker network if both are on the same machine (not the case here), or
- Via the host networking if each container is on a separate machine, with IP routing between `192.168.10.15` and `192.168.10.20`.

---

## 6. Validation & Logs

Check if the containers are running:

- docker ps        (on Machine B)
- docker ps        (on Machine A)

Look for containers named `ur-rosdriver` and `babylon-rosbridge` with status `Up ...`.

### Viewing Logs or Shell Access

- docker logs ur-rosdriver
- docker logs babylon-rosbridge

- docker exec -it ur-rosdriver bash
- docker exec -it babylon-rosbridge bash

---

## 7. References

- [Docker Documentation](https://docs.docker.com/)
- [Docker Network Overview](https://docs.docker.com/network/)
- [Macvlan Driver Overview](https://docs.docker.com/network/macvlan/)

---

## 8. Summary

1. **Install Docker** on both machines and configure user access.  
2. **Set up network interfaces** so Machine A (`192.168.10.15`) and Machine B (`192.168.10.20`) can communicate over `192.168.10.x`, while Machine B also connects to the UR robot on `192.168.1.x`.  
3. **Create or use a Docker network** (bridge or macvlan) that suits your needs.  
4. **Launch the UR-rosdriver container** on Machine B, connecting to the UR robot network.  
5. **Launch the Babylon-Rosbridge container** on Machine A.  
6. **Verify** container status and connectivity (logs, network pings, ROS topic checks, etc.).  

With this setup, your containers on Machine A and Machine B can communicate over the configured network, while Machine B also interfaces with the UR robot.

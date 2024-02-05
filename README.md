# Docker for ROS
## Overview
Repository with a boilerplate layout for using ROS with Docker, including examples for various network setups and external hardware.

## Introduction
This README is a summation of the key notes I picked up from a few other sources specified below. It's primarily as a reminder for myself in future but may be of some use to others starting out with Docker and ROS.

## Installation
### Install and Setup Docker
Follow the [Docker install guide](https://docs.docker.com/engine/install/ubuntu/).

Allow permission for Docker to run without the sudo command being issued.
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

Confirm your setup.
```bash
docker run hello-world
```

Or trial Ubuntu 20.04 focal in an interactive session.
```bash
docker run -it --name ubuntu_test ubuntu:focal
```

### Clone this Repository
```bash
git clone https://github.com/MShields1986/docker_ros.git
```

## Usage
### Folder Structure
The repository root directory contains two directories...
- `docker` contains the template Dockerfiles and docker-compose files for the various setups
- `src` is provided as a placeholder for the ROS packages and will be mounted as a volume inside the catkin_ws/src for some of the examples
- `systemd` holds example systemd service files and a deploy script
- `TODO: rosinstall file?`

### Self-Contained Network - Docker *"bridge"* Network
This example is a slight modification of the [ROS Docker tutorials](https://wiki.ros.org/docker/Tutorials/Network) and provides a ROS master, publisher and subscriber each launched in a separate Docker container, launched from a single docker compose file.

Navigate to the Docker compose files...
```bash
cd docker_ros/docker
```

Create the Docker network and launch the containers...
```bash
docker compose -f docker-compose-network-self-contained.yml up --remove-orphans
```

You ought to see the "Hello World!" output from the subscriber-1 container.
```
[+] Running 3/0
 ✔ Container docker-rosmaster-1   Created                                                                                                                                                                0.0s 
 ✔ Container docker-publisher-1   Created                                                                                                                                                                0.0s 
 ✔ Container docker-subscriber-1  Created                                                                                                                                                                0.0s 
Attaching to publisher-1, rosmaster-1, subscriber-1
publisher-1   | ERROR: Unable to communicate with master!
publisher-1 exited with code 0
subscriber-1  | data: "Hello World!"
subscriber-1  | ---
subscriber-1  | data: "Hello World!"
subscriber-1  | ---
subscriber-1  | data: "Hello World!"
```

If you open another terminal and run some basic commands looking for a ROS master...
```bash
rostopic list
```

...you should be met with...
```
ERROR: Unable to communicate with master!
```

This is because we don't have a link between out host system's network and the Docker network we've created in this example.

If we take a look over this Docker compose file...
```bash
cat docker-compose-network-self-contained.yml
```

```yaml
version: '2'

networks:
  ros_test_bridge_network:
    driver: bridge

services:
  rosmaster:
    image: ros:noetic-robot
    environment:
      - "ROS_HOSTNAME=rosmaster"
    command: stdbuf -o L roscore
    networks:
      - ros_test_bridge_network
    restart: always

  publisher:
    image: ros:noetic-robot
    depends_on:
      - rosmaster
    environment:
      - "ROS_MASTER_URI=http://rosmaster:11311"
      - "ROS_HOSTNAME=publisher"
    command: stdbuf -o L rostopic pub /chatter std_msgs/String "Hello World!" -r 2
    networks:
      - ros_test_bridge_network
    restart: always

  subscriber:
    image: ros:noetic-robot
    depends_on:
      - rosmaster
      - publisher
    environment:
      - "ROS_HOSTNAME=subscriber"
      - "ROS_MASTER_URI=http://rosmaster:11311"
    command: stdbuf -o L rostopic echo /chatter
    networks:
      - ros_test_bridge_network
    restart: always
```

You can see we are starting a bridge network and three services, **rosmaster**, **publisher** and **subscriber**. Each service is created using the offical ROS Noetic base Docker image. We can specify dependencies between containers. Note that we set our ROS network environment varibales as detailed in the [ROS network setup guide](https://wiki.ros.org/ROS/NetworkSetup). Each service is spawned connected to a specified Docker network.

### Exposed Network - Docker *"host"* Network
Navigate to the Docker compose files...
```bash
cd docker_ros/docker
```

Launch the containers, having them connect to the Docker *"host"* network...
```bash
docker compose -f docker-compose-network-host.yml up --remove-orphans
```

Similar to the previous example you ought to see the "Hello World!" output from the subscriber-1 container.
```
[+] Running 3/0
 ✔ Container docker-rosmaster-1   Created                                                                                                                                                                0.0s 
 ✔ Container docker-publisher-1   Created                                                                                                                                                                0.0s 
 ✔ Container docker-subscriber-1  Created                                                                                                                                                                0.0s 
Attaching to publisher-1, rosmaster-1, subscriber-1
publisher-1   | ERROR: Unable to communicate with master!
subscriber-1  | ERROR: Unable to communicate with master!
publisher-1 exited with code 1
subscriber-1 exited with code 1
subscriber-1  | data: "Hello World!"
subscriber-1  | ---
subscriber-1  | data: "Hello World!"
subscriber-1  | ---
subscriber-1  | data: "Hello World!"
```

However, this time if you open another terminal and run some basic commands looking for a ROS master...
```bash
rostopic list
```

We now get interaction as if we were running the nodes natively on our host system...
```
/chatter
/rosout
/rosout_agg
```

We can echo the `/chatter` topic..
```bash
rostopic echo /chatter
```

...and publish to it
```bash
rostopic pub -r 2 /chatter std_msgs/String "Hello from your host system!"
```

...which you can confirm by switching back to the terminal where the Docker compose is running...
```
subscriber-1  | data: "Hello World!"
subscriber-1  | ---
subscriber-1  | data: "Hello from your host system!"
subscriber-1  | ---
subscriber-1  | data: "Hello World!"
subscriber-1  | ---
subscriber-1  | data: "Hello from your host system!"
```

You should take a look at the Docker compose file again to see what's changed here to enable this.
```bash
cat docker-compose-network-host.yml
```

A big change here is that we are using an external .env file to specify our environment variables.
```bash
cat .env
```

```
CATKIN_WORKSPACE_DIR="/catkin_ws"

ROS_MASTER_URI="http://localhost:11311"
ROS_HOSTNAME="localhost"
ROS_IP="172.31.1.10"

ETC_HOSTNAME_1=""
ETC_HOST_IP_1=""
```

### Exposed Network ROS Master Running on Host Sytem - Docker *"host"* Network
Navigate to the Docker compose files...
```bash
cd docker_ros/docker
```

Launch the containers, having them connect to the Docker *"host"* network...
```bash
docker compose -f docker-compose-network-host-no-master.yml up --remove-orphans
```

Should yield the rather disappointing...
```
[+] Running 2/0
 ✔ Container docker-publisher-1   Created                                                                                                                                                                0.0s 
 ✔ Container docker-subscriber-1  Created                                                                                                                                                                0.0s 
Attaching to publisher-1, subscriber-1
publisher-1   | ERROR: Unable to communicate with master!
subscriber-1  | ERROR: Unable to communicate with master!
publisher-1 exited with code 1
subscriber-1 exited with code 1
publisher-1   | ERROR: Unable to communicate with master!
subscriber-1  | ERROR: Unable to communicate with master!
```

You'll see that we have only launched two services here, we are missing a ROS master but as the network setup is the same as the previous example the intention here is to run a ROS master on the host machine prior to launch the Docker containers.

Let's try that, open another terminal and run a ROS master...
```bash
roscore
```

Then back to the previous terminal and launch the containers...
```bash
docker compose -f docker-compose-network-host-no-master.yml up --remove-orphans
```

This time you ought to see...
```
[+] Running 2/0
 ✔ Container docker-publisher-1   Created                                                                                                                                                                0.0s 
 ✔ Container docker-subscriber-1  Created                                                                                                                                                                0.0s 
Attaching to publisher-1, subscriber-1
subscriber-1  | data: "Hello World!"
subscriber-1  | ---
subscriber-1  | data: "Hello World!"
```

In a third terminal you can interact with the ROS network as we did in the previous example, subscribing and publishing to and from the containers and host machine.

Again, take a look at the Docker compose file, which is using the same .env file as before...
```bash
cat docker-compose-network-host-no-master.yml
```

### Using External Hardware
`TODO`

### Using GUIs
`TODO`

### Launching on Boot With systemd
`TODO`

### Notes on Further Implementations
If you are using `roslaunch` and want to be sure that your system holds until the ROS master is up on first start up be sure to add the `--wait` option to the `roslaunch` command.

## Bugs, Issues and Feature Requests
Please report bugs, issues and request features using the [Issue Tracker](https://github.com/MShields1986/docker_ros/issues).

## Sources
Most of what can be found here is derived from the following resources:
- Tobit Flatscher's more indepth explanations in their repos for [robotics](https://github.com/2b-t/docker-for-robotics) and [real-time](https://github.com/2b-t/docker-realtime) Docker use cases
- Ruffin White and Henrik Christensen's [ROS and Docker white paper](https://www.researchgate.net/publication/317751755_ROS_and_Docker)
- The [Docker documentation](https://docs.docker.com/), especially around networking
- [This gist](https://gist.github.com/mosquito/b23e1c1e5723a7fd9e6568e5cf91180f) from Mosquito on using Docker compose from systemd

## Useful Commands
When I was trying out Docker for the first time (again...) I ran into multiple issues that were often down to network name conflicts and networks/containers being up when I thought they were down. I found the following commands usefull.

Show all running containers...
```bash
docker ps -a
```

Stop all running containers...
```bash
docker stop $(docker ps -a -q)
```

Show all active Docker networks...
```bash
docker network ls
```

Clean inactive Docker networks...
```bash
docker network prune
```

Clean up Docker runtime but keep tagged images...
```bash
docker system prune
```

Clean up Docker runtime including images, except for anything currently being used...
```bash
docker system prune -a
```

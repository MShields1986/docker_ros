# Docker for ROS
Repository with a boilerplate layout for using ROS with Docker, including examples for various network setups and external hardware.

## Introduction
`TODO`

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
This example is taken from the ROS tutorials and provides a ROS master, publisher and subscriber each launched in a separate Docker container, launched from a single docker compose file.

Navigate to the Docker compose files...
```bash
cd docker_ros/docker
```

Create the Docker network and launch the containers...
```bash
docker compose -f docker-compose-network-self-contained.yml up --remove-orphans
```



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

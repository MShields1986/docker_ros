# Docker for ROS
Repository with a boilerplate layout for using ROS with Docker, including examples for various network setups and external hardware.

## Introduction
Most of what can be found here is derived from the following resources:
- Tobit Flatscher's more indepth explanations in their repos for [robotics](https://github.com/2b-t/docker-for-robotics) and [real-time](https://github.com/2b-t/docker-realtime) Docker use cases
- Ruffin White and Henrik Christensen's [ROS and Docker white paper](https://www.researchgate.net/publication/317751755_ROS_and_Docker)
- The [Docker documentation](https://docs.docker.com/), especially around networking
- [This gist](https://gist.github.com/mosquito/b23e1c1e5723a7fd9e6568e5cf91180f) from Mosquito on using Docker compose from systemd

When I was trying out Docker for the first time I ran into multiple issues that were often down to network name conflicts and networks/containers being up when I thought they were down. I found the following commands usefull.

Stop all running containers...
```bash
docker stop $(docker ps -a -q)
```

Clean up Docker runtime but keep tagged images...
```bash
docker system prune
```

Clean up Docker runtime including images, except for anything currently being used...
```bash
docker system prune -a
```

## Installation


## Usage


## Bugs, Issues and Feature Requests
Please report bugs, issues and request features using the [Issue Tracker](https://github.com/MShields1986/docker_ros/issues).

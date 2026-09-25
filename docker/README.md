# Docker Environment

This directory contains the Docker configuration used to create and manage simulated Linux servers for the Ansible Server Usage Monitor project.

## Architecture

WSL2 Ubuntu acts as the Ansible controller. Docker Compose runs three simulated Linux servers that Ansible can connect to over SSH.

```text
WSL2 Ubuntu
(Ansible Controller)
        |
        | SSH
        |
        +--------> server1
        |           localhost:2221 -> container:22
        |
        +--------> server2
        |           localhost:2222 -> container:22
        |
        +--------> server3
                    localhost:2223 -> container:22
```

Each simulated server provides:

- Ubuntu 24.04-based Linux environment
- SSH server for Ansible connectivity
- `ansible` user for the lab
- `/data` directory for filesystem usage monitoring
- Connectivity through a shared Docker bridge network

The project also includes a Mailpit container as a local SMTP test server. It is used by the Ansible reporting workflow and is intentionally kept separate from the `monitor-network`.

## Docker Components

The environment is built using the following components:

```text
Dockerfile
    |
    | docker build
    v
Docker Image
    |
    | Docker Compose
    v
Three Containers
    |
    +--> server1
    +--> server2
    +--> server3

Docker Compose
    |
    +--> mailpit
         SMTP: 1025
         Web UI: 8025
```

### Dockerfile

`Dockerfile` defines the reusable Ubuntu-based server image. It:

- Uses `ubuntu:24.04` as the base image
- Installs `openssh-server` and `sudo`
- Creates the `ansible` user
- Adds the `ansible` user to the `sudo` group
- Creates `/var/run/sshd` and `/data`
- Enables SSH password authentication
- Exposes container port `22`
- Runs `sshd` in the foreground as the container's main process

### Docker Image

The Dockerfile is built into the reusable image:

```text
ansible-monitor-server:latest
```

Build the image from the project root with:

```bash
docker build -t ansible-monitor-server ./docker
```

The current image is a Linux `amd64` image.

### Docker Compose

`compose.yml` defines and manages the three simulated servers using the same Docker image.

The containers use the following SSH port mappings:

| Server | Host Port | Container Port |
|--------|-----------|----------------|
| server1 | 2221 | 22 |
| server2 | 2222 | 22 |
| server3 | 2223 | 22 |

The three simulated servers are connected to the Compose-created `docker_monitor-network` bridge network.

The published host ports are used by the WSL2 Ansible controller to connect to each container through `localhost`. The Docker bridge network provides connectivity between the containers themselves, but the Ansible inventory does not use the containers' Docker-assigned IP addresses.

### Mailpit

Mailpit provides a local SMTP test endpoint for validating the project's email reporting workflow.

| Service | Host Port | Container Port |
|---------|-----------|----------------|
| SMTP | 1025 | 1025 |
| Web UI | 8025 | 8025 |

The Mailpit service is not attached to `monitor-network`. It is therefore on the Compose default network instead of the network used by the simulated servers.

Mailpit is a disposable local test sink for development and validation. It is not intended to represent a production mail server or production email storage system.

## `/data` Directory

Each simulated server contains a `/data` directory.

At the current stage, `/data` is a directory inside the container and is backed by the container's writable filesystem layer. It is not configured as a separate Docker volume.

For example:

```bash
docker exec server1 sh -c 'ls -ld /data && df -Th /data'
```

The `/data` filesystem will be the target of the Ansible monitoring automation, which will collect filesystem usage information from each server.

## Running the Environment

From the project root, start the environment with:

```bash
docker compose -f docker/compose.yml up -d
```

Check the running containers with:

```bash
docker compose -f docker/compose.yml ps
```

Stop the environment with:

```bash
docker compose -f docker/compose.yml down
```

If the Docker environment is stopped or the host environment is shut down, the containers can be started again using the `up` command above. The project does not configure a Docker restart policy for these lab containers.

## Connectivity Validation

The Docker environment has been validated from the WSL2 Ansible controller.

SSH connectivity was verified to all three simulated servers using their published host ports:

```bash
ssh -p 2221 ansible@localhost
ssh -p 2222 ansible@localhost
ssh -p 2223 ansible@localhost
```

The `ansible` user exists inside the server image and is a member of the `sudo` group.

The `/data` directory can be verified with:

```bash
docker exec server1 sh -c 'ls -ld /data && df -Th /data'
```

Ansible connectivity was subsequently verified using the project inventory and the `ping` module. All three servers returned `SUCCESS` with `pong`.

## Network Design

The three simulated servers use the same Docker bridge network:

```text
docker_monitor-network
        |
        +---- server1
        |
        +---- server2
        |
        +---- server3
```

The current containers have Docker-assigned IP addresses on this network. These addresses are not used by the Ansible inventory; Ansible connects through the published localhost SSH ports instead.

Mailpit uses the Compose default network and does not need to communicate directly with the simulated servers. The Ansible controller connects to Mailpit through its published SMTP port.

## Current Status

The Docker environment required for the current project scope is complete and provides the infrastructure required for the Ansible automation stage.

```text
Docker environment       Complete
Ansible inventory        Complete
Ansible connectivity     Complete
/data monitoring         Complete
Reporting                Complete
Mail automation          Complete
```

Future Docker changes will be made only when required by later project milestones.


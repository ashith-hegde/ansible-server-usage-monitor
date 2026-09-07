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

- Ubuntu-based Linux environment
- SSH server for Ansible connectivity
- `ansible` user for the lab
- `/data` directory for filesystem usage monitoring
- Connectivity through a shared Docker bridge network

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
```

### Dockerfile

`Dockerfile` defines the reusable Ubuntu-based server image. It installs the required SSH and sudo packages, creates the `ansible` user, creates the `/data` directory, and configures the container to run the SSH server.

### Docker Image

The Dockerfile is built into the reusable image:

```text
ansible-monitor-server:latest
```

### Docker Compose

`compose.yml` defines and manages the three simulated servers using the same Docker image.

The containers use the following SSH port mappings:

| Server | Host Port | Container Port |
|--------|-----------|----------------|
| server1 | 2221 | 22 |
| server2 | 2222 | 22 |
| server3 | 2223 | 22 |

All three containers are connected to the same Docker bridge network.

## `/data` Directory

Each simulated server contains a `/data` directory.

At the current stage, `/data` is a directory inside the container and is backed by the container's writable filesystem layer. It is not configured as a separate Docker volume.

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

## Connectivity Validation

The Docker environment has been validated from the WSL2 Ansible controller.

SSH connectivity was verified to all three simulated servers using their published host ports:

```bash
ssh -p 2221 ansible@localhost
ssh -p 2222 ansible@localhost
ssh -p 2223 ansible@localhost
```

The `/data` directory was also verified on the containers.

Ansible connectivity was subsequently verified using the project inventory and the `ping` module. All three servers returned `SUCCESS` with `pong`.

## Current Status

The Docker environment is complete and provides the infrastructure required for the Ansible automation stage.

```text
Docker environment       Complete
Ansible inventory        Complete
Ansible connectivity     Complete
/data monitoring         Next
Reporting                Planned
Mail automation          Planned
```

Future Docker changes will be made only when required by later project milestones.


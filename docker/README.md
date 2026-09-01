# Docker Environment

This directory contains the Docker configuration used to create simulated Linux servers for the Ansible Server Usage Monitor project.

## Initial Design

The project will initially simulate multiple Linux servers using Docker containers.

```text
WSL2 Ubuntu
(Ansible Controller)
        |
        | SSH
        |
        +--------> server1
        |
        +--------> server2
        |
        +--------> server3
                     |
                     v
                   /data
```

Each simulated server is intended to have:

- A Linux/Ubuntu-based environment
- A unique hostname
- SSH access for Ansible
- Its own `/data` filesystem
- Network connectivity with the other project components

The Docker environment will be implemented using Docker Compose so that the simulated servers can be created and managed as a reproducible multi-container environment.

## Planned Implementation

The Docker environment will eventually contain:

```text
docker/
├── README.md
├── Dockerfile
└── compose.yml
```

The exact Docker configuration will be finalized during implementation.

The immediate objective is to create a reliable simulated server environment that Ansible can connect to and manage.


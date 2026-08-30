# Ansible Server Usage Monitor

## Project Overview

A hands-on DevOps automation project that uses Ansible to collect `/data` filesystem usage information from multiple simulated servers.

The servers will initially be simulated using Docker containers.

## Project Goal

The goal is to automate the collection and reporting of filesystem information from multiple servers.

The project will eventually:

1. Provision multiple simulated servers using Docker.
2. Configure Ansible to communicate with the servers.
3. Collect `/data` filesystem information.
4. Determine filesystem availability and utilization.
5. Generate a structured report.
6. Send the report through a simulated mail server.
7. Use Git branches and development workflows to simulate a realistic engineering environment.

## Planned Architecture

```text
Docker Containers
       |
       v
    Ansible
       |
       v
Collect /data information
       |
       v
Generate Report
       |
       v
Simulated Mail Server
       |
       v
Email Report
```

## Initial Architecture

The project uses WSL2 Ubuntu as the Ansible controller and Docker containers as simulated Linux servers.

```text
Windows
   |
   v
WSL2 Ubuntu
(Ansible Controller)
   |
   | SSH
   |
   +--------> server1 (Docker)
   |
   +--------> server2 (Docker)
   |
   +--------> server3 (Docker)
                    |
                    v
                  /data
```

The Docker containers will simulate separate Linux servers that Ansible can manage.

Each simulated server will have:

- A unique hostname
- SSH access for Ansible
- Its own `/data` filesystem
- Network connectivity to the Ansible controller

## Technologies

- Linux
- Ansible
- Docker
- Git
- GitHub
- YAML
- Bash

## Project Status

### Milestone 1 — Project Foundation

- [x] GitHub repository created
- [x] Local Git repository initialized
- [x] `main` branch created
- [x] `develop` branch created
- [x] Initial README created

### Milestone 2 — Docker Environment

- [ ] Create simulated servers
- [ ] Configure server filesystems
- [ ] Establish networking
- [ ] Verify container connectivity

### Milestone 3 — Ansible

- [ ] Create Ansible inventory
- [ ] Configure Ansible connectivity
- [ ] Test connectivity with `ansible.builtin.ping`
- [ ] Collect `/data` information
- [ ] Process the collected information

### Milestone 4 — Reporting

- [ ] Generate structured report
- [ ] Add timestamp
- [ ] Handle unavailable servers
- [ ] Format output for email

### Milestone 5 — Mail Automation

- [ ] Deploy simulated mail server
- [ ] Configure mail delivery
- [ ] Send generated report
- [ ] Test end-to-end automation

## Repository Structure

The repository structure will evolve as the project develops.

```text
ansible-server-usage-monitor/
├── ansible/
├── docker/
├── inventory/
├── README.md
└── .gitignore
```

## Future Enhancements

Potential future improvements include:

- HTML email reports
- Scheduled execution
- Logging
- Error handling
- Server health checks
- Historical usage data
- Alerting based on disk utilization


# Ansible Configuration

This directory contains the Ansible configuration used to manage the simulated Linux servers for the Ansible Server Usage Monitor project.

## Architecture

WSL2 Ubuntu acts as the Ansible controller, while the Docker containers act as managed servers.

```text
WSL2 Ubuntu
(Ansible Controller)
        |
        | SSH
        |
        +--------> server1
        |           localhost:2221
        |
        +--------> server2
        |           localhost:2222
        |
        +--------> server3
                    localhost:2223
```

## Inventory

The inventory is defined in `inventory.ini`.

The three servers belong to the `servers` group:

```ini
[servers]
server1 ansible_port=2221
server2 ansible_port=2222
server3 ansible_port=2223

[servers:vars]
ansible_host=localhost
ansible_user=ansible
ansible_password=ansible
```

Common connection variables are defined at the group level, while the SSH port is defined per host because each Docker container is exposed through a different host port.

> **Note:** The username and password above are intentionally simple lab credentials. Production environments should use secure authentication mechanisms such as SSH keys and/or Ansible Vault rather than storing plaintext credentials in an inventory.

## Connectivity Verification

Ansible connectivity was verified against all three managed servers using the `ping` module:

```bash
ansible servers -i ansible/inventory.ini -m ping
```

All three hosts returned:

```text
SUCCESS
"ping": "pong"
```

The Python interpreter was automatically discovered on the managed containers.

## Planned Ansible Components

As the project develops, this directory will contain the Ansible automation used to:

- Collect `/data` filesystem usage from the managed servers
- Generate structured usage information
- Produce a report
- Send the report through the simulated mail server
- Add error handling and other operational improvements

The Ansible implementation will be expanded incrementally as each project milestone is completed.


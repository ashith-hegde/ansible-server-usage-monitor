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

The Ansible configuration uses the Docker-published SSH ports to connect to each managed server.

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

## Automation

The main automation is implemented in:

```text
playbooks/
└── collect_usage.yml
```

The playbook runs against the `servers` group and performs the following workflow:

```text
Managed Servers
       |
       | df -Th /data
       v
Collect filesystem information
       |
       | set_fact
       v
Store server-specific information
       |
       | Jinja2 template
       v
Generate consolidated report
       |
       | community.general.mail
       v
Send report through Mailpit
```

### Filesystem Collection

The playbook runs:

```bash
df -Th /data
```

on each managed server.

The command result is captured using Ansible's `register` keyword:

```yaml
register: data_usage
```

The registered result is then used by `ansible.builtin.set_fact` to extract the required filesystem values.

The stored information includes:

- Server name
- Total filesystem size
- Used space
- Available space
- Utilization percentage

The collection task is read-only and is configured with:

```yaml
changed_when: false
```

### Report Generation

The report is generated using:

```text
templates/usage_report.txt.j2
```

The template uses the information stored for each managed server to produce one consolidated filesystem usage report.

The Jinja2 template accesses the filesystem information for each managed host through Ansible's `hostvars` dictionary. This allows the controller-side template to combine data collected from all three servers into a single report.

Report generation uses:

```yaml
delegate_to: localhost
run_once: true
```

`delegate_to: localhost` executes the report-generation task on the Ansible controller rather than on the managed servers. `run_once: true` ensures that the task runs only once, producing one consolidated report instead of a separate report for each server.

The generated report is stored under the project's `reports/` directory with a timestamped filename.

### Usage Thresholds

The report evaluates filesystem utilization using the following thresholds:

```text
OK        <= 80%
WARNING   > 80% and <=90%
CRITICAL  > 90%
```

Each server receives a status in the report, and an overall status is calculated for the complete report.

A `CRITICAL` status takes precedence over `WARNING` when determining the overall status.

### Email Reporting

The playbook uses the `community.general.mail` module to send the generated report through the local Mailpit SMTP service.

The current lab configuration uses:

```text
SMTP host: localhost
SMTP port: 1025
```

The report is included both in the email body and as an attachment.

Email delivery is performed on the Ansible controller using `delegate_to: localhost` and `run_once: true`.

Mailpit is used only as a local SMTP test sink for this project. A production implementation would use an appropriate organizational SMTP relay or mail service.

## Running the Automation

From the project root, run:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/collect_usage.yml
```

The playbook requires the three Docker-based managed servers to be running and reachable through their configured SSH ports.

## Connectivity Verification

Ansible connectivity was verified against all three managed servers using the `ping` module:

```bash
ansible servers -i ansible/inventory.ini -m ping
```

All three hosts have been successfully validated and returned:

```text
server1 | SUCCESS
server2 | SUCCESS
server3 | SUCCESS

"ping": "pong"
```

The inventory can also be inspected with:

```bash
ansible-inventory -i ansible/inventory.ini --graph
```

and individual host variables can be checked with:

```bash
ansible-inventory -i ansible/inventory.ini --host server1
```

Ansible automatically discovered `/usr/bin/python3.12` as the Python interpreter on the managed containers during validation. The discovery warning does not prevent the automation from running successfully.

## Ansible Environment

The current controller environment was validated with:

```text
Ansible Core: 2.20.1
Python:       3.14.4
Jinja2:       3.1.6
```

No project-specific `ansible.cfg` is currently defined; Ansible reports `config file = None` in the current controller environment.

## Current Status

The Ansible configuration and automation for the current project scope are complete.

```text
Inventory configuration       Complete
Server connectivity           Complete
/data collection              Complete
Filesystem report generation  Complete
Usage thresholds              Complete
Email reporting               Complete
```

Future Ansible changes will be made only when required by later project milestones.


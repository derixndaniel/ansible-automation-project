# Ansible Server Configuration Automation

A hands-on DevOps project demonstrating automated Linux server configuration management using **Ansible**, with a focus on SSH-based automation, user provisioning, Nginx deployment, firewall configuration, Jinja2 templates, handlers, roles, and idempotency.

## Project Overview

This project automates the configuration of two CentOS Linux servers from a centralized Ansible controller.

Instead of manually connecting to each server to install software, configure services, and deploy application files, Ansible uses reusable playbooks and roles to ensure consistent server configurations.

The project includes two main automation workflows:

1. **Server preparation:** Create a dedicated Ansible user, configure SSH key authentication, and provide passwordless sudo privileges.
2. **Web server configuration:** Install Nginx, manage its service, configure HTTP firewall access, and deploy a dynamic webpage using an Ansible role.

## Technologies Used

| Technology | Purpose |
|---|---|
| Ansible | Infrastructure configuration automation |
| CentOS Linux | Controller and managed server operating system |
| SSH | Agentless communication and authentication |
| YAML | Ansible playbooks and task definitions |
| Nginx | Web server |
| firewalld | HTTP firewall configuration |
| Jinja2 | Dynamic HTML template rendering |
| Ansible Roles | Reusable configuration management |
| Git & GitHub | Version control and project documentation |

## Architecture

```text
                  Ansible Controller
                         |
                  SSH Authentication
                         |
               Ansible Inventory
                         |
                +--------+--------+
                |                 |
          Managed Node 1    Managed Node 2
              CentOS            CentOS
                |                 |
             Nginx             Nginx
                |                 |
           HTTP Port 80      HTTP Port 80
                |                 |
          Dynamic HTML      Dynamic HTML
```

The controller executes playbooks against the `web_servers` inventory group. Both managed nodes receive the same configuration, while Jinja2 renders host-specific webpage content.

## Project Structure

```text
ansible-project-automation/
├── inventory.ini
├── setup-users.yaml
├── deploy-nginx.yaml
├── README.md
├── .gitignore
└── roles/
    └── nginx/
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── templates/
        │   └── index.html.j2
        ├── defaults/
        │   └── main.yml
        ├── vars/
        │   └── main.yml
        └── meta/
            └── main.yml
```

## Features Implemented

### 1. Automated User Provisioning

The `setup-users.yaml` playbook prepares the managed nodes for automation.

It performs the following operations:

- Creates a dedicated `ansible` Linux user.
- Configures SSH public-key authentication using `authorized_keys`.
- Creates a sudoers configuration file under `/etc/sudoers.d/ansible`.
- Validates sudoers syntax using `visudo`.
- Enables passwordless privilege escalation for the lab environment.

This allows Ansible to connect using a non-root SSH account and perform administrative operations through `become: true`.

### 2. Automated Nginx Configuration

The `nginx` role manages the web server configuration through four tasks:

- Installs Nginx using the Ansible package module.
- Starts and enables the Nginx service.
- Allows HTTP traffic using firewalld.
- Deploys a Jinja2-rendered HTML webpage.

### 3. Dynamic Webpage Deployment

The project uses Jinja2 templating to generate host-specific webpage content.

Example:

```html
<h2>Server: {{ inventory_hostname }}</h2>
```

Ansible renders `inventory_hostname` separately for each managed node, allowing a single template to generate different server-specific outputs.

### 4. Ansible Handlers

A handler named `Reload nginx` is notified when the webpage deployment task reports a change.

This demonstrates conditional task execution using `notify` and handlers.

**Note:** Reloading Nginx is not normally necessary for static HTML changes. This implementation demonstrates handler behavior in a learning environment.

### 5. Reusable Ansible Roles

The Nginx automation is organized into a reusable role with separate directories for tasks, handlers, templates, and variables.

The main deployment playbook invokes the role:

```yaml
---
- name: Configure Nginx web servers using roles
  hosts: web_servers
  become: true

  roles:
    - nginx
```

This structure improves maintainability and allows the Nginx configuration to be reused across different server groups.

## Prerequisites

- A Linux-based Ansible controller.
- Two reachable CentOS managed nodes.
- Ansible installed on the controller.
- SSH connectivity between controller and managed nodes.
- Python available on the managed nodes.
- Required Ansible collections, including `ansible.posix` if used by the playbooks.
- Appropriate administrative access for initial user provisioning.

## How to Run

### Step 1 — Clone the repository

```bash
git clone <repository-url>
cd ansible-project-automation
```

### Step 2 — Configure inventory

Update `inventory.ini` with your managed nodes' actual IP addresses and SSH user:

```ini
[web_servers]
ansible_target1 ansible_host=192.0.2.10 ansible_user=ansible
ansible_target2 ansible_host=192.0.2.11 ansible_user=ansible
```

The addresses above are placeholders and must be replaced before execution.

### Step 3 — Prepare the managed servers

For initial provisioning, use an account that already has the required administrative privileges. The initial SSH user and authentication method must be configured appropriately.

```bash
ansible-playbook -i inventory.ini setup-users.yaml
```

After successful provisioning, configure the inventory to connect as the dedicated `ansible` user.

### Step 4 — Verify connectivity

```bash
ansible web_servers -i inventory.ini -m ping
```

### Step 5 — Validate the Nginx playbook

```bash
ansible-playbook -i inventory.ini deploy-nginx.yaml --syntax-check
```

### Step 6 — Deploy the Nginx role

```bash
ansible-playbook -i inventory.ini deploy-nginx.yaml
```

### Step 7 — Verify web server responses

```bash
curl http://<managed-node-1-ip>
curl http://<managed-node-2-ip>
```

The expected response is the deployed HTML page, including the hostname of the responding server.

## Testing and Validation

The project was tested against two CentOS managed nodes.

The role-based deployment was executed successfully with the following play recap:

```text
ansible_target1 : ok=5 changed=0 unreachable=0 failed=0
ansible_target2 : ok=5 changed=0 unreachable=0 failed=0
```

This confirmed that the existing configuration matched the desired state and that Ansible did not make unnecessary changes.

Additional validation included:

- SSH connectivity and Ansible module execution.
- Privilege escalation using `become`.
- Nginx configuration syntax checks.
- Jinja2 template deployment.
- Firewall configuration.
- Handler-based Nginx reload behavior.

## Key Concepts Demonstrated

**Agentless automation:** Ansible manages remote Linux systems over SSH without requiring a dedicated Ansible agent.

**Idempotency:** Repeated execution maintains the desired configuration without making unnecessary changes.

**Privilege escalation:** Ansible connects as a dedicated user and uses sudo for administrative tasks.

**Configuration consistency:** The same automation is applied across multiple managed servers.

**Reusable roles:** Nginx configuration is separated into a reusable and maintainable component.

**Dynamic templating:** Jinja2 enables server-specific content using shared templates.

**Conditional execution:** Handlers respond to changes rather than executing unconditionally.

## Security Considerations

This project was developed in an isolated learning environment.

- SSH private keys and credentials must never be committed to Git.
- Inventory files should not contain secrets.
- Passwordless unrestricted sudo is used for lab demonstration and should be restricted according to production security requirements.
- Production environments should consider Ansible Vault, least-privilege access, and appropriate secrets management.

## Future Improvements

Potential extensions include:

- Ansible Vault for secrets management.
- Parameterized roles using default variables.
- Automated role testing.
- Integration with Terraform-provisioned cloud infrastructure.
- CI/CD validation of Ansible playbooks.

## Conclusion

This project demonstrates practical configuration management by automating Linux user provisioning, SSH authentication, sudo access, Nginx deployment, firewall configuration, and dynamic webpage delivery across multiple managed servers.

It also demonstrates how Ansible roles, handlers, and idempotent modules help create reusable and maintainable infrastructure automation.

## Ansible Role Creation and Project Setup

The Nginx automation was initially developed as a standalone Ansible playbook and later refactored into a reusable Ansible role.

### 1. Create the Ansible Role

Navigate to the project directory on the Ansible controller:

```bash
cd /ansible
```

Generate the standard Ansible role structure:

```bash
ansible-galaxy role init roles/nginx
```

This command automatically creates directories for tasks, handlers, templates, variables, defaults, metadata, and tests.

### 2. Verify the Role Structure

```bash
ls roles/nginx
find roles/nginx -type f
```

The important directories are:

- `tasks/main.yml` — Nginx installation, service management, firewall configuration, and webpage deployment.
- `handlers/main.yml` — Nginx reload handler.
- `templates/` — Jinja2 HTML template.
- `defaults/main.yml` — Default role variables.
- `vars/main.yml` — Role-specific variables.
- `meta/main.yml` — Role metadata.

### 3. Add the Existing Automation Tasks

The tasks from the original `configure-webservers.yaml` were moved into:

```text
roles/nginx/tasks/main.yml
```

Unlike a complete playbook, a role task file contains only the list of tasks. It does not need `hosts`, `become`, or a `tasks:` heading.

### 4. Configure the Handler

The Nginx reload handler was moved into:

```text
roles/nginx/handlers/main.yml
```

The webpage deployment task uses:

```yaml
notify: Reload nginx
```

This notifies the handler when the deployed template changes.

### 5. Move the Jinja2 Template

The existing HTML template was copied into the role:

```bash
cp /ansible/templates/index.html.j2 /ansible/roles/nginx/templates/
```

The template task can then reference the file directly:

```yaml
src: index.html.j2
```

Ansible automatically searches the role's `templates/` directory.

### 6. Create the Role-Based Playbook

A new playbook named `deploy-nginx.yaml` was created:

```yaml
---
- name: Configure Nginx web servers using roles
  hosts: web_servers
  become: true

  roles:
    - nginx
```

This playbook calls the reusable Nginx role for every server in the `web_servers` inventory group.

### 7. Validate the Role

Check playbook syntax:

```bash
ansible-playbook -i inventory.ini deploy-nginx.yaml --syntax-check
```

List the tasks that will execute:

```bash
ansible-playbook -i inventory.ini deploy-nginx.yaml --list-tasks
```

Execute the role:

```bash
ansible-playbook -i inventory.ini deploy-nginx.yaml
```

Run it again without changing anything to verify idempotency.

### 8. Verify Handler Execution

Modify the role's HTML template:

```bash
vi roles/nginx/templates/index.html.j2
```

Rerun the playbook:

```bash
ansible-playbook -i inventory.ini deploy-nginx.yaml
```

When the template changes, Ansible notifies the Nginx reload handler. When no changes are detected, the handler is not triggered.

### Note

`ansible-galaxy role init` is a **one-time development command** used to generate the role structure. Users cloning this repository do not need to run it again because the role is already included in the repository.
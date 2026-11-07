# 🐳 Automated Docker Container Provisioning via Ansible Playbooks

![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)

A fully automated deployment pipeline that uses Ansible playbooks to provision Docker, build a containerized web application, and run it across multiple target virtual machines (Ubuntu and CentOS). The project demonstrates infrastructure-as-code, configuration management, and containerization best practices — from dynamic image tagging to multi-stage Docker builds.

---

## Table of Contents

- [Features](#features)
- [Technology Stack](#technology-stack)
- [System Architecture](#system-architecture)
- [Getting Started](#getting-started)
- [API Endpoints](#api-endpoints)
- [Project Context](#project-context)

---

## Features

### 🌐 Web Application (React + Vite)

- **Modern portfolio interface** — a responsive, dark-themed landing page showcasing team members and hands-on activities.
- **Client-side routing** — navigation between home, team profiles, and activity pages.
- **Vite build pipeline** — fast development and optimized production builds.
- **Static asset generation** — compiled into a `dist/` folder ready for containerization.

### ⚙️ Ansible Playbooks

- **Inventory-driven orchestration** — `inventory.yml` defines target hosts (`ubuntu1`, `ubuntu2`, `centos1`) with IP addresses and SSH users.
- **Role-based playbook structure** — `site.yml` sequentially executes three roles: `docker_install`, `docker_build`, and `docker_run`.
- **Tag-based execution** — run specific stages using `--tags install`, `--tags build`, or `--tags run`.
- **Cross-platform support** — conditional tasks for Ubuntu (APT) and CentOS (DNF) package managers.
- **Docker installation** — installs Docker CE, Python SDK, and required dependencies on all target hosts.
- **Service management** — ensures the Docker daemon is running and enabled on boot.

### 🐳 Docker Build & Run

- **Multi-stage Dockerfile** — Node.js build stage compiles the React app, then copies static assets into a lightweight Nginx Alpine image.
- **Dynamic image tagging** — uses the latest Git commit short hash (`git rev-parse --short HEAD`) as the image tag for traceability.
- **Automated image build** — Ansible transfers the Dockerfile and web app archive, then builds the image on each target host.
- **Container lifecycle management** — runs the container with configurable port mappings, restart policies, and name.
- **Cleanup tasks** — prunes dangling Docker images to free disk space.
- **Firewall configuration** — opens the required port on CentOS using `firewalld`.

---

## Technology Stack

| Layer | Technologies Used |
|-------|-------------------|
| **Frontend** | React.js, Vite, React Router |
| **Containerization** | Docker (multi-stage builds), Nginx Alpine |
| **Configuration Management** | Ansible (playbooks, roles, inventory) |
| **Target Operating Systems** | Ubuntu, CentOS |
| **Build Tooling** | Node.js (npm), Git |
| **Provisioning** | Ansible built-in modules (`apt`, `dnf`, `copy`, `archive`, `unarchive`, `service`, `command`, `set_fact`, `firewalld`) |
| **Docker Modules** | `community.docker.docker_image`, `community.docker.docker_container`, `community.docker.docker_prune` |

---

## System Architecture

1. **Control Node** — The Ansible control node (local machine) holds the playbook repository, inventory file, and SSH credentials.
2. **Inventory Resolution** — Ansible reads `inventory.yml` to identify target VMs (`ubuntu1`, `ubuntu2`, `centos1`) and their connection details.
3. **Playbook Execution** — Running `ansible-playbook -i inventory.yml site.yml` triggers the orchestration.
4. **Role 1: `docker_install`** — Installs Docker prerequisites, Docker engine, Python SDK, and ensures the Docker service is running and enabled on all hosts.
5. **Role 2: `docker_build`** — Captures the current Git commit hash, creates the deployment directory, transfers the Dockerfile and web app archive, builds the Docker image with a dynamic tag, and prunes dangling images.
6. **Role 3: `docker_run`** — Opens the application port on CentOS, then launches the container with the specified image tag, port mappings, and restart policy.
7. **Result** — The containerized web application is running on all target VMs, accessible via their respective IP addresses and configured ports.

```text
Control Node → Ansible Playbook → inventory.yml → Target VMs (Ubuntu/CentOS)
                                      ↓
                            docker_install → docker_build → docker_run
                                      ↓
                            Nginx Alpine Container serving React app
```

---

## Getting Started

### Prerequisites

Ensure the following are installed and configured:

- **Ansible** (v2.15 or higher) on the control node
- **Docker** installed on target VMs (handled by the playbook, but SSH access is required)
- **SSH key-based authentication** to all target VMs (`ubuntu1`, `ubuntu2`, `centos1`)
- **Git** for cloning the repository
- **Python 3** and `pip` on target hosts (for Ansible modules)
- Target VMs with **sudo privileges** (the playbook uses `become: true`)

> **⚠️ Important:** The `inventory.yml` file contains hardcoded IP addresses and usernames. Update these values to match your actual environment before running the playbook. Ensure your SSH keys are added to the target hosts and that the Ansible control node can reach them over the network.

### 1. Clone the Repository

```bash
git clone https://github.com/ivnwynn/CPE212-Final-Project.git
cd CPE212-Final-Project
```

### 2. Configure the Inventory

Edit `ansible/inventory.yml` to reflect your target VMs:

```yaml
all:
  children:
    managed_vms:
      hosts:
        ubuntu1:
          ansible_host: 192.168.56.102
          ansible_user: server-1
        ubuntu2:
          ansible_host: 192.168.56.103
          ansible_user: server-2
        centos1:
          ansible_host: 192.168.56.106
          ansible_user: cent-server
```

### 3. Review and Customize Variables

The `docker_build` role uses `defaults/main.yml` for configurable paths, image names, and tags. Adjust these if needed:

```yaml
app_deploy_dir: "/opt/portfolio_app"
app_image_name: "portfolio_app"
app_image_default_tag: "latest"
local_archive_path: "/tmp/webapp.tar.gz"
```

### 4. Run the Ansible Playbook

Execute the full orchestration:

```bash
cd ansible
ansible-playbook -i inventory.yml site.yml
```

To run specific stages using tags:

```bash
# Install Docker only
ansible-playbook -i inventory.yml site.yml --tags install

# Build the Docker image only
ansible-playbook -i inventory.yml site.yml --tags build

# Run the container only
ansible-playbook -i inventory.yml site.yml --tags run
```

### 5. Verify the Deployment

After the playbook completes successfully:

- SSH into any target VM and verify the container is running:

```bash
ssh server-1@192.168.56.102 "docker ps"
```

- Access the web application in your browser at `http://<target-vm-ip>:<app_host_port>` (default port is configured in the playbook).

### 6. Rebuild and Redeploy

If you make changes to the web application:

```bash
# Re-run the build and run tags
ansible-playbook -i inventory.yml site.yml --tags build,run
```

> **Note:** The dynamic image tag uses the latest Git commit hash, so each build produces a uniquely tagged image. The `docker_run` role automatically uses the most recent commit tag when launching the container.

---

## API Endpoints

**N/A** — This project does not expose a REST API. It is an infrastructure automation and deployment project that provisions Docker containers to serve a static React web application.

---

## Project Context

This project was developed as the final requirement for **CPE 212 — Automating Server Management** at the **Technological Institute of the Philippines, Quezon City Campus**, under the instruction of **Engr. Jimlord M. Quejado**. It demonstrates the practical application of configuration management and containerization: using Ansible to automate the installation of Docker across heterogeneous Linux distributions (Ubuntu and CentOS), building a container image from a multi-stage Dockerfile, and deploying the containerized application with dynamic Git-based tagging. The project emphasizes infrastructure-as-code principles, idempotent playbook design, and cross-platform compatibility — skills essential for modern DevOps and site reliability engineering roles.

- **GitHub Repository:** [https://github.com/ivnwynn/CPE212-Final-Project](https://github.com/ivnwynn/CPE212-Final-Project)
- **Video Walkthrough:** [https://youtu.be/feREoYRUZ2U](https://youtu.be/feREoYRUZ2U)

---

<p align="center"><i>Built with ☕ and Ansible by Andoy, Dueñas, Pizarro, and San Juan.</i></p>

# Ansible Role: docker_build

An Ansible role designed to automate the building, deployment, and lifecycle management of containerized web applications across managed remote target nodes.

---

## Overview

The `docker_build` role executes a complete build-and-deploy pipeline for Docker-based web applications:
1. **Dynamic Tagging:** Queries the local Git repository short commit hash on the control node (`localhost`) to assign a dynamic tag (`v-<hash>`), falling back to `latest` if Git context is unavailable.
2. **Environment Preparation:** Creates target deployment directories on remote hosts and syncs the project `Dockerfile`.
3. **Artifact Compression:** Archives local web application files on `localhost`, excluding heavy/unnecessary development assets (e.g., `node_modules`, `dist`, `.git`).
4. **Extraction & Building:** Transfers and unarchives the application package onto target nodes before building the Docker container image.
5. **Disk Maintenance:** Prunes dangling and intermediate Docker images to save disk space on target systems.

---

## Role Variables

Default variable values are set in `roles/docker_build/defaults/main.yml`:

| Variable | Default Value | Description |
| :--- | :--- | :--- |
| `app_deploy_dir` | `"/opt/portfolio-app"` | Remote target directory where Dockerfile and application files are deployed. |
| `app_image_name` | `"portfolio_app"` | Name of the resulting Docker image built on target nodes. |
| `app_image_default_tag` | `"latest"` | Fallback image tag used if no Git commit hash is found. |
| `app_image_tag` | `"{{ app_image_default_tag }}"` | Image tag variable (dynamically updated to `v-<hash>` when Git is present). |
| `local_archive_path` | `"/tmp/webapp.tar.gz"` | Local temporary file path for the compressed web application archive. |

---

## Tasks Breakdown

The main tasks in `tasks/main.yml` perform the following steps:

1. **Query current local Git commit short hash:** Runs `git rev-parse --short HEAD` on `localhost` without failing if the directory is not a Git repo.
2. **Set dynamic image tag fact if git commit was found:** Registers `app_image_tag` as `v-<hash>` if the Git lookup was successful.
3. **Ensure deployment directory exists on remote hosts:** Creates `app_deploy_dir` on target nodes with `0755` permissions.
4. **Copy Dockerfile to remote hosts:** Transfers the `Dockerfile` to the remote deployment path.
5. **Compress webapp directory locally into a single tarball:** Creates a `.tar.gz` archive of `../webapp` on `localhost` excluding `node_modules`, `dist`, and `.git`.
6. **Transfer and extract webapp archive on managed nodes:** Unpacks the archived application files into `app_deploy_dir` on managed nodes.
7. **Build Docker image on all nodes:** Uses `community.docker.docker_image` to build the container image using the uploaded source files.
8. **Remove dangling Docker images to free disk space:** Uses `community.docker.docker_prune` to clean up unused intermediate build layers.

---

## Requirements & Dependencies

### Ansible Control Node
* Ansible 2.9+
* `git` CLI installed
* Collections:
  * `ansible.builtin`
  * `community.general`
  * `community.docker`

### Managed Remote Nodes
* Docker Engine installed and running
* Python `docker` library installed for Ansible Docker module support (`pip install docker`)

---

## Basic Usage

Include `docker_build` in your playbook:

```yaml
---
- name: Deploy Web Application via Docker
  hosts: webservers
  become: true
  roles:
    - role: docker_build

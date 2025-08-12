# README for Web Service Deployment with Ansible

## Overview

This Ansible project automates the deployment of a web service on both Linux (Nginx) and Windows (IIS) environments using a clean, reusable role (`websvc`). It supports separate configurations for development (`dev`) and production (`prod`) environments with secure secret management via Ansible Vault.

---

## Features

### Option A – Linux (Nginx)
- Installs Nginx web server.
- Serves a dynamic `index.html` generated from a Jinja2 template displaying:
  - Environment name
  - Hostname
  - Deployment date
  - Injected API key (stored securely in Ansible Vault)
- Listens on a configurable port (default: 8080).
- Opens the listening port in the firewall (supports both `firewalld` and `ufw`), ensuring idempotency.
- Restarts Nginx only when configuration or content changes (using handlers).
  
### Option B – Windows (IIS)
- Installs the IIS Web-Server feature if not already present.
- Creates an IIS site with configurable port bindings and content directory.
- Deploys a `web.config` file from a Jinja2 template, including the environment name variable.
- Configures the IIS application pool identity with credentials securely stored in Ansible Vault.
- Opens the HTTP port in the Windows firewall.
- Restarts the IIS site only if necessary (using handlers).

---

## Variable Hierarchy

- `group_vars/dev/vars.yml` and `group_vars/prod/vars.yml`:
  - Environment-specific variables (e.g., `env_name`, ports).
- `group_vars/dev/vault.yml` and `group_vars/prod/vault.yml` (encrypted with Ansible Vault):
  - Linux: `api_key`
  - Windows: `iis_pool_password`

---

## Directory Structure

ansible/
├── group_vars/
│   ├── dev/
│   │   ├── vars.yml
│   │   └── vault.yml  # ansible-vault encrypted
│   └── prod/
│       ├── vars.yml
│       └── vault.yml  # ansible-vault encrypted
├── inventory/
│   ├── dev.ini
│   ├── prod.ini
├── roles/
│   └── websvc/
│       ├── defaults/main.yml
│       ├── tasks/
│       │   ├── main.yml
│       │   ├── linux.yml
│       │   └── windows.yml
│       ├── templates/
│       │   ├── index.html.j2
│       │   └── web.config.j2
│       ├── handlers/main.yml
│       └── ...
├── site-linux.yml
├── site-windows.yml

---

## Idempotency & Testing

- All tasks are designed to be idempotent; subsequent runs will report zero changes unless the configuration actually changes.
- Supports Ansible's `--check` and `--diff` modes for dry runs and previewing changes.
- Meaningful tags allow running subsets of the playbook:
  - `install` – package and feature installation tasks
  - `config` – configuration files and templates deployment
  - `firewall` – firewall port management

---

## Usage

1. Encrypt secrets in vault files using `ansible-vault`.
2. Run the main playbook targeting the appropriate inventory and environment.
3. Use `--ask-vault-pass` or a vault password file to supply vault credentials.
4. Use tags to run specific parts of the deployment if needed.

---

## Summary

This project enables reliable, secure, and configurable deployment of Nginx on Linux and IIS on Windows with best practices in secret management, idempotency, and modular Ansible roles.

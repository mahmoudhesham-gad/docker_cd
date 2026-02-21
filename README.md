# container_CD

An Ansible role that performs a **continuous delivery** workflow for Docker containers. It logs into a container registry, pulls the latest image, replaces the running container, and cleans up dangling images — all idempotently.

---

## Requirements

- Docker must be installed and accessible on the target host.
- The `community.docker` Ansible collection must be installed:

```bash
ansible-galaxy collection install community.docker
```

---

## Role Variables

### Required (must be provided per-deployment)

| Variable | Description |
|---|---|
| `image_name` | Docker image name, e.g. `username/repository` |
| `container_name` | Name of the container to manage |
| `registry_username` | Registry login username — **store in an Ansible Vault** |
| `registry_password` | Registry login password — **store in an Ansible Vault** |

### Optional (have sensible defaults)

| Variable | Default | Description |
|---|---|---|
| `image_tag` | `latest` | Tag of the image to pull and run |
| `container_ports` | `[]` | List of port mappings, e.g. `["80:80"]` |
| `container_env` | `{}` | Dict of environment variables passed into the container |
| `container_volumes` | `[]` | List of volume mounts |
| `container_networks` | `[]` | List of Docker networks to attach the container to |
| `container_restart_policy` | `unless-stopped` | Container restart policy |

### Role-level configuration (`vars/main.yml`)

| Variable | Default | Description |
|---|---|---|
| `target_host` | `star_server` | Inventory host or group to target |
| `registry_url` | `docker.io` | Container registry URL |

---

## Secrets

Sensitive variables (`registry_username`, `registry_password`, `container_env`) should **never** be committed in plain text. Use Ansible Vault to encrypt them:

```bash
ansible-vault create vars/secrets.yml
```

The `vars/` directory has a `.gitignore` that excludes environment-specific subdirectories from version control.

---

## Dependencies

- Collection: [`community.docker`](https://docs.ansible.com/ansible/latest/collections/community/docker/)

---

## Example Playbook

```yaml
- hosts: star_server
  roles:
    - role: container_CD
      vars:
        image_name: "myuser/myapp"
        image_tag: "1.2.3"
        container_name: "myapp"
        container_ports:
          - "8080:80"
        container_env:
          DATABASE_URL: "{{ db_url }}"
          SECRET_KEY: "{{ app_secret }}"
        container_volumes:
          - "/data/myapp:/app/data"
        registry_username: "{{ vault_registry_username }}"
        registry_password: "{{ vault_registry_password }}"
```

---

## Workflow

The role executes the following steps in order:

1. **Ensure Docker is running** — starts and enables the Docker service.
2. **Log in to registry** — authenticates with the registry; triggers the `Pull latest image` handler.
3. **Log out of registry** — immediately revokes the session after the pull is complete.
4. **Remove old container** — force-kills and removes the existing container; triggers the `Start updated container` handler.
5. **Prune dangling images** — cleans up unused image layers.

---

## License

MIT-0

---

## Author

Mahmoud Hesham — [mahmoudhesham656@gmail.com](mailto:mahmoudhesham656@gmail.com)

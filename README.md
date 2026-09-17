```text
.
├── ansible/
│   ├── README.md
│   ├── ansible.cfg
│   ├── inventory.yml
│   ├── group_vars/
│   │   └── all.yml
│   ├── playbook.yml
│   ├── requirements.yml
│   └── templates/
│       └── daemon.json.j2
├── docker/
│   ├── Dockerfile
│   ├── compose.yml
│   └── test_app/
└── README.md
```
## Components

### Ansible

Ansible is responsible for:

- installing and configuring Docker;
- configuring Docker daemon logging;
- configuring firewall rules;
- configuring scheduled Docker cleanup;
- deploying the Docker Compose project.

See [`ansible/README.md`](ansible/README.md) for deployment instructions.

### Docker Compose

The application is deployed using Docker Compose.

The Compose project is located in [`docker/`](docker/) and contains:

- `Dockerfile`;
- `compose.yml`;
- application source code in `test_app/`.

See [`docker/README.md`](docker/README.md) for application-specific documentation.

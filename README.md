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
Installs and configures Docker on the target host, configures Docker daemon logging, firewall and scheduled cleanup, and deploys the Docker Compose project.

See ansible/README.md for deployment instructions.

Docker Compose

The application consists of:

FastAPI application;
PostgreSQL database.

The Compose project is located in docker/ and is copied to the target host by Ansible.

Quick start
cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook --syntax-check playbook.yml
ansible-playbook playbook.yml

See ansible/README.md for configuration and detailed instructions.

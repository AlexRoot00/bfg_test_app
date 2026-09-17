# Ansible Docker Deployment

Ansible-плейбук для развёртывания  проекта на Ubuntu .

## Требования

### Control node

* Ansible Core 2.20+
* Python 3
* SSH-доступ к хосту

Установить необходимую Ansible collection:

```bash
cd ansible
ansible-galaxy collection install -r requirements.yml
```

### хост для развертывания

* Ubuntu 22.04
* SSH-доступ для пользователя Ansible
* права `sudo`

Docker, Docker Compose и необходимые пакеты устанавливаются плейбуком автоматически.

## Настройка inventory

Изменить `ansible/inventory.yml`:

```yaml
all:
  hosts:
    target:
      ansible_host: 192.168.1.100
      ansible_user: ubuntu
```

При необходимости можно указать SSH-ключ:

```yaml
      ansible_ssh_private_key_file: ~/.ssh/....
```

## Переменные

Основные параметры находятся в:

```text
ansible/group_vars/all.yml
```

### Docker logging

Для Docker daemon используется `json-file` с ограничением размера и количества файлов:

```text
max-size: 10m
max-file: 5
```

При изменении конфигурации Docker daemon Ansible автоматически перезапускает Docker.

### Docker cleanup

Ежедневно через cron выполняется:

```bash
docker system prune -f
```

Время задаётся переменными:

```yaml
docker_prune_hour: "3"
docker_prune_minute: "0"
```

## Запуск

Перейти в каталог Ansible:

```bash
cd ansible
```

Проверить доступность target host:

```bash
ansible all -m ping
```

Проверить playbook:

```bash
ansible-playbook --syntax-check playbook.yml
```

Запустить deployment:

```bash
ansible-playbook playbook.yml
```

## Что делает playbook

Плейбук выполняет следующие действия:

1. Проверяет целевую ОС.
2. Устанавливает зависимости, необходимые для Docker repository.
3. Добавляет официальный Docker repository.
4. Устанавливает:

   * Docker Engine;
   * Docker CLI;
   * containerd;
   * Docker Buildx;
   * Docker Compose plugin.
5. Включает и запускает Docker.
6. Настраивает Docker daemon logging и log rotation.
7. Настраивает ежедневный Docker cleanup через cron.
8. Настраивает правила `DOCKER-USER` для Docker traffic.
9. Копирует Compose проект на target host.
10. Запускает Compose project через `community.docker.docker_compose_v2`.

Обновление проекта

Docker Compose проект не клонируется непосредственно на target host.

Файлы из:

docker/

копируются Ansible на:

/opt/notes/

При изменении Dockerfile, application source или других файлов build context используется:

build: always

Это позволяет при повторном запуске Ansible пересобрать image при необходимости и запустить контейнер с актуальной версией проекта.

## Firewall

Правила применяются через Docker chain:

```text
DOCKER-USER
```

Разрешается:

* существующий `ESTABLISHED/RELATED` traffic;
* внутренний traffic между Docker containers;
* исходящий traffic контейнеров;
* опубликованный application port.

## Идемпотентность:

При повторном запуске без изменений:

* установленные пакеты не переустанавливаются;
* Docker daemon не перезапускается без изменения конфигурации;
* cron не создаёт дубликаты;
* существующий проект  приводится к требуемому состоянию;
* контейнеры не пересоздаются без необходимости.


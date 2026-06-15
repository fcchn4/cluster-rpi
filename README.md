# Cluster Raspberry Pi - Ansible

Proyecto Ansible para automatizar la instalación y configuración de un clúster Kubernetes en Raspberry Pi 3B y 4B, con el sistema operativo instalado en almacenamiento SSD via USB.

## Hardware

| Rol        | Modelo           | Almacenamiento |
|------------|------------------|----------------|
| Master     | Raspberry Pi 4B  | SSD USB        |
| Workers    | Raspberry Pi 3B / 4B | SSD USB    |

## Topología del clúster

| Rol     | IP             |
|---------|----------------|
| Master  | 192.168.1.41   |
| Worker  | 192.168.1.42   |
| Worker  | 192.168.1.43   |
| Worker  | 192.168.1.44   |
| Worker  | 192.168.1.45   |

## Requisitos previos

1. Raspberry Pi OS (Debian Bookworm o posterior) instalado en SSD USB
2. Servidor SSH habilitado en cada nodo
3. Clave SSH pública copiada al usuario `pi` en cada nodo (`~/.ssh/authorized_keys`)
4. IP estática configurada en cada nodo
5. Ansible instalado en la máquina de control

### Instalar Ansible (máquina de control)

```bash
sudo apt install ansible
# o con pip
pip install ansible
```

### Copiar clave SSH a los nodos

```bash
ssh-copy-id -i ~/.ssh/rpi-ssh.pub pi@192.168.1.41
ssh-copy-id -i ~/.ssh/rpi-ssh.pub pi@192.168.1.42
# repetir para cada nodo...
```

## Estructura del proyecto

```
cluster-rpi/
├── inventory/
│   └── inventory.yml       # Definición de hosts (master + workers)
├── base-os-update.yml      # Actualización del sistema operativo
├── base-packages.yml       # Instalación de paquetes base + repositorio Kubernetes
├── k8s-master.yml          # Configuración del nodo master
├── k8s-workers.yml         # Configuración de los nodos worker
├── Makefile                # Comandos de conveniencia
└── README.md
```

## Playbooks

### `base-os-update.yml`
Actualiza todos los paquetes del sistema operativo en todos los nodos.
```bash
ansible-playbook base-os-update.yml -i inventory/inventory.yml
```

### `base-packages.yml`
Instala los paquetes base necesarios y agrega el repositorio oficial de Kubernetes (v1.36).

Paquetes incluidos: `vim`, `curl`, `wget`, `git`, `rsync`, `jq`, `htop`, `build-essential`, herramientas de compresión, entre otros.
```bash
ansible-playbook base-packages.yml -i inventory/inventory.yml
```

### `k8s-master.yml`
Configura el nodo master del clúster Kubernetes.
```bash
ansible-playbook k8s-master.yml -i inventory/inventory.yml
```

### `k8s-workers.yml`
Configura los nodos worker del clúster Kubernetes.
```bash
ansible-playbook k8s-workers.yml -i inventory/inventory.yml
```

## Uso con Makefile

```bash
# Ejecutar actualización del SO + instalación de paquetes base
make

# Solo actualizar el SO
make base-update

# Solo instalar paquetes base
make base-packages

# Instalar K3s en el servidor (master)
make k3s-server

# Instalar K3s en los agentes (workers)
make k3s-agents
```

## Configuración del inventario

El archivo `inventory/inventory.yml` usa variables de entorno para las credenciales:

| Variable                   | Descripción                        |
|----------------------------|------------------------------------|
| `debian_user`              | Usuario SSH (por defecto: `pi`)    |
| `debian_port`              | Puerto SSH (por defecto: `22`)     |
| `debian_private_key_file`  | Ruta a la clave privada SSH        |

Se recomienda definir estas variables en un archivo `.env` o en `group_vars/`.

## Notas

- El usuario por defecto es **`pi`** con configuración `NOPASSWD` en sudoers, por lo que **no es necesario** pasar `--ask-become-pass`.
- La versión de Kubernetes configurada es **v1.36**. Para cambiarla, modificar la variable `k8s_version` en `base-packages.yml`.
- El intérprete de Python utilizado es `/usr/bin/python3`.
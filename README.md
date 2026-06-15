# Cluster Raspberry Pi - Ansible

Playbooks Ansible para instalar y configurar un clúster Kubernetes en Raspberry Pi, con sistema operativo en SSD USB.

## Clúster

| Nodo    | Modelo          | IP             | Rol    |
|---------|-----------------|----------------|--------|
| rpi-01  | Raspberry Pi 3B | 192.168.1.41   | Master |
| rpi-02  | Raspberry Pi 3B | 192.168.1.42   | Master |
| rpi-03  | Raspberry Pi 4B | 192.168.1.43   | Worker |
| rpi-04  | Raspberry Pi 4B | 192.168.1.44   | Worker |
| rpi-05  | Raspberry Pi 4B | 192.168.1.45   | Worker |

**SO:** Raspberry Pi OS — Debian 13 Trixie | **Almacenamiento:** SSD USB

## Requisitos

- Raspberry Pi OS (Debian 13 Trixie) instalado en SSD USB en cada nodo
- SSH habilitado con IP estática por nodo
- Clave SSH copiada al usuario `pi` en cada nodo
- Ansible instalado en la máquina de control

```bash
# Copiar clave SSH a cada nodo
ssh-copy-id -i ~/.ssh/rpi-ssh.pub pi@192.168.1.41
```

## Playbooks

| Archivo              | Descripción                                          |
|----------------------|------------------------------------------------------|
| `base-os-update.yml` | Actualiza el SO en todos los nodos                   |
| `base-os-packages.yml` | Instala paquetes base y repositorio Kubernetes v1.36 |
| `k8s-master.yml`     | Configura los nodos master (RPi 3B)                  |
| `k8s-workers.yml`    | Configura los nodos worker (RPi 4B)                  |

## Uso

```bash
# Actualizar SO
ansible-playbook base-os-update.yml -i inventory/inventory.yml

# Instalar paquetes base
ansible-playbook base-os-packages.yml -i inventory/inventory.yml

# Configurar masters
ansible-playbook k8s-master.yml -i inventory/inventory.yml

# Configurar workers
ansible-playbook k8s-workers.yml -i inventory/inventory.yml
```

## Lista de comandos

```bash
ansible all -m ping -i inventory/ping-inventory.yml
ansible-playbook base-os-update.yml -i inventory/inventory.yml --ask-become-pass
ansible-playbook base-os-packages.yml -i inventory/inventory.yml --ask-become-pass
```

## Notas

- Para entornos con sudo con password, usar `--ask-become-pass`
- Variables de conexión (`ansible_user`, `ansible_port`, `ansible_private_key_file`) definidas en `inventory/inventory.yml`
- Versión de Kubernetes: `k8s_version` en `base-os-packages.yml` (actualmente `1.36`)
# Quickstart Guide

## Install System Dependencies
### Installing Docker

Follow the [official docker installation guide](https://docs.docker.com/engine/install) and add your user to the docker group.

### Installing Libvirt
This is only needed for bootstrapping and testing the ISO installation in a VM
=== "Fedora"

    ```bash
    sudo dnf install libvirt virt-manager edk2-ovmf -y
    ```
=== "Debian"
    ```bash
    sudo apt install -y qemu-system libvirt-daemon-system virt-manager ovmf
    ```

## Clone Project
```bash
git clone https://github.com/projectpotos/potos-iso-builder.git
```
```bash
cd potos-iso-builder
```

## Install Build Dependencies
The main build tool used by the Potos-ISO-Builder is go-task.
It can be installed follow the [offical installation guide](https://taskfile.dev/docs/installation) 

## Preparing minimal Potos config
Copy the example configuration
```bash
mkdir input
cp examples/minimal/config.yml input
```

### Set keyboard layout
Set a keyboard layout in `input/config.yml` so that the iso installation runs automatically trough.
```yaml
locale:
  lang: "en_US.UTF-8"
  keyboard: "ch"
  timezone: "UTC"
```

### Admin Password
Generate a password for the Potos admin user
```bash
openssl passwd -6
```
Replace password_hash in `input/config.yml`:
```yaml
initial_user:
  username: "admin"
  password_hash: <your_hash>
```

### Disk Encryption Password
Set the disk encryption password under `disk_encryption` in `input/config.yml`:
```yaml
disk_encryption:
  enabled: true
  init_password: <put you password here>
```

## Build minimal Potos ISO
The Potos installation-iso will be created at the path `tests/output/potos-installer.iso` using this command:

```bash
OUTPUT_DIR=tests/output task build
```

A installation into a VM can be started with the this command:

```bash
task test:bootstrap
```

!!! note

    In case `ERROR Unknown OS name` is show check the output of `virt-install --osinfo list | grep fedora` and replace the `OS_VARIANT` value in `tests/scripts/bootstrap.sh` with an available option e.g. `OS_VARIANT="fedora-unknown"`

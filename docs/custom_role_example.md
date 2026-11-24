---
icon: lucide/rocket
---

# Custom Role Example
This minimal example shows how custom roles can be integrated into a Potos client.

## Create Role Collection
Create a public repository hosting the roles. We assume that the repo is hosted under `https://github.com/somebody/projectpotos.custom_roles.git` for this example.
For demonstration purposes we will create a simple role that install thunderbird using the file structure and content as show below.

```bash
├── galaxy.yml
├── README.md
└── roles
    └── install_packages
        ├── defaults
        │   └── main.yml
        └── tasks
            └── main.yml
```

```yaml
# cat galaxy.yml
namespace: projectpotos
name: custom_roles
version: 1.0.0
readme: README.md
authors:
  - Potos
description: Potos role example
license:
  - MIT
```

```yaml
# cat roles/install_packages/defaults/main.yml
---
packages_to_install:
  - thunderbird
```

```yaml
# cat roles/install_packages/tasks/main.yml
---
- name: Install packages using DNF
  ansible.builtin.dnf:
    name: "{{ packages_to_install }}"
    state: present
    update_cache: true
```

## Integrate Role into Potos
This new role can now be used by adapting the [Specs repo](spec_repo_overview.md) configuration of the Potos client.
First add your new collection to the `requirements.yml`:
```yaml
collections:
  - name: https://github.com/somebody/projectpotos.custom_roles.git
    type: git
    version: main
```
After that the `install_packages` roll can be called from `runtime.yml`:
```yaml
---
runtypes:
  hourly:
    - projectpotos.custom_roles.install_packages

  daily:
    - projectpotos.custom_roles.install_packages
```

To test the role, manually trigger a Ansible pull on the client with:
```
potos-ansible-pull -r daily -v
```

## Set Variables per Host
Using the new role, we now can install different packages per group or by host.
To install a different set of packages for the host `potos-test` we can overwrite the `packages_to_install` variable of `roles/install_packages/defaults/main.yml` from within our specs repository by creating the following files and folders:

```
inventory/
└── host_vars
    └── potos-test.yml
```
```yaml
# cat inventory/host_vars/potos-test.yml
packages_to_install:
  - thunderbird
  - htop
```

---
icon: lucide/rocket
---

# Specs Repository Overview
The minimal Specs Repository can be cloned from [GitHub](https://github.com/projectpotos/ansible-specs-fedora-test).
Refer to [Add custom Specs Repository](add_custom_spec.md) for information on how  to intagrate it into the potos client.

* The `runtime.yml` specifies what roles are run `hourly` and `daily` by the triggers managed by the potos base collection.
* The file `base-requirements.yml` specifies the URI and branch of the Potos base collection.
* Resources that are need and have to be installed for running roles specified in `runtime.yml` go to the file `requirements.yml`. To make use of of the Potos role collection add it to `requirements.yml` as show in the following listing.

```yaml
collections:
  - name: https://github.com/projectpotos/ansible-collection-potos.roles.git
    type: git
    version: main
```

## Setting Wallpaper Example
In this example we are going to set the wallpaper for all managed clients using the `projectpotos.roles.wallpaper` role.
Create the missing files and folders in the specs repository as shown below.
```bash
├── base-requirements.yml
├── files
│   └── wallpaper
│       └── <place your wallpaper picture here (e.g. my_wallpaper.jpg)>
├── inventory
│   └── group_vars
│       └── all
│           └── wallpaper.yml
├── renovate.json
├── requirements.yml
└── runtime.yml
```

Copy the following content to `inventory/group_vars/all/wallpaper.yml` and and adjust the filename of your wallpaper.
```yaml
---
wallpaper_images:
  - name: wallpaper_1
    filename: /var/lib/potos/specs/files/wallpaper/<place your wallpaper picture filename here (e.g. my_wallpaper.jpg)>
    options: zoom
    pcolor: "#1c2e5d"
    scolor: "#1c2e5d"
    shade_type: solid
```

Add the `projectpotos.roles.wallpaper` role to `runtime.yml`:

```yaml
---
runtypes:
  hourly:
    - ansible-role-motd
    - projectpotos.roles.wallpaper
  daily:
    - ansible-role-motd
    - projectpotos.roles.wallpaper
```

Commit the changes and push it to your specs repository. Then manually trigger a Ansible pull on the client with:

```bash
potos-ansible-pull -r daily -v
```

If everything worked out, you will see the new wallpaper on the next login in the client.
